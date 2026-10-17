---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## 1. Locate the active transcript

Before fanning out, find this conversation's transcript in the active workspace's session directory. Do not glob across every project's directory. That crosses workspace boundaries and reads private chats from unrelated projects.

In Antigravity CLI, the active conversation's transcript is located at `<appDataDir>/brain/<conversation-id>/.system_generated/logs/transcript.jsonl` (or `transcript_full.jsonl`). Use the active conversation's transcript file directly. If no path resolves, write a tight digest of the session and pass that instead.

## 2. Spawn three reviewers in parallel

Spawn three reviewers concurrently using Antigravity's `invoke_subagent` tool (`TypeName: 'self'`, `Workspace: 'inherit'`). Reviewers have MCP access to look up the tickets, chat threads, and observability traces the transcript references. Each reviewer and the synthesizer can also read its role line from `~/.agents/pstack-models.md` if present, per the Subagents section of the **poteto-mode** skill.

| Lens | Role | Default `model` | Prompt template |
|---|---|---|---|
| Judgment | `'Reflect Judgment Reviewer'` | `'pro'` | `references/judgment-reviewer.md` |
| Tooling | `'Reflect Tooling Reviewer'` | `'flash'` | `references/tooling-reviewer.md` |
| Divergent | `'Reflect Divergent Reviewer'` | `'pro'` | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the transcript path or digest where marked. Reviewers return findings in their response body.

## 3. Synthesize

Spawn one synthesizer subagent using `invoke_subagent` (`TypeName: 'self'`, `Role: 'Reflect Synthesizer'`, `Model: 'pro'`). It spot-verifies citations through MCP. Pass `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. It returns an Accepted / Rejected / Backlog list.

## 4. Structural enforcement check

Move any Accepted item that a lint rule, script, metadata flag, or runtime check would enforce more reliably to Backlog. See the **encode-lessons-in-structure** principle skill.

## 5. Apply

Present the synthesizer's full Accepted / Rejected / Backlog output and wait for explicit approval before applying any Accepted edit. The user picks the subset and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

File each Backlog item to your team's devex or backlog tracker without waiting. Only the Accepted list waits for approval.

Follow each approved row's Routing exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): the parent does it directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): follow Antigravity's **agy-customizations** skill (`docs/skills.md`) and edit the target skill in `.agents/skills/<skill-name>/SKILL.md` (or the plugin's `skills/<skill-name>/SKILL.md`).
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): update the YAML frontmatter `description` of the target skill per **agy-customizations**.
- `new skill: <kebab-name>`: create `.agents/skills/<kebab-name>/SKILL.md` following the **agy-customizations** skill format. Do not invent the shape ad hoc.

Run `agy plugin validate` (or your environment's SKILL.md validator) on every touched skill before declaring done.

## 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
