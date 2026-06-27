# OpenClaw Cron Operations Runbook

Use this runbook when scheduled jobs are failing, noisy, slow, or suspected to be using the wrong model.

## 1. Start with read-only state

```sh
openclaw status --deep
openclaw doctor
openclaw cron status
openclaw cron list --all --json
```

Do not disable jobs at this stage. First identify whether failures are current execution failures or historical last-run state.

## 2. Confirm gateway health

Healthy signals:

- Gateway reachable on local loopback.
- Gateway service installed, loaded, and running.
- No active/queued/running task backlog unless expected.

If doctor reports the LaunchAgent is installed but not loaded:

```sh
openclaw gateway restart
openclaw status --deep
```

## 3. Confirm model inheritance

Cron jobs should normally inherit the global model stack.

Check for per-job model overrides:

```sh
openclaw cron list --all --json \
  | jq '[.jobs[] | select(.payload.model != null) | {name, model: .payload.model}]'
```

Expected default for this practice:

- Primary model: Volcengine Ark.
- Fallbacks: Ark-compatible fallbacks.
- Cron-level model overrides: `0`.

## 4. Inspect tools and task type

Agent jobs that need shell access should have a narrow tool allowlist.

Prefer:

```json
["exec"]
```

Avoid allowing unavailable tools such as `web_search` or `web_fetch` in cron contexts if the actual implementation can fetch sources through bounded shell commands.

Use `command` cron for:

- Health reports.
- SQLite summaries.
- GitHub read-only summaries.
- Billing reminders.
- Other deterministic shell work.

Use `agentTurn` cron for:

- Summarization.
- Ranking.
- Ideation.
- Content generation.
- Work that genuinely needs model judgment.

## 5. Bound external work

Every external source fetch should have:

- Short timeout.
- Fallback path.
- Output size cap.
- No-op behavior when source data is unavailable.

Examples:

```sh
curl --fail --silent --show-error --max-time 20 "$URL"
```

```sh
curl --max-time 20 "$URL" | head -c 20000
```

For commands that do not provide native timeout support on macOS:

```sh
LC_ALL=C perl -e 'alarm shift; exec @ARGV' 20 command args...
```

## 6. Make GitHub auth deterministic

If `GH_TOKEN` is stale, it can override a valid keyring login.

Check without printing tokens:

```sh
printf 'GH_TOKEN=%s\nGITHUB_TOKEN=%s\n' "${GH_TOKEN:+set}" "${GITHUB_TOKEN:+set}"
env -u GH_TOKEN -u GITHUB_TOKEN gh auth status
env -u GH_TOKEN -u GITHUB_TOKEN gh api user --jq '.login'
```

Inside cron payloads, prefer:

```sh
env -u GH_TOKEN -u GITHUB_TOKEN gh ...
```

## 7. Verify without causing writes

For deterministic command jobs, use temporary no-delivery verification jobs or direct local command execution.

For write jobs, prefer one natural scheduled run or a narrowly scoped manual run. Do not batch-trigger all GitHub-writing jobs.

Verification targets:

- Scheduler can enqueue and finish a job.
- Agent path can call Ark.
- Command cron can run without recursive OpenClaw gateway noise.
- GitHub CLI can authenticate as the expected account.
- Temporary verification jobs are removed.

## 8. Preserve evidence

Do not manually clear `error` status just to make dashboards green.

Historical `error` should turn into `ok` after the job actually runs successfully. Until then, keep it visible so follow-up monitoring can see which jobs still need natural validation.

## 9. Back up before repair

Before changing cron configuration:

```sh
mkdir -p "$HOME/.openclaw/backups/YYYYMMDD-HHMMSS"
```

Back up only local runtime state. Do not commit backups to a public repository.

Recommended backup targets:

- `~/.openclaw/openclaw.json`
- `~/.openclaw/state/openclaw.sqlite`
- `~/.openclaw/state/cron`
- selected agent/session metadata if needed

## 10. Final checklist

- [ ] Gateway reachable.
- [ ] LaunchAgent loaded and running.
- [ ] Telegram/channel delivery is healthy if used.
- [ ] Enabled/disabled counts are intentional.
- [ ] Command jobs are pure deterministic shell work.
- [ ] Model-driven jobs remain agentTurn.
- [ ] Cron model overrides are intentional or zero.
- [ ] No stale temp verification jobs remain.
- [ ] No raw secrets/config/transcripts are published.
