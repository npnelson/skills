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

## Skill-only noise and value matrix

`run-skill-only-matrix.py` runs five skill-enabled trials for each combination
of GPT-5.6 Sol, Terra, and Luna at medium and high effort: 30 Codex calls total.
The schedule interleaves the six cells, and five high-effort Opus 5 calls judge
one anonymized output from every cell per batch. Resulting fixture files are
included with the response so terse agents that edited the project are judged
on their actual work.

Run from Windows Command Prompt:

```cmd
wsl bash -lic "python3 ./eng/skill-validator/spikes/run-skill-only-matrix.py --runs 5"
```

The only additional WSL prerequisite is Python 3. Results include the schedule,
every raw response and work directory, blinded judge mappings and outputs,
per-run CSV data, and an aggregate Markdown/JSON report with score variance,
7/7 frequency, criterion-level misses, tokens, wall time, and the observed
quality/efficiency frontier. Each output is judged once, so this measures
end-to-end evaluation noise rather than separating agent noise from judge noise.

This experiment compares model/effort choices when the skill is present. It
does not estimate the skill's uplift over a no-skill baseline.
