# Syncing this mirror

[`fsauter/pstack-antigravity`](https://github.com/fsauter/pstack-antigravity) is an Antigravity-CLI native mirror of upstream [`backnotprop/pstack`](https://github.com/backnotprop/pstack). It adapts pstack's skills, playbooks, and tools to run seamlessly with Google Antigravity CLI while preserving all 24 engineering principles, triage ladders, and verification protocols.

Two branches keep those edits safe:

- `upstream` tracks [`backnotprop/pstack`](https://github.com/backnotprop/pstack) `main` exactly, with no local edits.
- `main` is `upstream` plus this mirror's Antigravity-CLI adaptations and additions.

Never copy upstream files onto `main` directly. That erases the edits.

## Remotes

Ensure your git remotes are configured:

```bash
git remote -v
# origin    https://github.com/fsauter/pstack-antigravity (fetch & push)
# upstream  https://github.com/backnotprop/pstack.git (fetch & push)
```

If `upstream` is not yet configured:

```bash
git remote add upstream https://github.com/backnotprop/pstack.git
```

## Steps

1. Fetch and fast-forward `upstream`:

   ```bash
   git fetch upstream main
   git switch upstream
   git merge --ff-only upstream/main
   ```

2. Merge `upstream` into `main`. Git applies only what changed upstream and keeps the mirror's edits:

   ```bash
   git switch main
   git merge upstream
   ```

   A conflict means upstream changed a line this mirror also adapted. Keep upstream's intent and reapply the Antigravity-CLI native wording / tools.

3. Refresh the bundled Comment Sicko prompt, then check that no new Cursor/Claude-only instructions arrived:

   ```bash
   cp agents/comment-sicko.md skills/no-comments/references/comment-sicko.md
   git diff upstream@{1} upstream -- skills | grep -nE '\.cursor/|agent-transcripts|cursor-team-kit|create-skill|Task|/loop\s+\d'
   ```

   Rewrite any new hits using Antigravity-native equivalents:
   - `Task` / subagents → `invoke_subagent` (with `Subagents` array)
   - `/loop <cadence>` → `schedule` tool (e.g. `CronExpression`) or `/schedule`
   - `/loop until done` → `/goal`
   - UI / terminal verification → `control-ui` (Chrome DevTools MCP / `/browser`) and `control-cli` (`run_command` / `manage_task`)
   - Code cleanup → `/deslop` (code diffs) and `/unslop` (prose)
   - Brain transcripts → `<appDataDir>/brain/<conversation-id>/.system_generated/logs/transcript.jsonl`

4. Run tests:

   ```bash
   cd skills/poteto-mode/scripts && bun test
   ```

5. Push both branches to origin:

   ```bash
   git push origin main upstream
   ```
