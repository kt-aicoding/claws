# 2026-06-27 OpenClaw Cron Repair

This case study records a real OpenClaw maintenance session. It is sanitized for sharing: no tokens, raw config dumps, transcripts, or local database files are included.

## Situation

A local OpenClaw installation had many scheduled jobs showing failure. The initial temptation was to disable failing jobs, but that would have hidden useful signal and reduced automation coverage without fixing root causes.

The working question was:

> Why are scheduled jobs failing, are they all using Ark, and how do we fix the current cron system without blindly shutting things off?

## Findings

The failures were mostly model-driven scheduled jobs that fetched external sources, summarized content, and sometimes wrote to GitHub. Several jobs had historical `error` status from earlier runs, but that did not mean the repaired configuration was still broken.

Main contributing factors:

- Long or unbounded external fetch/tool phases.
- Disabled or unavailable web tools in cron contexts.
- Source-specific instability such as Reddit/Hugging Face primary endpoints being unreliable.
- macOS incompatibility with GNU-style `timeout`.
- Per-job model overrides that could drift from the intended default stack.
- Some pure reporting jobs were implemented as agent turns even though no model reasoning was needed.
- A stale `GH_TOKEN` environment variable could shadow a valid GitHub CLI keyring login.
- OpenClaw gateway can appear broken if the LaunchAgent is installed but not loaded.

## Repairs

### Model stack

All enabled and disabled cron jobs were normalized to inherit the global model stack. Cron-level model overrides were removed.

Reference target:

- Primary: `volcengine/ark-code-latest`.
- Fallbacks: Ark-compatible code/model fallbacks.
- Per-job model override count: `0`.

### Tool policy

Enabled jobs were moved to a narrow tool surface:

- `toolsAllow=["exec"]` for cron jobs needing shell access.
- No `web_search` or `web_fetch` allowlist in enabled cron jobs.
- Source retrieval happens through explicit command-line fetches with timeouts and fallback logic.

### Timeouts and bounded output

GNU `timeout` was replaced with macOS-safe alternatives:

```sh
curl --max-time 20 ...
```

or:

```sh
LC_ALL=C perl -e 'alarm shift; exec @ARGV' 20 command args...
```

Large responses are truncated before entering model context:

```sh
curl --max-time 20 "$URL" | head -c 20000
```

### GitHub CLI auth

Scheduled jobs use keyring-backed GitHub CLI auth and explicitly ignore stale environment tokens:

```sh
env -u GH_TOKEN -u GITHUB_TOKEN gh ...
```

This matters because `GH_TOKEN` takes precedence over keyring auth. If it is expired, `gh` can fail with `Bad credentials` even though keyring auth is valid.

### Command cron conversion

Pure reporting/reminder jobs were converted from model-driven `agentTurn` jobs to `command` jobs:

- `定时任务健康巡检`
- `三爪协作同步`
- `每周系统健康检查`
- `AI 订阅账单提醒`

This reduced model usage, removed unnecessary reasoning loops, and made these tasks easier to verify.

### Avoid recursive OpenClaw CLI calls inside command cron

Command cron jobs that called `openclaw ...` from inside the gateway process produced config/gateway validation noise. The fix was to read local SQLite state directly for health summaries, rather than recursively invoking OpenClaw from a scheduled command.

Pattern:

```sh
DB="$HOME/.openclaw/state/openclaw.sqlite"
sqlite3 "$DB" "select count(*) from cron_jobs where enabled=1;"
```

### Gateway LaunchAgent recovery

After a later check, the gateway returned `1006 abnormal closure`. `openclaw doctor` showed:

- LaunchAgent installed.
- LaunchAgent not loaded.

Repair:

```sh
openclaw gateway restart
```

Verification:

```sh
openclaw status --deep
```

The gateway returned to loaded/running and cron listing worked again.

## Verification

Verification covered both static configuration and live execution paths.

Static checks:

- Total cron jobs stayed at `36`.
- Enabled jobs stayed at `28`.
- Disabled jobs stayed at `8`.
- Enabled command jobs: `4`.
- Enabled agentTurn jobs: `24`.
- Cron-level model overrides: `0`.
- Temporary verification jobs left behind: `0`.
- No enabled jobs allowed `web_search` or `web_fetch`.

Live checks:

- Ark gateway smoke test returned an expected marker.
- A temporary no-delivery command job completed successfully and auto-deleted.
- All four command payloads were executed locally without GitHub writes.
- A natural write-path job completed on Ark and created a GitHub issue.
- OpenClaw status reported the gateway reachable and the LaunchAgent loaded.

## What was intentionally not done

Some items were left unchanged on purpose:

- Failed cron jobs were not blindly disabled.
- Historical error status was not manually cleared.
- OpenAI/Qwen expired auth profiles were not reauthenticated because current cron/default model usage is Ark.
- Orphan transcript files were not manually archived because official non-interactive repair did not archive them and manual matching risked damaging session history.
- Model-driven content tasks were not mechanically converted to command jobs.

## Lessons

The highest-leverage fixes were not turning things off. They were making execution bounded, making auth deterministic, removing model drift, converting non-reasoning work to commands, and validating the real scheduler path.
