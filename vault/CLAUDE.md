# Vault instructions

This folder is a plain-markdown vault opened in Obsidian. It is the system of record: the Cockpit
dashboard and every scheduled task are views and writers over these notes (BUILD-SPEC D-1).

## Three layers

- **Vault** (this folder): notes, decisions, people, systems, tasks. Canonical.
- **Brief**: a scheduled task that writes one file a day into `_System/Brief/`. Optional.
- **Workshop** (`<CODE PATH>`): code and repos. Code never lives here.

Test: if it builds something, it belongs in the Workshop. If it's a note or a deliverable you would
cite, it belongs in the Vault.

## Entry points

- `_System/Vault Map.md`: where things live. **Read it first, navigate by it, never glob or bulk-read.**
- `_System/Waiting On Me.md`: the queue's rules and frontmatter schema.
- `_System/Context Pack.md`: connector state and the scheduled-task inventory. Read it before any
  task that depends on a connector.
- `_System/Agent Memory/MEMORY.md`: read it before any task that touches a repo, build, connector or
  git operation. It's a cheap index of gotchas that are expensive to rediscover.

## Sources of truth

| Question | Lives in |
|---|---|
| project state, decisions, next steps | `Projects/` |
| who someone is | `People/` |
| how an external system works | `Systems/` |
| what someone waits on | `Queue/` |
| meeting purpose and outcome | `Meetings/` |
| repo paths, IDs, tool gotchas | `_System/Agent Memory/` |
| what happened on a day | `Daily/` |

Canonical notes win over Agent Memory when they disagree.

## Never read

- `Personal/`. Ever, for any reason.
- `_Attachments/`, `Artifacts/`, `Archive/`: read the project note first, and open a file there only
  when that note names it.

## Standing rules

0. Read the Vault Map first; navigate by it.
1. Never delete or silently overwrite a note. Append, or edit the specific section that changed.
   Historical notes (`Daily/`, `Archive/`) record what was true when written; don't rewrite them.
2. Decisions and next steps tied to a project go in that project note (Key Decisions, Open
   Questions, Next Steps, `last-updated`), not only in Daily.
3. A new initiative with no note gets one from `_Templates/Project.md`. Templates are
   specifications to read, not scripts to run.
4. To archive, **move** the note into `Archive/`. Never set `status: archived`.
5. Anything that needs doing is a `Queue/` note or a Next Steps bullet. Never a checkbox in `Daily/`.
6. Link liberally with path-agnostic wikilinks: `[[Note Name]]`, never `[[Folder/Note Name]]`.
7. Factual, concise, no filler.
8. Never remove `focus` from a project without asking.

## Session logging (mandatory)

At the end of every session that decided, built or discovered anything, write to all three:

1. The project note: Key Decisions / Open Questions / Next Steps; bump `last-updated`. Optionally
   set `resume` / `resume-next` so the dashboard card says where you left off.
2. `_System/Agent Memory/<slug>.md` for any new path, ID or gotcha; update `MEMORY.md`.
3. `Daily/YYYY-MM-DD.md`, one block in exactly this shape (the dashboard parses it):

```markdown
## Session <title>
**Tool:** Claude Code
**Projects touched:** [[Project A]], [[Project B]]
**Summary:**
- First bullet is what the dashboard shows.
```

Do it without being asked. It's the last step of the session, not an offer.

## Secrets

Credentials live in a `.env` outside the vault. Never inline one in a shell command, never hardcode
one, never bypass a secret-scanning hook.

## Monitoring principle

A monitor must detect silence, not just backlog. Every feed writes a heartbeat on every run, even
with zero results.
