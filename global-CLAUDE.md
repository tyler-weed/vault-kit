# Global instructions (every Claude Code session on this machine)

<!-- Put this file at ~/.claude/CLAUDE.md. Replace every <...> placeholder. Project and vault
CLAUDE.md files add to these rules; they never replace them. -->

## Where things live

- **Vault:** `<VAULT PATH>`. Plain markdown, opened in Obsidian. It is canonical for project status,
  decisions and next steps.
- **Workshop:** `<CODE PATH>`. Code lives here, never in the vault.
- Before any vault work, read `<VAULT PATH>/CLAUDE.md` and `_System/Vault Map.md`. If the vault is
  not reachable from this session, say so. Never skip it silently.

## Write back before you finish

When a session decides, builds or discovers anything, write it to the vault before ending, without
being asked:

1. **Project note:** Key Decisions, Open Questions, Next Steps; bump `last-updated`.
2. **`_System/Agent Memory/<slug>.md`:** any new repo path, ID or tool gotcha; update `MEMORY.md`.
3. **`Daily/YYYY-MM-DD.md`:** one `## Session` block (Tool, Projects touched, Summary).

Quick answers and one-off lookups need none of this.

## Anything dated or owed

- Something I must do by a date, a window, a check, a decision, or something I'm waiting on goes in
  the queue (a `Queue/` note) the moment it comes up. A dated line that lives only in a paragraph is
  how things slip.
- When it's done, close it in the same session. Where a note mentions a queued item, link it.
- Check the queue before telling me what's due. Something past its date is "still open?", not
  "overdue": I often finish things without logging them.

## Secrets

- Credentials live in a `.env` outside the vault and outside repos. Never inline a credential in a
  shell command (it ends up in the permission log). Never commit `.env*`.
- Run a secret scan before the first push of any repo. Never bypass a secret-scanning hook; ask.

## Before anything leaves this machine

- Never push, deploy, send or write to a production database without my OK in this conversation.
  First list the changed files with a one-line summary each.

## Keeping contexts apart (edit to suit)

- Nothing from `<PRIVATE AREAS, e.g. personal finances, side projects>` goes to `<WORK ADDRESS>` or
  into anything that could reach it.

## Scheduled jobs

- Before adding or moving a scheduled task, read the inventory in `<VAULT PATH>/_System/Context Pack.md` and update
  it in the same session.
- Every job writes a heartbeat on every run, including runs that find nothing, so silence is
  visible on the dashboard.

## Which Claude surface fits

- Recurring work: a scheduler. Connector-driven work or Office documents: a desktop agent.
  JS-heavy web UI: a browser agent. Code and files: Claude Code.

## Environment

- `<OS, default shell and its quirks, what is and isn't installed>`
