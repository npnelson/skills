# Direct model smoke test

`run-direct-model-smoke.sh` is a deliberately narrow spike for comparing the
`generate-testability-wrappers` skill with Claude and Codex without consuming
GitHub Copilot credits. It runs the no-DI `AsyncLocal` scenario once through
eight isolated arms:

1. Claude baseline
2. Claude with only `generate-testability-wrappers`
3. GPT-5.6 Sol baseline and skill
4. GPT-5.6 Terra baseline and skill
5. GPT-5.6 Luna baseline and skill

It then makes one direct Claude Opus judge call at high effort. This is useful
for a quick directional result; it does not reproduce skill-validator's
position-swapped judging or statistical confidence calculation.

## Prerequisites in WSL

- `claude`, authenticated with `claude auth login`
- `codex`, authenticated with `codex login`
- `git` and `jq`

The runner checks both logins before consuming model usage.

## Run from Windows Command Prompt

```cmd
wsl bash ./eng/skill-validator/spikes/run-direct-model-smoke.sh medium
```

Use `low`, `medium`, `high`, `xhigh`, or `max` as the single argument. Results
go to `artifacts/direct-cli-eval/<timestamp>-<effort>/` and include raw CLI
output, final responses, stderr, per-arm metadata, the judge result, and all
isolated work directories. The summary reports per-arm wall time and total,
input, cached-input, and output tokens, plus full agent/judge totals.

The defaults are `claude-opus-5`, all three GPT-5.6 variants, and a high-effort
Opus judge. Environment variables can override them:

```bash
CLAUDE_MODEL=claude-opus-5 CODEX_MODELS="gpt-5.6-sol gpt-5.6-terra gpt-5.6-luna" JUDGE_EFFORT=high RUN_JUDGE=1 \
  bash ./eng/skill-validator/spikes/run-direct-model-smoke.sh medium
```

Set `RUN_JUDGE=0` to skip the judge call. This script intentionally fixes the
skill, fixture, prompt, rubric, and run count instead of becoming a second eval
framework.

If a CLI or network failure interrupts a run, resume it without repeating valid
arms:

```cmd
wsl env RESULTS_DIR=artifacts/direct-cli-eval/<timestamp>-medium RESUME=1 bash ./eng/skill-validator/spikes/run-direct-model-smoke.sh medium
```

Claude OAuth 401s are retried once automatically; the first response is kept as
`*.attempt1.raw.json` for diagnosis.
