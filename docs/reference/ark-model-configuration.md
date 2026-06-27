# Ark Model Configuration Notes

This note records the model configuration strategy used during the OpenClaw cron repair.

## Goal

Scheduled jobs should consistently use the Volcengine Ark model stack unless a job has a documented reason to use something else.

## Desired shape

Global defaults:

- Primary model: `volcengine/ark-code-latest`.
- Fallback models: Ark-compatible fallbacks.
- Cron job model overrides: `0`.

Cron jobs should inherit global defaults instead of storing their own model value.

## Why remove per-job overrides

Per-job model overrides create silent drift:

- Old jobs keep using deprecated providers.
- Disabled jobs can become broken when re-enabled later.
- Debugging becomes harder because scheduler behavior differs per task.
- Migration to a new provider is incomplete unless every job is audited.

Removing overrides makes the actual model path easier to reason about.

## Verification commands

List cron jobs with explicit model overrides:

```sh
openclaw cron list --all --json \
  | jq '[.jobs[] | select(.payload.model != null) | {name, model: .payload.model}]'
```

Expected result:

```json
[]
```

Run an explicit gateway smoke test:

```sh
openclaw infer model run --gateway --model volcengine/ark-code-latest
```

Expected behavior:

- The model call succeeds.
- The provider shown by logs/status is Volcengine Ark.
- The command returns a known marker if using a marker prompt.

## Expired non-Ark auth

Expired OpenAI or Qwen auth profiles do not block Ark-based cron execution. Reauthenticate them only if they are still expected to be used:

```sh
openclaw models auth login --provider openai
```

For legacy Qwen portal profiles, follow the current OpenClaw reauth flow printed by `openclaw doctor`.

## Migration checklist

- [ ] Confirm the global default model is Ark.
- [ ] Confirm Ark fallback models are configured.
- [ ] Remove cron-level `payload.model` values.
- [ ] Verify enabled and disabled jobs both have no overrides.
- [ ] Run at least one Ark smoke test.
- [ ] Let write-path jobs validate naturally or run one narrowly scoped manual test.
