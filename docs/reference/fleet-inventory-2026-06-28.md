# Fleet Inventory - 2026-06-28

This is a sanitized current-state inventory. It is derived from local command evidence, but raw OpenClaw output is not included.

## Summary

| Alias | Role | OS family | OpenClaw version | Channel | Gateway | Service | Default model family | Cron summary | Model overrides | Last checked | Main action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| machine-01 | workstation | macOS | 2026.6.10 | stable | healthy | running | Ark | 28 enabled / 8 disabled | 0 | 2026-06-28 | monitor natural reruns and keep optional issues classified |

No other OpenClaw machines were added to this inventory because no additional authorized access path was available in this run.

## machine-01

Role: workstation

Health classes:

- Gateway: healthy.
- Service manager: running.
- Task backlog: clear.
- Channel delivery: configured and healthy.
- Security: no critical or warning findings in the status summary.
- Default model family: Ark.

Cron aggregate:

| Metric | Count |
| --- | ---: |
| Total jobs | 36 |
| Enabled jobs | 28 |
| Disabled jobs | 8 |
| Enabled command jobs | 4 |
| Enabled agentTurn jobs | 24 |
| Cron-level model overrides | 0 |
| Historical ok status | 15 |
| Historical error status | 13 |
| Temporary verification jobs | 0 |
| Enabled agentTurn jobs outside exec-only allowlist | 0 |

Current issue classes:

- `channel-setup-mode`: Telegram remains in allowlist-oriented setup mode. Delivery is healthy; broad group access is not enabled.
- `auth-expired`: optional non-Ark provider profiles are expired. Current default and cron path use Ark, so this is not blocking.
- `orphan-transcripts`: orphan transcript files exist. Archival is safe-deferred unless performed by a reliable official path or explicitly approved.
- `cron-store-needs-normalization`: doctor still flags model-driven isolated agent jobs that use shell/process tools. These are intentionally retained as `agentTurn` because they perform model-driven content work; pure report jobs were already converted to command cron.
- `memory-search-disabled`: memory search is explicitly disabled. This is intentional in the current configuration.

Verification evidence classes:

- `openclaw status --deep`: Gateway reachable, service loaded/running, no task backlog, default model Ark.
- `openclaw doctor`: warnings classified above; no channel security warning.
- `openclaw cron list --all --json`: aggregate counts shown above.
- Ark smoke test: completed through Gateway on the Ark model path.
- GitHub CLI: keyring auth works when stale environment tokens are unset.
- Remote repository: `kt-aicoding/claws` is reachable on `main`.

Deferred or not in scope:

- Reauth for optional non-Ark providers.
- Manual orphan transcript archival.
- Converting model-driven content jobs to command cron.
- Additional machines without authorized access.
