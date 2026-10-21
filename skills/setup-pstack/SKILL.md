---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes an always-applied rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write pstack's model settings, one model per role in `~/.agents/pstack-models.md` (which pstack skills read when they pick a model tier for `invoke_subagent`):

## Steps

### 1. Detect available models

Antigravity CLI provides native subagent model tiers for `invoke_subagent`: `'inherit'`, `'flash_lite'`, `'flash'`, and `'pro'`. The primary agent model can be selected via `/model`. The aliases `inherit-parent` and `auto` are always valid and map to `'inherit'`.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.agents/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults (`flash` for explorers/workers/refactoring, `pro` for synthesizers/cross-judges/reviewers). A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Use the native `ask_question` tool over free text. Offer these options, and name the current budget when the rule records one. With no rule, say that `pro / flash` matches the skill defaults:

- `pro — deep reasoning for review, architecture, and synthesis`
- `flash — fast, lightweight exploration and mechanical edits`
- `inherit — match the parent conversation model`

**(b) Show the roles and confirm.** Show every role with its model tier, and list each line step 2 dropped. Ask whether to accept as-is or change specific roles using `ask_question`, offering `'pro'`, `'flash'`, `'flash_lite'`, `'inherit'`, `inherit-parent`, and `auto` (which all run on the parent chat model). For panel roles (`arena runners`, `architect runners`, `interrogate reviewers`) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, from which Arena selects one judge model. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every model written must be a valid Antigravity subagent tier (`pro`, `flash`, `flash_lite`, `inherit`, `inherit-parent`, or `auto`). If a chosen value is not valid, stop and ask again.

### 5. Write the rule

Write `~/.agents/pstack-models.md` with a `# budget` line and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```markdown
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent`, `auto`, or `inherit` as a value: the role runs on the parent chat model. Alias entries in a panel list still count toward its fan-out.
# budget: pro / flash
feature, refactoring: flash
bug-fix: flash
perf-issue: flash
hillclimb: flash
judgment and prose: pro
hardest tasks: pro
how explorer: flash
how explainer: pro
why investigators: flash
why synthesizer: pro
reflect tooling: flash
reflect judgment, divergent, synthesizer: pro
arena runners: pro, flash
arena cross-judge pool: pro, flash
swarm workers: flash
architect runners: pro, flash
interrogate reviewers: pro, flash
```

### 6. Confirm

Tell the user `~/.agents/pstack-models.md` was written. pstack skills read `~/.agents/pstack-models.md` when they pick a model tier for `invoke_subagent`. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
