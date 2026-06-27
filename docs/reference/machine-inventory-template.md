# Machine Inventory Template

Use this template to track OpenClaw across multiple machines without exposing private local data.

## Rules

- Use aliases, not hostnames.
- Use roles, not owner names.
- Use health classes, not raw command output.
- Use coarse cadence, not exact cron expressions.
- Use issue classes, not raw error messages.
- Keep local backup paths out of public notes.

## Fleet summary

| Alias | Role | OS family | OpenClaw version | Channel | Gateway | Service | Default model family | Cron summary | Model overrides | Last checked | Main action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| machine-01 | workstation | macOS | 2026.x | stable | healthy | running | Ark | 28 enabled / 8 disabled | 0 | YYYY-MM-DD | monitor natural reruns |
| machine-02 | workstation | unknown | unknown | unknown | unknown | unknown | unknown | not collected | unknown | YYYY-MM-DD | run read-only audit |

## Per-machine detail

Copy this section once per machine.

```md
## machine-XX

Role: workstation / server / lab / CI
OS family: macOS / Linux / Windows / unknown
OpenClaw version: 2026.x / unknown
Channel: stable / dev / unknown
Gateway: healthy / degraded / offline / unknown
Service manager: running / installed-not-loaded / missing / unknown
Default model family: Ark / mixed / unknown
Cron totals: N total, N enabled, N disabled
Cron type split: N command, N agentTurn
Cron model overrides: 0 / N / unknown
Historical status split: N ok, N error, unknown
Delivery channels: configured / partial / unknown
Primary issue classes:
- no-critical-issues
- auth-expired
- gateway-unreachable
- launch-agent-not-loaded
- cron-store-needs-normalization
- orphan-transcripts
- memory-search-disabled

Actions taken:
- YYYY-MM-DD: read-only audit
- YYYY-MM-DD: gateway restored
- YYYY-MM-DD: cron model overrides removed

Pending:
- monitor natural reruns
- reauth optional providers
- archive orphan transcripts only through reliable official path
```

## Issue class glossary

| Issue class | Meaning | Public detail level |
| --- | --- | --- |
| `no-critical-issues` | No blocking issue found | safe |
| `auth-expired` | A non-current provider profile needs reauth | do not name tokens |
| `gateway-unreachable` | CLI could not reach gateway | do not include IP/URL |
| `launch-agent-not-loaded` | Service exists but is not loaded | safe |
| `cron-store-needs-normalization` | Doctor recommends cron store repair | safe |
| `orphan-transcripts` | Unreferenced transcript files exist | count may be shared if useful |
| `memory-search-disabled` | Memory search disabled by config | safe |
| `delivery-channel-warning` | Telegram or other channel needs review | do not include IDs |

## Sanitized aggregate command

This command is intended for local use. Review output before sharing.

```sh
openclaw cron list --all --json \
  | jq '{
      total: .total,
      enabled: ([.jobs[] | select(.enabled == true)] | length),
      disabled: ([.jobs[] | select(.enabled != true)] | length),
      command_jobs: ([.jobs[] | select(.enabled == true and .payload.kind == "command")] | length),
      agent_turn_jobs: ([.jobs[] | select(.enabled == true and .payload.kind == "agentTurn")] | length),
      model_overrides: ([.jobs[] | select(.payload.model != null)] | length),
      historical_ok: ([.jobs[] | select(.status == "ok")] | length),
      historical_error: ([.jobs[] | select(.status == "error")] | length)
    }'
```

## Publication checklist

- [ ] No hostnames.
- [ ] No usernames.
- [ ] No IP addresses.
- [ ] No absolute paths.
- [ ] No exact cron expressions.
- [ ] No raw status output.
- [ ] No raw doctor output.
- [ ] No config dumps.
- [ ] No SQLite databases.
- [ ] No transcript files.
- [ ] No auth material.
