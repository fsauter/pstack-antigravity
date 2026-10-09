---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes an always-applied rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write pstack's model settings, one model per role in `~/.agents/pstack-models.md` (or rely on Antigravity's native `inherit`, `flash`, and `pro` model tiers):

## Steps

### 1. Detect available models

Antigravity CLI provides native subagent models: `'inherit'`, `'flash_lite'`, `'flash'`, and `'pro'`. The primary agent model can be selected via `/model`.

### 2. Load current state

If `~/.agents/pstack-models.md` exists, read its role values as the current choices. Otherwise start from the default mapping (`flash` for explorers/workers/refactoring, `pro` for synthesizers/cross-judges/reviewers).

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Use the native `ask_question` tool. Offer these options:

- `pro — deep reasoning for review, architecture, and synthesis`
- `flash — fast, lightweight exploration and mechanical edits`
- `inherit — match the parent conversation model`

**(b) Show the roles and confirm.** Show each role with its model tier using `ask_question`.

### 4. Validate and Write

Write `~/.agents/pstack-models.md`:

```markdown
# pstack model configuration. One line per role.
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
arena cross-judge pool: pro
swarm workers: flash
architect runners: pro, flash
interrogate reviewers: pro, flash
```

### 5. Confirm

Tell the user `~/.agents/pstack-models.md` was written. Antigravity subagents will respect these model tiers during `invoke_subagent`.

### 6. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
