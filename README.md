# Claws

OpenClaw operations notes, repair playbooks, and field-tested practices from real local usage.

This repository is intentionally documentation-first. It does not contain local runtime state, tokens, private transcripts, database files, or raw OpenClaw configuration dumps. The goal is to preserve reusable operational knowledge: what failed, how it was diagnosed, what was fixed, and what should be checked next time.

## What is inside

- [2026-06-27 cron repair case study](docs/case-studies/2026-06-27-openclaw-cron-repair.md): a concrete incident write-up covering model migration, cron failures, command conversion, and verification.
- [Cron operations runbook](docs/runbooks/cron-operations.md): a repeatable checklist for diagnosing and repairing OpenClaw scheduled jobs.
- [Final cron inventory](docs/reference/final-cron-inventory.md): the enabled scheduled jobs after optimization.
- [Ark model configuration notes](docs/reference/ark-model-configuration.md): how to keep scheduled jobs on Volcengine Ark without per-job drift.

## Core principles

1. Do not blindly disable failed scheduled jobs. First separate historical status from current configuration health.
2. Keep scheduled jobs bounded: short external timeouts, explicit fallbacks, limited output, and clear no-op behavior.
3. Avoid cron-level model overrides unless there is a deliberate exception. Prefer a global default model stack.
4. Use command cron for pure reporting and health checks. Keep agentTurn cron for tasks that need model reasoning.
5. Never publish raw local OpenClaw state. Share sanitized patterns, commands, and decisions instead.

## Current reference outcome

The 2026-06-27 repair ended with:

- 36 total cron jobs.
- 28 enabled cron jobs.
- 8 disabled cron jobs.
- 4 enabled command jobs.
- 24 enabled agentTurn jobs.
- 0 cron-level model overrides.
- Default model stack inherited from Volcengine Ark.

Historical `error` status remained on jobs that had not naturally re-run yet. This was intentional: status should be cleared by successful execution, not by hiding evidence.

## Safe GitHub usage

When a stale environment token shadows a valid keyring login, prefer this pattern:

```sh
env -u GH_TOKEN -u GITHUB_TOKEN gh api user
```

Use the same pattern inside scheduled jobs that rely on the GitHub CLI:

```sh
env -u GH_TOKEN -u GITHUB_TOKEN gh issue list --repo OWNER/REPO --limit 20
```

This avoids a bad `GH_TOKEN` causing `Bad credentials` even when `gh` keyring auth is valid.
