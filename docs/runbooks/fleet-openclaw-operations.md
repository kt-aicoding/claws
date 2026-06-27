# Fleet OpenClaw Operations Runbook

Use this runbook when checking OpenClaw across more than one machine. It extends the single-machine cron runbook with inventory, comparison, and data-minimization rules.

## Scope

The goal is to answer four questions safely:

1. Which machines have OpenClaw installed and active?
2. Are their gateways, model defaults, cron stores, and delivery channels healthy?
3. Which fixes from a known-good machine should be applied elsewhere?
4. What can be shared publicly without leaking local operational data?

## Data rule

Never publish raw command output from a machine.

Treat these as sensitive by default:

- Hostnames and usernames.
- Public or private IP addresses.
- Absolute local paths.
- Raw `openclaw.json`.
- SQLite databases.
- Transcript `.jsonl` files.
- Auth profiles and tokens.
- Telegram IDs and channel identifiers.
- Exact cron expressions.
- Raw `openclaw status --deep` output.
- Raw `openclaw doctor` output.
- Full GitHub issue bodies if they came from private automation.

Share only sanitized summaries and reusable practices.

## Machine aliases

Assign each machine a stable alias before collecting notes:

| Alias | Intended meaning |
| --- | --- |
| `machine-01` | Primary workstation |
| `machine-02` | Secondary workstation |
| `machine-03` | Always-on host or server |
| `machine-lab-*` | Experimental installs |

The alias should not reveal the real hostname, location, owner username, IP address, or device serial number.

## Safe collection checklist

Run these commands on each machine, but only copy the sanitized fields into the inventory template.

### 1. Version and gateway class

```sh
openclaw status --deep
```

Do not paste the raw output into a public repository. Extract only:

- OpenClaw version.
- Channel.
- Gateway class: healthy, degraded, offline, or unknown.
- LaunchAgent/service class: running, installed-not-loaded, missing, or unknown.
- Active/queued/running task counts.
- Default model family.

### 2. Doctor summary

```sh
openclaw doctor
```

Record only issue classes, not raw details:

- auth-expired
- gateway-unreachable
- launch-agent-not-loaded
- cron-store-needs-normalization
- orphan-transcripts
- channel-setup-mode
- memory-search-disabled
- no-critical-issues

### 3. Cron aggregate

Use aggregate counts instead of raw job payloads:

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

Do not publish job payloads, exact schedules, delivery targets, or raw errors.

### 4. Model inheritance check

```sh
openclaw cron list --all --json \
  | jq '[.jobs[] | select(.payload.model != null)] | length'
```

Public summary should say only:

- `0 overrides`
- `N overrides remain`
- `unknown`

### 5. Tool allowlist shape

```sh
openclaw cron list --all --json \
  | jq '{
      enabled_jobs: ([.jobs[] | select(.enabled == true)] | length),
      non_exec_only_jobs: ([.jobs[]
        | select(.enabled == true and .payload.kind == "agentTurn")
        | select((.payload.toolsAllow // []) != ["exec"])
      ] | length)
    }'
```

Use the result to decide whether the machine follows the hardened cron pattern.

## Comparison baseline

A healthy machine does not need to be identical to the primary machine. Compare behavior, not names.

Recommended target:

| Area | Target |
| --- | --- |
| Gateway | reachable |
| Service manager | loaded/running where applicable |
| Active backlog | no unexplained active/queued/running tasks |
| Default model | Ark family, if that is the chosen fleet default |
| Cron model overrides | zero, unless documented |
| Pure report jobs | command cron |
| Model reasoning jobs | agentTurn cron |
| External fetches | timeout, fallback, output cap |
| GitHub CLI usage | keyring auth with stale env tokens unset |
| Published inventory | sanitized aliases and aggregate counts |

## Repair order

Apply fixes in this order on each machine:

1. Back up local runtime state locally only.
2. Restore gateway/service health.
3. Confirm global model default.
4. Remove unnecessary cron-level model overrides.
5. Convert pure reports/reminders to command cron.
6. Harden external fetches with timeouts and output caps.
7. Normalize GitHub CLI auth behavior.
8. Run read-only aggregate checks.
9. Run one no-delivery verification job if needed.
10. Let write jobs validate naturally unless there is a narrow reason to trigger one.

## Backup policy

Backups stay on the machine being repaired. Do not push backup files to this repository.

Local backup examples:

- OpenClaw config.
- OpenClaw state database.
- Cron store.
- Agent metadata required for rollback.

Never copy raw backups into a shared workspace unless there is an explicit incident response reason.

## Public reporting format

Use this shape when sharing fleet status:

```md
| Alias | Role | Version | Gateway | Cron totals | Model overrides | Main action |
| --- | --- | --- | --- | --- | --- | --- |
| machine-01 | workstation | 2026.x | healthy | 28 enabled / 8 disabled | 0 | monitor natural reruns |
| machine-02 | workstation | unknown | unknown | not collected | unknown | run read-only audit |
```

Keep details coarse. Public readers need the method and operating posture, not your private machine map.

## When to avoid automation

Do not run fleet-wide write actions when:

- GitHub auth status differs by machine.
- Delivery targets are unclear.
- Cron jobs write to public repositories.
- The gateway is flapping.
- You have not separated historical error state from current configuration health.

In those cases, collect read-only summaries first and repair one machine at a time.
