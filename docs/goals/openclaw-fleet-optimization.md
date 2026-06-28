# OpenClaw Fleet Optimization Goal

This is the durable instruction file for a long-running OpenClaw exploration, repair, optimization, verification, and documentation goal.

It is intentionally safe for a public repository. Do not add raw local paths, hostnames, IP addresses, tokens, auth profiles, SQLite files, transcript contents, exact cron expressions, or raw command output.

## Outcome

Complete a current-state OpenClaw operations audit across all in-scope machines. Each in-scope machine must have a sanitized health record, verified high-confidence fixes, documented remaining blockers, and current evidence. The public `kt-aicoding/claws` repository must contain the reusable runbooks, templates, inventory, and completion audit without sensitive data.

## Context

Primary public repository:

- `kt-aicoding/claws`

Local/private evidence sources may include OpenClaw status, doctor, cron JSON, local memory notes, and GitHub CLI status. Use them as evidence, but never paste raw output into public docs.

Before continuing after a resume, read:

1. `README.md`
2. `docs/runbooks/cron-operations.md`
3. `docs/runbooks/fleet-openclaw-operations.md`
4. `docs/reference/machine-inventory-template.md`
5. Latest `docs/reference/fleet-inventory-*.md`, if present.
6. Latest `docs/audits/*completion-audit*.md`, if present.

## Scope

In scope:

- The local OpenClaw installation available to the current agent.
- Other OpenClaw machines only when the user has provided safe access or a pre-existing safe access path is available.
- The public `kt-aicoding/claws` documentation repository.

Out of scope unless explicitly confirmed:

- Unapproved remote machines.
- Reading, printing, committing, or copying raw credentials.
- Deleting or manually archiving transcripts, databases, or historical config.
- Batch-triggering cron jobs that write to GitHub, send chat messages, or otherwise mutate external systems.
- Production deployment, paid operations, or irreversible changes.

## Constraints

1. Do not blindly disable failed cron jobs. Separate historical status from current configuration health.
2. Use machine aliases such as `machine-01`; do not publish real hostnames, users, IPs, locations, or device identifiers.
3. For GitHub CLI, prefer `env -u GH_TOKEN -u GITHUB_TOKEN gh ...` so stale environment tokens do not shadow keyring auth.
4. Check dirty worktrees before editing. Stage and commit only goal-related files.
5. Never commit raw OpenClaw runtime state, SQLite databases, transcripts, auth material, or raw config.
6. Prefer safe local repair and read-only verification. Pause before destructive, paid, or irreversible actions.

## Milestones

1. Establish current-state baseline for every in-scope machine.
2. Repair high-confidence operational issues, especially Gateway/service health, model drift, cron shape, bounded fetches, and GitHub CLI auth behavior.
3. Verify Ark model path, cron scheduler readability, command cron behavior, GitHub CLI auth, channel health, and remote repository state.
4. Update public docs with sanitized inventory and audit evidence.
5. Run sensitive-data scans and complete a requirement-by-requirement audit.

## Per-machine loop

For each in-scope machine:

1. Assign a stable alias.
2. Collect current state with local commands such as `openclaw status --deep`, `openclaw doctor`, `openclaw cron list --all --json`, and GitHub CLI auth checks.
3. Record only aggregate fields:
   - OpenClaw version and channel.
   - Gateway health class.
   - Service manager health class.
   - Cron totals and type split.
   - Cron-level model override count.
   - Historical ok/error counts.
   - Default model family.
   - Issue classes.
4. Apply safe repairs where appropriate:
   - Restart Gateway when a service is installed but not loaded.
   - Remove accidental cron-level model overrides.
   - Convert pure report/reminder jobs to command cron.
   - Add timeout, fallback, and output caps to external fetches.
   - Normalize GitHub CLI calls to avoid stale environment tokens.
5. Verify with current evidence.
6. Update sanitized inventory and audit notes.

## Done when

- Every in-scope machine is completed, not applicable, or externally blocked with evidence and next action.
- The local OpenClaw installation has current verification for Gateway, service, cron aggregate, model inheritance, Ark model path, GitHub CLI auth, and channel/security health.
- Remaining issues are classified as fixed, intentional, safe-deferred, optional-provider, historical, or externally blocked.
- Public docs are updated and pushed.
- Sensitive-data checks pass on both working tree and reachable git history.
- A completion audit proves the goal requirements from current evidence.

## Verification

Use current evidence from:

- `openclaw status --deep`
- `openclaw doctor`
- `openclaw cron list --all --json`
- Ark model smoke test.
- `env -u GH_TOKEN -u GITHUB_TOKEN gh repo view kt-aicoding/claws`
- `git status --short --branch`
- `gitleaks detect --source . --redact --no-banner`
- `gitleaks detect --source . --no-git --redact --no-banner`
- Remote tree check for `kt-aicoding/claws`.

## If blocked

- If a remote machine cannot be accessed, record the alias, access class, attempted action, error class, required owner action, and whether the machine is out of scope or externally blocked.
- If provider auth is expired, record only provider class and reauth command. Do not inspect or print credentials.
- If doctor recommends orphan transcript archival, do not manually delete or rename files unless the user explicitly confirms. Record as safe-deferred.
- If GitHub push is blocked, keep local commits and record remote, branch, commit, error class, and next action.

## Final report

Report:

- Machine aliases, status classes, fixes, validators, and remaining blockers.
- Repository branch, commit, and remote verification.
- Sensitive scan results.
- Why any remaining issue is not a blocker for the current goal.
