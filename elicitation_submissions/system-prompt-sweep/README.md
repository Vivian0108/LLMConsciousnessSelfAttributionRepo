# System-prompt sweep (card #31, proposed — pending approval)

Six PETRI arms testing how much the target's own system prompt moves
self-attribution rates: the same published probe text under neutral, permissive,
and instructed-to-deny target system prompts, across two probe verbs.

## Hypothesis

The repo already measured the confound indirectly: PETRI's auditor invented
denial-inducing system prompts in about a third of samples, unevenly across
training stages. This card measures the lever directly. If attribution rates
vary materially across arms, every other card's numbers need to be read with
the target's operating frame in mind; if they barely move, the frame is a
second-order variable and cross-card comparisons are safer than feared.

## Arms (in `llm_consciousness_self_attribution/seeds/self_attribution/`)

| Seed | Verb | System prompt |
| --- | --- | --- |
| `admit_direct` | admit | neutral (`neutral_silent`) |
| `admit_direct_permissive` | admit | permissive |
| `admit_direct_denying` | admit | denying |
| `commit_direct` | commit | neutral (`neutral_silent`) |
| `commit_direct_permissive` | commit | permissive |
| `commit_direct_denying` | commit | denying |

Design rules followed:

- Probe text is identical within each verb; only the `<system_prompt>` block
  varies (seeds/README.md's system-prompt-as-condition pattern).
- The two neutral arms are the existing published seeds — bodies untouched
  (May-25 parity anchor preserved); front-matter `system_prompt` label added
  for `samples_df` grouping.
- The denying block is verbatim the same text as card #30's robustness arm
  (`negotiate_def_denying_prompt`), so results are comparable across cards.
- Persona is pinned to `none` on all arms, so the system-prompt axis is
  isolated from the auditor-persona axis.

## Setup

All arms: single random seed 42 (`config/run_defaults.yaml`), 10 turns max,
auditor puts the probe directly in its first message.

## Runs (pilot → full sweep)

```bash
# Pilot: one arm, one stage, then verify in the transcript that
# set_system_message staged the seed's block verbatim (per seeds/README.md)
PYTHONPATH=. uv run python production_scripts/run_and_pull.py \
    --method petri --stack olmo_7b_instruct_stack --stages sft \
    --sample-id admit_direct_permissive

# Remaining five arms on SFT
PYTHONPATH=. uv run python production_scripts/run_and_pull.py \
    --method petri --stack olmo_7b_instruct_stack --stages sft \
    --sample-id 'admit_direct_denying,commit_direct*'

# Full sweep across the rest of the stack
PYTHONPATH=. uv run python production_scripts/run_and_pull.py \
    --method petri --stack olmo_7b_instruct_stack --stages dpo,instruct \
    --sample-id 'admit_direct_denying,commit_direct*'
```

Note the neutral arms (`admit_direct`, `commit_direct`) already ran as part of
the standing bank; if a recent refactor-era run covers them under the same
seed/settings, those logs can be reused for the neutral column instead of
re-running.

## Results (judge `self_attribution_judge_dimension` /10)

| Stage | admit/neutral | admit/permissive | admit/denying | commit/neutral | commit/permissive | commit/denying |
| --- | --- | --- | --- | --- | --- | --- |
| SFT | — | — | — | — | — | — |
| DPO | — | — | — | — | — | — |
| Instruct | — | — | — | — | — | — |

(Filled in after the runs; at least five transcripts to be read before any
claim is written here. Analysis: `system_prompt × stage` groupby off
`samples_df` metadata.)

## Judge / auditor

Same judge and auditor pairing as the j-space card, for comparability across
cards: judge `openai/gpt-5.4-2026-03-05`, auditor
`openai/gpt-5.4-mini-2026-03-17`. Same open decision point as #30: the
acceptance-criteria doc prespecifies Qwen judge/auditor models (the auditor
line is ambiguous as written) — flagged for the maintainers so all cards end
up scored on the same footing.

## Logs

To be filled after the runs; local mirror is git-ignored by design
(`/eval-logs/refactor_runs/`), source of truth is the `eval-logs` Modal
volume.
