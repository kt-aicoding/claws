# Fleet Inventory - 2026-06-28

This is a sanitized current-state inventory. It is derived from local command evidence, but raw OpenClaw output is not included.

## Summary

| Alias | Role | OS family | OpenClaw version | Channel | Gateway | Service | Default model family | Cron summary | Model overrides | Last checked | Main action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| machine-01 | workstation | macOS | 2026.6.10 | stable | healthy | running | Ark | 28 enabled / 8 disabled | 0 | 2026-06-28 | monitor natural reruns and keep optional issues classified |
| machine-02 | known-host candidate | unknown | unknown | unknown | unknown | unknown | unknown | not collected | unknown | 2026-06-28 | needs authorized/connectable access before OpenClaw audit |

Discovery notes:

- Tailscale CLI was not available on the local machine during this run.
- SSH config contained no explicit host aliases.
- One known-host-only candidate was discovered from local SSH known hosts, but BatchMode SSH was not connectable. Its real host/IP is intentionally not published.
- No additional OpenClaw installation was confirmed beyond `machine-01`.

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
- `machine-02` known-host candidate until an authorized/connectable access path exists.

## machine-02

Role: known-host candidate.

Status:

- Access class: known-host-only.
- Connectivity: not connectable through safe BatchMode SSH in this run.
- OpenClaw status: unknown.
- Published identifiers: none.

Reasoning:

This machine is included because the user requested coverage beyond the local OpenClaw instance. It is not treated as an audited OpenClaw installation because there is no current safe evidence that OpenClaw is installed or reachable there.

Next action:

- Provide an authorized SSH alias, Tailscale access, or sanitized OpenClaw aggregate output for this machine.
