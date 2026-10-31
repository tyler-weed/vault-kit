# Personal Vault + Cockpit Dashboard - Build Spec

*Version 1.0 · 2026-10-27 · by Tyler Weed · [tylerweed.com/vault-kit](https://tylerweed.com/vault-kit)*

**Purpose:** everything needed to stand up, from nothing, a plain-markdown vault run with Claude, plus a custom Obsidian plugin ("Cockpit") that serves as the daily dashboard over it. Written to be handed to a Claude Code session as the primary brief, or read by a person.

**How to use this file**

- Sections 1-3: the why and the shape. Read first.
- Sections 4-5: the vault contract (folders, note types, frontmatter).
- Section 6: the plugin contract (modules, model, writes, UI).
- Sections 7-9: optional automation (brief, capture sweeps, workspaces).
- Section 10: build order with a "done when" check per step.
- Sections 11-13: tests, adaptation points, known gaps.
- IDs (D-n decisions, G-n gaps) are stable references.

## 1. The Problem This Solves

The problem is RE-ENTRY. After a week on something else you forget where you left a project, so projects go stale and asks go unanswered. When work arrives from many directions, you need to see what else is live when you have a free hour.

The system answers four questions fast:

1. What am I working on right now, and exactly where did I leave it?
2. What did I drop, and what should I pick up next?
3. Who is waiting on me, and what did I promise?
4. Are the feeds that fill this thing actually still running?

It is NOT a task manager (no priorities, labels, boards), not a "second brain", not an AI chat. The plugin makes no LLM calls (D-5).

Why a plugin over markdown instead of Airtable/Notion/an app: the vault is the system of record, so the dashboard reads and writes the same notes your Claude sessions log to. A hosted database needs a connector for every read and write and cannot see your notes. The trade-off is real: desktop only, not always-on, no mobile.

## 2. Three Layers

| Layer | What it is |
|---|---|
| **Vault** | A local folder of markdown notes opened in Obsidian. Canonical. Business facts, decisions, people, systems, tasks. |
| **Brief** | A scheduled task that reads the vault plus live connectors and produces one daily summary. (Optional, Phase 2.) |
| **Workshop** | A separate folder for code and repos. **Code does not live in the vault; vault notes do not live in the workshop.** Finished deliverables are copied into the vault's attachments folder. |

Test for which layer: if it builds something, it is Workshop. If it is a note or a deliverable you would cite, it is Vault.

Keep the vault OUT of any cloud-sync folder (OneDrive/Dropbox/iCloud). Sync clients fight the file watcher and can leave credentials in synced files. Back it up deliberately instead (section 4.5).

## 3. Design Decisions (keep the reason with the rule)

Each came from a measured failure in the system this replaced.

- **D-1** Vault is canonical. The plugin and scheduled tasks are views and writers over it. A second store (a base, a state file) drifts.
- **D-2** One note per task, in Queue/. Project work stays a bullet under "## Next Steps". A Bases row is a file, so a table over checkbox lines is impossible; and an ask has a counterparty and a clock, which earns structured fields. Project work has neither; a note per step would spawn hundreds of stubs.
- **D-3** Hierarchy: Workstream -> Projects, Tasks, Meetings. Four to six hand-named workstreams. A project is optional under one. "Which of five areas?" is easy; "which of fifty projects?" is not, and many tasks have no project.
- **D-4** A project is something you engage Claude on. Work you only support is archived and its tasks stand alone. Otherwise the project list fills with other people's initiatives and every one reads as "stale".
- **D-5** No LLM calls from the plugin. Intelligence arrives precomputed from scheduled runs. An API key would have to live in the vault, which the secrets policy forbids.
- **D-6** "Last worked" comes from logged sessions and Park, NEVER last-updated. Agents bump last-updated when tidying; one audit bumped 14 notes at once and made a 46-day-old project look fresh.
- **D-7** Daily blocks written by scheduled tasks, and blocks that touch more than 5 projects, do not count as work. A brief LISTS projects it only mentioned.
- **D-8** A resume note written during Review does not count as work (resume-via: review). "reviewed" never counts as work. Triage is a decision about a project, not work on it.
- **D-9** Now is capped (default 3) and the cap REFUSES rather than warns. The point of Now is to choose.
- **D-10** Park defaults to taking the project out of Now (parking means switching away), and every park also appends a dated line to "## Resume log" so a frontmatter-only resume point never silently overwrites the last one.
- **D-11** Pick up ranks "someone is waiting" first, then longest untouched, in bands: going stale (21+ days), cooling (7+), recently active.
- **D-12** Stale backlog is cleared by Review and YOU decide every item: one key each, with undo. Each item carries a suggested action and its reason. Auto-hiding or auto-sorting was rejected: you must own every drop.
- **D-13** Captures land as status: verify. Only your "It's real" makes one open. Intake and commitment are never reported as one number (a queue of "37 waiting" was really 30 unread plus 7 real).
- **D-14** Classify conversations, not messages. An ask is a candidate only if the whole thread still shows it open. A channel-based test kept 32 tasks and 24 were dropped on review: precision, not routing, was the defect.
- **D-15** Recall over precision when starting capture. Over-capture, then tune. You usually close loops by email or by doing the work, not by replying in the same chat, so a sweep cannot see closure.
- **D-16** A monitor must detect SILENCE, not just backlog. Every feed has a freshness light with an "off" state. A count of unprocessed items cannot distinguish "nothing arrived" from "everything handled"; a dead capture pipe once reported healthy for 44 days.
- **D-17** Archive = MOVE the note into Archive/. Never status: archived. Location is the signal; a status-archived note vanishes from views only if every view remembers to exclude it.
- **D-18** A private folder (Personal/) is invisible to the plugin and to agents. The plugin filters it out of files() and refuses read(). Private files are protected by code, not only by instruction.
- **D-19** Plugin writes are confined to Projects/, Queue/ and Meetings/ (links only), plus new Queue/ notes. Nothing is ever deleted. Edits use processFrontMatter or vault.process with exact-line re-verification.
- **D-20** "person" is plain text, not a wikilink (people without a note would produce broken links on day one).
- **D-21** The brief may write into the vault (_System/Brief/) but never edits Queue, Project or Daily notes. A brief that only talks to a chat window is never read twice.
- **D-22** Sweeps run locally (OS scheduler -> headless claude), not as a cloud routine. A cloud routine cannot write to, or de-duplicate against, a local vault.
- **D-23** Phase order: dashboard, then brief, then sweeps. Re-entry is answerable from vault data alone; capture was the part that failed before.
- **D-24** The read model is pure (no obsidian import) and exercised from Node against a real vault. It is the only way to test the logic without launching Obsidian.
- **D-25** Never delete or silently overwrite a note. Append, or edit the specific section that changed. Historical notes (Daily/, Archive/) record what was true when written; do not rewrite them to match the present.
- **D-26** Agents write to the vault at the end of every session that decided, built or discovered anything: three targets, all three, every time (section 5.1). A session logged only to the daily note is how 82 consecutive entries came to read "Decisions made: -".

## 4. Vault Specification

### 4.1 Folder structure

```text
<vault>/                 local folder, NOT cloud-synced
  CLAUDE.md              agent operating instructions (5.1)
  Cockpit.md             optional Bases version of the dashboard
                         + "Standing context for the brief"
  Home.md                optional Dataview dashboard (tag-driven)
  Projects.base  Queue.base  Meetings.base  People.base
  Workstreams/           4-6 notes, type: workstream
  Projects/              one note per project, type: project
  Queue/                 one note per task, type: ask | task
  Meetings/              curated dated notes, type: meeting
  People/                one note per person, type: person
  Systems/               one note per external system/tool
  Investigations/        optional, type: investigation
  Daily/                 YYYY-MM-DD.md, a LOG (never a queue)
  Archive/  Archive/Queue/   retired work; swept closed tasks
  Artifacts/             standalone HTML prototypes, workbooks
  _Attachments/          FINISHED deliverables only
  _System/
    Vault Map.md         where things live, truth hierarchy
    Context Pack.md      connector state, scheduled-task inventory
    Waiting On Me.md     queue front door: rules, schema, housekeeping
    Capture Runbook.md   procedure for sweeps (section 8)
    Backup Status.md     optional, feeds the Backup light
    Sweeps/              heartbeat files ONLY
    Brief/               YYYY-MM-DD*.md (Phase 2)
    Agent Memory/        agent scratchpad: repo paths, IDs, gotchas + MEMORY.md index
    Handoffs/            optional
  _Templates/            SPECIFICATIONS, read not executed
  Personal/              private, never read by anything
  .obsidian/             app.json (showUnsupportedFiles: true),
                         core-plugins.json (bases on), snippets/cockpit.css
```

The plugin EXCLUDES from every read: Personal/, .trash/, .obsidian/, _Templates/, _Attachments/, Artifacts/, and all of Archive/. It reads exactly four locations under _System/ for the freshness lights: Sweeps/, Brief/, Waiting On Me.md, Backup Status.md. Nothing else in _System/.

Turn on "show unsupported files" or .xlsx/.csv/.html/.pptx stay hidden in the explorer (this once hid 9 of 11 files in a folder).

### 4.2 Note types and frontmatter

Controlled vocabularies are enforced by convention, not by Obsidian. A value outside a list makes the note appear in NO filtered view, and Bases inline editing makes a typo one keystroke away. The plugin surfaces a warning for an out-of-vocabulary project status.

#### PROJECT (`Projects/<Name>.md`)

| Field | Meaning |
|---|---|
| `type` | project (required) |
| `status` | active \| blocked \| winding-down \| paused \| complete |
| `status-detail` | free text: what the status means now |
| `workstream` | `"[[<Workstream>]]"` (required on live projects) |
| `aliases` | every name people use for it (tool, repo, nickname). Do NOT alias a term two projects share. |
| `parent` | name of a parent project, ONLY if subordinate |
| `owner`, `started`, `last-updated`, `tags: [project]` | `last-updated` is a FALLBACK only (D-6) |
| `focus` | true puts it in Now. Written by the plugin. |
| `resume`, `resume-next`, `resume-date` | the Park resume point. Plugin writes; agents may set resume / resume-next at session end (better cards). |
| `resume-via` | "review" when written during Review. Plugin. |
| `reviewed` | YYYY-MM-DD; quiets it in Review for 21 days. Plugin. |
| `archived` | YYYY-MM-DD, set just before the move to Archive/. Plugin. |

Rule: agents never remove focus without asking.

Body sections the plugin reads:

- `## Next Steps`: top-level unchecked `- [ ]` items. The section runs to the next heading of the same or higher level, so `### Sub` headings inside are included. Items under a heading containing "resume here" rank first. Fenced code is skipped.
- `## Resume log`: written by Park, newest first.

Recommended other sections: Overview, Key Decisions, Open Questions, Open asks (an embedded Base, see 4.4), Systems/Tools, Related.

#### TASK / ASK (`Queue/<short imperative>.md`)

| Field | Meaning |
|---|---|
| `type` | ask (a named person is waiting) \| task (yours alone) |
| `status` | open \| active \| blocked \| verify \| done. blocked = waiting on someone ELSE. verify = unconfirmed capture, NOT a commitment, never counted with the queue. This list is separate from project status. |
| `workstream` | required (inherit from the project if there is one) |
| `person` | plain text (D-20) |
| `raised` / `added` | when the ask was made / when it entered the queue (the gap is capture latency) |
| `due` | optional. `slipped: true` = "known late, no date" |
| `project` | list of `"[[Project]]"`; only live projects count |
| `notes` | one-line inline state, edited from the dashboard |
| `snooze`, `reviewed`, `resolution` (done\|dropped) | written by the plugin |
| `closed` | stamped ONLY by the sweeper, never by plugin or you |
| captures only | `source`, `confidence` (high\|medium), `origin` (request\|promise\|self), and a body `## Evidence` section with a `>` verbatim quote |

Closing something means setting status: done.

#### WORKSTREAM (`Workstreams/<Name>.md`)

- type: workstream, optional status: retired, aliases, keywords. keywords match at >=2 characters, longest wins; keep generic words out.
- A note named exactly "Admin" is special: sorts last, neutral colour, no project count on its tile. Embed live Bases lists of the projects, tasks and meetings that link to it (4.4).
- A non-workstream hub (type: topic) can sit in the same folder as a linked concept that projects reference without being a top-level area.

#### MEETING (`Meetings/YYYY-MM-DD - Title.md`)

- type: meeting, date REQUIRED (the plugin ignores a meeting without one), optional start "HH:mm-HH:mm", organizer, prep, verdict, notes, project, workstream.
- CURATION RULE: a note exists only when there is something to say (prep is needed, a decision is expected, or you organize it). Measured once: of a busy week's calendar, only about one event in five had prep value; several were large all-hands standups. A note per event is write-more-than-you-read. A past meeting with an empty verdict is the meeting equivalent of an unswept queue; Meetings.base has a "Needs a verdict" view for it.

#### DAILY (`Daily/YYYY-MM-DD.md`)

- A log. Session blocks use:

```markdown
## Session <title>
**Tool:** ...
**Projects touched:** [[A]], [[B]]   (inline or bullet list)
**Summary:** first bullet is shown on the dashboard
```

- NO CHECKBOXES in Daily/, ever. Between two months, 321 unchecked follow-ups accumulated there at a 3.9% close rate because scheduled tasks wrote them and nothing read them. Anything that needs doing is a Queue/ note or a Project "## Next Steps" bullet. The test: an ask has a counterparty and a clock; project work has neither.

#### PERSON (`People/<Name>.md`)

- type: person, role, team, last-updated, aliases for alternate names.
- People coverage is the hard ceiling on any "who is waiting on me" logic: a person with no note cannot be surfaced by a person-based rule.

#### HEARTBEAT (`_System/Sweeps/YYYY-MM-DD HHmm <run>.md`)

- The Sweeps light reads the first 10 characters of the newest file NAME, so nothing but heartbeats may live in that folder. Format in section 8.

#### BRIEF (`_System/Brief/YYYY-MM-DD*.md`)

- type: brief, date. See section 7.

### 4.3 Invariants (check these periodically; a sweep or script can assert them)

1. Every live project and every open task has a workstream.
2. Every status is inside its vocabulary.
3. No Daily/ checkbox.
4. Nothing in _System/Sweeps/ except heartbeats.
5. Projects.base "Archived (should be empty)" view is empty.
6. An archived (moved) project has no focus.
7. No loose file without type: frontmatter at a folder root; flag, don't reason from it.

### 4.4 Conventions

- Note names: Title Case With Spaces.md.
- Path-agnostic wikilinks everywhere: [[Note Name]], never [[Folder/Note Name]]. Folders can then be restructured without breakage (a full restructure once moved every folder and broke zero links).
- type: on every canonical note, plus an identity tag (#project) if you use Dataview.
- Archiving: move into Archive/. Projects.base filters on file.inFolder("Projects"), so a moved note disappears by construction.
- Working artifacts vs finished deliverables: Projects/ may hold a project's LIVE working files beside its note (a workbook being iterated). _Attachments/ is for FINISHED deliverables. Test: is it still being edited?
- Embedded base forms: ![[Queue.base#Cockpit]] embeds a named view. A fenced ```base block inside a note with filter file.hasLink(this.file) lists "everything that links here", nothing to maintain. (Bases have no rollups and no backlinks; this is the substitute. "this" resolves to the embedding note.)
- Bases inline editing writes to frontmatter, which is exactly why the controlled vocabularies matter.
- Truth hierarchy (canonical wins over Agent Memory on conflict):

| Question | Lives in |
|---|---|
| project state, decisions, next steps | Projects/ |
| who someone is | People/ |
| how an external system works | Systems/ |
| what someone waits on | Queue/ |
| meeting purpose and outcome | Meetings/ |
| repo paths, IDs, tool gotchas | _System/Agent Memory/ |
| what happened on a day | Daily/ |

### 4.5 Backup and secrets

- The vault has no cloud sync, so schedule a backup and have it write _System/Backup Status.md with a table row "**Last run** | YYYY-MM-DD" and "**Result** | OK" (the Backup light reads exactly those).
- Credentials live in a .env OUTSIDE the vault/repo, or a per-user config folder. Never inline a credential in a shell command: it lands in the agent's permission-grant file, the one place nobody reviews. Never hardcode a token in source. Do not bypass a secret-scanning pre-commit hook; ask.
- Before any first "git push" of any repo, run a secret scan.

### 4.6 Bases views (the .base files are the spec; this is what each is for)

- **Queue.base**: filters on note.type (not the folder) so it spans Queue/ and Archive/Queue/. Views: Cockpit (open, not verify), Overdue (due < today OR slipped), By person, Undated, Newest intake, Needs verification, Board, Closed-awaiting-sweep, and three Resolved views (7 days, 30 days, all time) that read "closed" and reach into the archive: this is what answers "what did I finish last week?".
- **Projects.base**: Active, All, Blocked, Paused, Winding down, Complete, "Archived (should be empty)" as a deliberate invariant check, Board.
- **Meetings.base**: Upcoming, Today, Needs a verdict, All.
- **People.base**: All, By team, Stale (30+ days), Board.

Formula traps (cannot be verified outside Obsidian; check on screen):

- Date subtraction returns a Duration: use `(a - b).days`, never `/86400000`.
- A hyphenated property name parses as subtraction and fails SILENTLY: write `note["last-updated"]`.

Navigating by the file explorer feels bad by design (no status, recency or grouping). Use the Bases, the dashboard, or Ctrl+O.

## 5. Agent Operating Contract

### 5.1 Vault CLAUDE.md (put at vault root; Claude Code auto-loads it)

Contents to write:

- What the vault is; the three layers; where the root is.
- Entry points (Cockpit dashboard, Bases, Waiting On Me, Context Pack, Vault Map).
- Canonical sources of truth (the 4.4 hierarchy).
- Required reading before tooling work: Agent Memory MEMORY.md before any task that touches a repo, build, connector or git operation. It is a cheap index holding gotchas that are expensive to rediscover. (A session once recommended bypassing a security hook while asserting no risk; the note saying not to was indexed and one search away.)
- NEVER READ list: Personal/; any large index file (search it for a title, never load it whole); _Attachments/, Artifacts/, Archive/ (read the project note first, open a file only when that note names it).
- Standing rules:
  - (0) Read the Vault Map first; navigate by it, do not glob or bulk-read.
  - (1) Never delete or silently overwrite (D-25).
  - (2) Decisions/next steps tied to a project update that Project note (Key Decisions, Open Questions, Next Steps, last-updated), not only Daily.
  - (3) A new initiative with no note gets one from the Project template. Templates are specifications to READ, not scripts to execute.
  - (4) To archive, move into Archive/.
  - (5) Anything that needs doing is a Queue/ note or a Next Steps bullet, never a Daily checkbox.
  - (6) Link liberally, path-agnostic.
  - (7) Factual, concise, no filler.
- Secrets and credentials (4.5).
- Session logging, mandatory (5.2).
- Monitoring principle (D-16).

### 5.2 Session logging: three targets, every session that decided/built/found

1. Project note: Key Decisions / Open Questions / Next Steps; bump last-updated.
2. _System/Agent Memory/<slug>.md: any new repo path, ID, wrapper quirk or tool gotcha; update MEMORY.md.
3. Daily/YYYY-MM-DD.md: a "## Session" block per 4.2 (what the plugin's "last worked" logic reads).

Do it without being asked; it is the last step of the session, not an offer. Optionally set resume / resume-next on the project at the same time.

### 5.3 Global (user-level) CLAUDE.md

A second file in the user config folder that applies to every session regardless of working directory: where the vault and workshop are, "read the vault CLAUDE.md before vault work", the session-logging rule, the secrets rules, and a short "which Claude surface fits" note (recurring work -> a scheduler; connector-driven/Office docs -> a desktop agent; JS-rendered web UI -> a browser agent). If the vault is unreachable from a session, say so rather than silently skipping it.

### 5.4 Two memories, deliberately separate

The tool's own per-project auto-memory is NOT the vault's Agent Memory. Do not sync them automatically (the vault has a rule against unreviewed automated writes). Auto-memory is stored per working folder, so a session started in a repo folder cannot see the home-folder index; the vault's Agent Memory MEMORY.md is the shared source both surfaces read on purpose.

### 5.5 Templates

Four plain markdown skeletons, each carrying its own fill rules: Project, Person, Meeting, Daily-log. Agents read them as specifications. (A prior scripted-template plugin was retired once content became almost entirely agent-authored.)

### 5.6 Core system notes to write

- **Vault Map**: where things live, the three layers, folder table with a "source of truth?" column, Never Read, truth hierarchy, where NOT to search, conventions, operating principles (below), known gaps.
- **Waiting On Me**: the queue's front door: rules, the Queue/ frontmatter schema, housekeeping, "How closure works", and a last-swept date.
- **Context Pack**: connector state, scheduled-task inventory, session read order, standing cautions. Read before any task depending on a connector.

### 5.7 Operating principles

Learned from a system review; the reasons behind much of the above.

1. A monitor must detect silence, not just backlog.
2. Never report intake and commitment as one number.
3. Route by confidence. Low and High confidence must not land in the same queue (four of five Low items were things already dismissed).
4. One entry per ask, updated in place. A new checkbox per reminder inflated one ask into five.
5. People coverage is the ceiling on "who is waiting on me" logic.
6. Classify conversations, not messages.

## 6. Plugin Specification (Obsidian plugin id: cockpit)

### 6.1 Platform

- TypeScript, bundled by esbuild to CommonJS main.js. No UI framework: vanilla DOM using Obsidian's createDiv/createEl and its CSS variables, so any theme works. One styles.css; classes prefixed "ck-" (plus "cockpit" on the tab root and "cockpit-park" on the Park modal).
- manifest.json: id cockpit, minAppVersion 1.5.0, isDesktopOnly true. Bases views in the vault need Obsidian 1.9+.
- Dev deps: obsidian, esbuild, typescript, yaml, builtin-modules, @types/node.
- Scripts: dev (watch+deploy), build, deploy (build then copy main.js, manifest.json, styles.css into <vault>/.obsidian/plugins/cockpit/), typecheck (tsc --noEmit), smoke (bundle model for Node, run vs a vault).
- Vault path for deploy and smoke from an env var (COCKPIT_VAULT) with a sensible default.
- No network calls, no telemetry, no LLM, no other plugin required. Optional in the vault: Meta Bind (inline INPUT[...] fields in notes) and Dataview.
- Settings persist via loadData()/saveData() (data.json, gitignored).

### 6.2 Module layout and the purity rule (~2,500 lines total)

| Module | Role |
|---|---|
| `types.ts` `dates.ts` `parse.ts` `model.ts` | **Pure** read model. No `obsidian` import. `buildSnapshot(Source, today, ModelSettings) -> Snapshot` |
| `actions.ts` | **Every** write. Imports `obsidian`. |
| `view.ts` (~650 lines) | The dashboard tab (`ItemView`) |
| `review.ts` | Review and Triage (`Modal`) |
| `modals.ts`, `picker.ts` | Park, quick-add, workstream/project pickers, `LinkDetector` |
| `main.ts` | Plugin class, `Source` adapter, events, commands, status bar |
| `settings.ts` | Settings tab and defaults |
| `test/smoke.ts` | Node harness against a real vault |

`Source` is the ONLY door to the vault:

```ts
files(): FileRef[]            // { path, basename, mtime }
frontmatter(path): FM | null
read(path): Promise<string>
```

The adapter in main.ts filters Personal/ out of `files()` and throws on `read()`. The smoke test asserts nothing under Personal/ was read.

Key types (pure): Ask, Project, Workstream, Meeting, Resume, NextStep {text, line, raw}, SessionRef, Freshness {key,label,last,ageDays,state, detail}, ReviewItem {kind, path, suggest, reason, linkTo?}, Snapshot {today, projects, focus, pickUp{fresh,cooling,stale}, blocked, paused, untriaged, asks{all,overdue,open,mine,blocked,verify,snoozed, suggestedLinks,closedAwaitingSweep}, review, aliasIndex, workstreams, workstreamIndex, meetings{today,upcoming,needsVerdict}, freshness, brief, warnings}. ModelSettings defaults: coolingDays 7, staleDays 21, upcomingDays 7, taskStaleDays 30, reviewQuietDays 21, backfillDate "".

### 6.3 The read model (buildSnapshot)

All dates are plain YYYY-MM-DD local-calendar strings, no time zones. daysBetween uses UTC midnights so DST cannot shift a count. Parsed results (Next Steps, Daily sessions, titles) are cached by file mtime.

- **Session index.** Every Daily/YYYY-MM-DD.md is parsed for "## Session" blocks. A block is dropped if its title matches /morning brief|triage sweep|friday rollup|vault backup check/i or its **Tool:** matches /scheduled/i (CHANGE these patterns to match your own scheduled tasks), or if it lists more than 5 projects (D-7). Each kept block is indexed under every project it links.

**Per project**

| Value | Rule |
|---|---|
| `lastTouched` | max(newest session date, resume-date unless resume-via is review); else last-updated (D-6, D-8) |
| `staleDays` | today - lastTouched |
| `band` | stale if >= staleDays setting (21); cooling if >= coolingDays (7); else fresh; "unknown" if no date (grouped with stale) |
| `sessionCount` | number of kept blocks naming it |
| `backfillOnly` | last-updated equals the optional bulk-import date setting, no resume note, no later session |

Open Next Steps per 4.2.

- **Groupings.** Top-level = no parent. Now = focus: true, newest-touched first. Pick-up pool = top-level, not focused, status active or winding-down, not backfillOnly. Bands and order per D-11. Separate collapsed groups: Blocked, Paused, Untriaged backfill. A status outside the five values produces a warning.

**Per task** (Queue/ notes of type ask or task)

| Value | Rule |
|---|---|
| `overdue` | status open\|active AND (due < today OR slipped) |
| `liveProject` | first linked project that is OPEN (active, winding-down, blocked, paused; top-level). A link to a complete or archived project sets closedProject instead and the task counts as standalone ("was X"). |
| `workstream` | its own, else its live project's. If neither, workstreamGuess = keyword match over title + notes + first 1,500 chars of body. |
| `suggestion` | a project-link guess ONLY when it has no live project: whole-word match of any open project's name or alias in the same text; longest match wins; terms under 3 characters ignored (workstream matching allows 2, e.g. "ux"). |
| `stale` | status in open\|active\|blocked\|verify, quiet for >= taskStaleDays (30) since max(raised, reviewed), and no future due date |
| `snoozed` | snooze > today: hidden from queue and project ask lists |

- **Buckets:** overdue; open (non-overdue asks); mine (non-overdue type: task); blocked; verify; snoozed; suggestedLinks; closedAwaitingSweep. Sorted by due (undated last), then oldest first.

- **Meetings:** today; upcoming (after today, within upcomingDays); needsVerdict (past, empty verdict, newest first). Tasks/meetings inherit workstream from their live project.

- **Workstream tiles:** open tasks, overdue, live projects (not Admin), focus count, going-stale count, "worked X ago" = newest lastTouched among its open projects.

**Freshness lights.** States: ok \| warn \| stale \| off. "off" means the feed does not exist yet and is DELIBERATELY NOT RED (a missing feed is not a failure).

| Light | Reads | Thresholds |
|---|---|---|
| Session log | newest Daily/ file name | ok <=1d, warn <=4, else stale |
| Brief | newest _System/Brief/ file | ok today, warn <=3d, else stale; off if none exists |
| Sweeps | newest _System/Sweeps/ name | ok <=1d, warn <=3, else stale; off if none exists |
| Queue swept | last-swept in Waiting On Me | ok <=7d, warn <=10, else stale; off if the note is absent or last-swept is blank |
| Backup | "Last run" date and "Result \| OK" in Backup Status.md | ok <=8d and OK, else stale; off if the note is absent or Result is "not run yet" |

**Review queue** (buildReview). Projects first (oldest first), then tasks (oldest first). Items you "kept" are quiet for reviewQuietDays (21).

**Project items**: live (active / winding-down / blocked), not in Now, not quiet, and (backfill-only OR stale 21+ days).

| Condition | Suggested action |
|---|---|
| 1 or fewer logged sessions | archive ("looks like a workstream you support rather than a Claude project") |
| blocked | pause |
| has open asks | park (keep, leave a resume note) |
| untouched 45+ days, nobody waiting | pause |
| otherwise | park |

**Task items**: stale, not quiet, not snoozed.

| Condition | Suggested action |
|---|---|
| verify, or person is self / unnamed / inferred | drop ("never confirmed as real") |
| linked only to a closed project | done ("does it stand alone?") |
| no project but a suggestion exists | link |
| overdue | keep ("do it, snooze it or drop it") |
| 60+ days, no notes, no due date | drop |
| otherwise | keep |

### 6.4 The write surface (actions.ts)

`fileAt()` throws for anything not under `Projects/`, `Queue/` or `Meetings/`, or in an excluded path. **Every action returns an undo closure.**

| Action | What it writes |
|---|---|
| `createTask` | New `Queue/<safe name>.md` (" (2)" on collision): type: task, status: open, added/raised today, optional project and workstream, empty notes; body "# title" + "## Notes". |
| `linkToProject` | Sets project to the target plus existing links to still-open projects; sets workstream from the project; bumps last-updated. |
| `setWorkstream` | Sets workstream. |
| `markReviewed` | Sets reviewed to today. |
| `setProjectStatus` | paused / complete / active; sets reviewed; removes focus if not active. |
| `archiveProject` | Removes focus, sets archived, **moves** the note to `Archive/<name>.md` ("... (archived <date>).md" on clash). Links keep resolving (D-17). |
| `closeTask` | status: done + resolution: done/dropped + last-updated. |
| `setAskStatus` | Changes status (used for "It's real" → open). |
| `snoozeAsk` / `snoozeFor` | Sets or clears snooze. |
| `setFocus` | Sets or removes focus (the cap refuses, D-9). |
| `park` | Sets resume, resume-next, resume-date, last-updated, and resume-via: review **only** when called from Review; optionally removes focus. Inserts "- **<date>** - Left off: ... · **Next:** ..." as the **first** item under "## Resume log", creating the section at the end of the note if missing. |
| `setAskNotes` | Sets notes. |
| `completeNextStep` | Re-locates the exact line (by line number, or the single identical line anywhere); if it cannot, **refuses** with "That step changed since the dashboard loaded it". Otherwise rewrites `[ ]` to `[x]`, appends " ✅ *<date>*", bumps last-updated, preserves the file's line endings. |
| `reopenNextStep` | Undo for the above. |

### 6.5 UI specification

- **Commands** (palette): Open dashboard; Refresh dashboard; Review stale projects and tasks; Add a task; Triage captured tasks; Park this project and Add this project to Now / take it out (both enabled only when the active file is under Projects/). A ribbon icon (layout-dashboard) and a status-bar item "◧ N now · N overdue · N feeds stale" (tinted when anything is overdue) open the tab.

**Dashboard layout** (top to bottom)

1. Header: greeting + date; four stats (in Now x/max; waiting on you = overdue + open asks; overdue; going stale); the five freshness pills, each with an age and tooltip; a refresh button.
2. Workstream tiles: one per workstream. Click filters the whole dashboard to it; right-click opens the note; an "✕ All" button clears. The filter is applied once to the snapshot (scoped()); the tiles always use the unfiltered one.
3. Triage banner (if any verify tasks) and Review banner (if anything is stale), each with a button. A warnings strip for vocabulary problems.
4. Two columns. Main: Now, Pick up. Side: Tasks, Today, Brief. Every section is collapsible and remembers its state.

- **Now.** A card per focused project, left border coloured by band. Header: title (link), status chip, "worked X ago", workstream chip, "aka <aliases>"; tools: Park, take out of Now, open note. Blocks: "Where you left off" (resume text and "Parked <date>"; else last session summary and tool; else status-detail; else a prompt to park); "Next" (the resume-next line as "start with", then the first 3 Next Steps with real checkboxes; a link expands to 15); "Waiting on you here" (chips, red if overdue). Empty state tells you to pin projects from Pick up.

- **Pick up.** Bands Going stale, Cooling, Recently active (last one collapsed by default), then collapsed Blocked, Paused, Untriaged backfill. A row: expand chevron, name, age, a waiting-people badge, status chip if not active, workstream chip, aliases, and one hint line ("Start with: <resume-next>", else "Next: <first step>", else status-detail). Row tools: pin into Now, Park. Expanded: resume text, last session, status detail, 5 steps, waiting people.

- **Tasks.** Quick-add input, then: Overdue, Someone's waiting and Yours (each shows 8, "Show all N"); Needs a workstream (one-click guess + "Choose..."); Suggested links ("Link to X" / "Other..."); Needs verification (promise tag, confidence, inline note, "It's real" / "Dismiss"); Blocked on someone else and Snoozed (collapsed); footer "N closed, awaiting sweep to Archive".
- **A task row:** a circle button that marks it done (hover shows a check), title link, meta (person, "you promised" tag, workstream chip, "due <date>" or "slipped", age, snooze date, project chip or "was X"), an inline one-line notes field (click to edit, Enter saves, Esc cancels), tools (link, snooze menu: tomorrow, +3 days, next Monday, +2 weeks, clear). Every action raises a toast with Undo for 6 seconds.

- **Quick-add** (dashboard input and the Add a task command). As you type, LinkDetector shows two chips: a workstream (from a matched project, else a keyword match, else the active filter) and an optional project (alias match). Either can be changed with a picker or cleared with ✕. Enter creates; Escape clears.

- **Today.** Today's meeting notes (time, title, prep tag, project, notes, a "link" affordance if unlinked), plus collapsed "Next 7 days" and "Owed a verdict". Empty state explains Meetings/ is curated.

- **Brief.** Renders the newest _System/Brief/ file via MarkdownRenderer with frontmatter stripped. Empty state describes Phase 2.

- **Park modal.** Two textareas ("Where I'm leaving it", "Next time, start with"); a toggle "Take it out of Now" (shown only when focused, default on); Ctrl+Enter saves; empty submits refused. Opened from a card, a row, Review or the command.

- **Review and Triage** (one modal, two queues). One item at a time with a progress bar, a kicker ("Project", "Your task", "Someone's waiting", "Unconfirmed capture", "Unconfirmed promise: you said you'd do this"), the facts (where it was left, next step, tasks on it; for tasks the quoted evidence, source, a note box, context), the SUGGESTION and its reason, then buttons. ENTER TAKES THE SUGGESTION; the suggested button shows ↵.
  - Projects: N keep + note (opens Park), K keep, F into Now (refuses if full), P pause, C complete, A archive.
  - Tasks: D done, X drop, K keep (for a capture it reads "It's real" and sets open + reviewed), Z snooze 2 weeks, L link..., N edit note.
  - All: S skip (offered again), U undo last decision (full history in the session), Esc stop.
  - Items whose note moved or closed since the queue was built are skipped silently. Triage orders verify tasks newest first; suggestion is "It's real" for confidence: high, a "link" when an alias matched, else "It's real".

- **Settings** (defaults): open on startup true; focus limit 3 (slider 1-6); cooling after 7 days; stale after 21; tasks go stale after 30; quiet period after Keep 21; upcoming meetings window 7; your name (greeting); optional bulk-import date (blank = off).

- **Colour.** Workstreams get a colour by position: styles.css defines five slots (.ws-0 .. .ws-4) plus .ws-admin. With more than five non-Admin workstreams, add rules or the extras render with no accent.

### 6.6 Events and performance

- A debounced (400 ms) rebuild runs on metadataCache.changed and on vault create/delete/rename, ignoring Personal/. A building/again pair coalesces overlapping rebuilds. A 5-minute interval rebuilds when the date rolls past midnight.
- Rendering is a full re-render of the tab; it DEFERS while an input in the view has focus (so typing in a notes field is not wiped) and re-renders on blur. Scroll position is preserved.
- Reference numbers: ~55 projects builds a snapshot in roughly 120-230 ms cold.

### 6.7 Obsidian hazards found the hard way (each cost an hour or more)

1. NEVER name a view method after an ItemView/View/Component method. A private helper called open(path) overrode ItemView.open(containerEl). setViewState SWALLOWED the error: the tab appeared and even rendered but the container was never attached, so the screen was blank with nothing in any error list. Use openNote, openPark, etc.; check every new method against the prototype chain.
2. Obsidian auto-focuses the first input when a Modal opens. A note box in Review swallowed every one-key shortcut. Fix: a noteActive flag set only by the N key or a real click, plus an immediate blur() of the auto-focused box. Any modal with both an input and single-key shortcuts needs this.
3. The file watcher drops events on a burst of external edits. A script edited 92 notes; 36 were missed (vault.read showed new text while metadataCache and even stat.mtime stayed stale). A Node smoke test reads the disk so it passes anyway. After any bulk scripted edit, reload Obsidian and compare disk with cache.
4. Moving files externally while Obsidian runs is safe only with a reload afterwards.
5. Synthetic KeyboardEvents do not reach Obsidian's Scope handlers (untrusted). Test keys with trusted events over CDP, and NEVER let a UI test reach a save path on a real note: record the original value before the first keystroke and restore it exactly.
6. The dev screenshot command can lag one render behind a state change made via eval; wait 2-3 s or query the DOM.
7. processFrontMatter + CRLF: Python in Windows text mode writes \r\n, then the YAML parser reads "focus: true\r" as a string. Parse booleans with a trim, normalise line endings in the harness, and write vault files in BINARY mode preserving the file's own line ending.
8. A BOM in manifest.json (PowerShell 5.1 Set-Content -Encoding utf8) can stop Obsidian loading the plugin. Never let PowerShell 5.1 write JSON; write it from Python in binary mode.
9. Write code with an editor, not a shell heredoc (it eats backslashes).
10. A Bases row is a file: no rollup, no backlinks (see 4.4 for the embedded-base substitute).

## 7. Phase 2 - The Brief (optional)

A scheduled task that WRITES _System/Brief/YYYY-MM-DD.md each weekday morning; the dashboard's Brief section renders it (D-21).

Reads: vault CLAUDE.md, Vault Map, Context Pack, Cockpit.md "Standing context for the brief" (read FIRST; it outranks the brief's own inference; flag any line older than 14 days), open Queue/ notes and their notes field, Projects/ focus cards, today's calendar, closures since the last run.

Writes: exactly one new file per day, frontmatter type: brief, date. NEVER edits Queue/, Projects/, Daily/ or People/.

Sections, in order:

0. Closed since last time, with an explicit silence check (zero closures plus a last-swept older than 10 days is called out as a possibly dead loop).
1. Today's calendar with prep needs.
2. Overdue / Open / Needs verification as THREE SEPARATE numbers.
3. Resume digest per focus project (one paragraph from resume, resume-next, last session, open asks).
4. Suggested focus and why.
5. Feeds: what is stale or off.

It must state a gap plainly rather than report an empty list as a clean one.

Gotcha: if the brief runs in a desktop agent's scheduled task, that task's folder access is fixed when it is CREATED; editing the prompt never changes it. A brief created against one vault path and later pointed at another runs BLIND and says so only in its own output. Create it from a session that has the vault connected, run it once by hand, and confirm it quotes the Context Pack. A health check on the brief must look for missing sections, not only for a finished run.

Open item: no prep generator exists. The dashboard displays prep; nothing writes it into a meeting note, and a read-only brief cannot. A separate evening task that writes only where a curated meeting note exists is the likely answer.

## 8. Phase 3 - Capture Sweeps (optional)

What a sweep does. For a time window it reads your email (inbox and sent), chat and ticket tracker, finds REQUESTS made of you and PROMISES you made, and files each as a Queue/ note with status: verify, a verbatim "## Evidence" quote, source, confidence and origin, plus one heartbeat file. It never edits an existing note, never replies, never reads Personal/.

Rules that carry the quality (write them into _System/Capture Runbook.md, which the running session reads verbatim):

- Classify conversations, not messages (D-14). A question answered in-thread minutes later is closed. Judging that needs the thread.
- Confidence: low is listed in the heartbeat but NEVER filed.
- Never re-create something you dropped. Skip people who have left.
- At most 40 notes per run.
- Judge the home project from the thread and write a "Project because" line; you confirm in Triage. (Matching on words alone linked none of 15 captures; 8 belonged to a project.)
- Set due from any stated deadline.
- Capture your OWN notes-to-self (a chat with yourself) as type: task captures with origin: self.
- Capture promises (origin: promise, "I'll...", "I can help with...") as well as requests; forgotten promises are the core problem. The dashboard shows a "you promised" tag.

Heartbeat contents (written EVERY run, even with zero results, so silence is visible): run name, window-start, window-end, per-source status (ok | partial | failed, with reasons), counts (captured, held back as low confidence, duplicates), and what each source did NOT read.

**Run modes**

- Interactive backfill: a Claude session with the vault connected; window = a start date to now. Use it first, to calibrate.
- Scheduled am/pm: window = newest heartbeat's window-end to now, so a missed run catches up automatically.

Calibration results to expect (four interactive passes): precision good (every capture had a verbatim ask and an honest confidence); recall needs tuning (chat and email search are lossy); a promises pass over a few thousand chat messages, a few hundred sent emails and a handful of tickets found about 20 captures. Tune the runbook by what it got wrong. An early capture window should start at a sensible date; older items are dropped in Triage.

Non-obvious connector lessons (also put in the runbook):

- Never page the chat list. One run made 113 calls without finishing. Search your sent messages one DAY at a time instead.
- Date filters on chat/email may be UTC days; a local evening falls in the next day's search.
- A message-read endpoint may return only the most recent 50 messages.
- Ticket queries cannot find @-mentions.
- Give every sub-agent a hard time limit.
- Fan out one scan agent per source, with the PARENT writing the notes.

**Unattended runs: proven design** (the earlier version of this failed)

- Problem: a headless `claude -p` session was DENIED path-scoped `Write(Queue/**)` and `Write(_System/Sweeps/**)` allow rules under four different spellings, and probing permission modes would have meant bypassing a safeguard.
- Fix that worked: the headless session gets NO write access at all. It reads through the connectors (pre-approved read-only tools) and returns its findings as JSON between SWEEP_JSON markers. The runner script (PowerShell or Python) parses the payload and writes the Queue notes and heartbeat itself.
- Tested first against a synthetic payload (filenames, collisions, BOM and line endings, heartbeat window math), then confirmed with a real manual run.
- Parser lesson: the runner demanded an exact fence and failed silently; take the LAST marker and the OUTERMOST braces instead.
- Runner responsibilities: set the window from the newest heartbeat; call claude -p in the vault with a read-tool allowlist; kill it after a time limit; verify a heartbeat appeared; on failure post an alert so silence is visible; collision-safe filenames; write files in binary mode.
- Registration: two weekday jobs (e.g. 07:00 and 16:00) in the OS scheduler, current user only (no stored password), run-missed-job-when-available on.
- Stay on the local scheduler, not a cloud routine (D-22).
- Alerting: a webhook-based channel post (incoming-webhook style) with a short card is enough. Credentials for it go in a .env outside the vault. Starting without alerting and relying on the dashboard's Sweeps light is acceptable; revisit if the light goes unchecked.

## 9. Workspaces and Sync Between Vault and Code (optional, scales well)

Once you have several code projects, keep vault and workshop from drifting:

- One .code-workspace file per active project in a workspaces/ folder in the workshop, main folder FIRST (a session starts there). A README maps each file to its vault project note. A vault-only workspace hides Personal/.
- Every repo's CLAUDE.md has a "Vault sync" section pointing at its vault project note, saying: the vault note is canonical for status, decisions and next steps; read the vault CLAUDE.md, the project note and Agent Memory index before changing anything; decisions go in the vault not the repo.
- A folder-to-project map (in Agent Memory) so a session started in a repo can find its project note. Check the map BEFORE creating a project folder or note: every duplicate found in one audit started as a second name for an existing project (a formal name, a "-2" draft, a one-character ID typo). If a tool gets a new name, record it in the map; do not just rename.
- Naming rule: one project, one folder, one project note. Code only in workshop/<project>/. Anything shared with other people lives in a shared folder with no .git, node_modules or .env. Stale copies go to workshop/_archive/<date>/ (moved, never deleted).
- A read-only drift-check script (exit 1 on drift) run at the end of any session that adds/renames/moves/retires a project, and weekly from the OS scheduler.
- Auto-memory is per working folder, so repo sessions cannot see the home folder's index; the vault's Agent Memory MEMORY.md is the shared source.

## 10. Build Order (each step ends with a check you can run)

| # | Step | Done when |
|---|---|---|
| 0 | **Vault.** Create folders and the four templates; write CLAUDE.md and Vault Map; create 4-6 workstreams; convert 3-5 *real* projects into notes with type, status, workstream, aliases. Turn on Bases; show unsupported files. Create a Personal/ folder and an Agent Memory index. | Opening Projects.base lists the projects; no status is outside the vocabulary. |
| 1 | **Pure model, read-only.** types, dates, parse (links, Next Steps, Daily sessions, findMention, safeFileName), then buildSnapshot and a Node smoke.ts that walks the vault and prints the snapshot. Do this **before** any UI. | Smoke passes against a scratch copy **and** your real vault, takes < 1 s, and asserts nothing under Personal/ was read. |
| 2 | **Plugin shell.** main.ts with the Source adapter (Personal filter), settings, the view registered, ribbon icon, rebuild on change. Render only the header and Now. | The tab opens, shows your focus projects, and no Personal/ path is in the console. |
| 3 | **Now + Park.** Cards, Park modal, park/setFocus writes, undo toasts, the Resume log section. | Park a project: frontmatter and a Resume log line appear; undo restores. |
| 4 | **Pick up + next-step checkboxes.** Bands, ranking, row expansion, completeNextStep with re-verification. | Edit a Next Step line in another window, then click its box: it **refuses** rather than ticking the wrong line. |
| 5 | **Tasks.** Quick-add, LinkDetector, task rows, done, snooze, notes field, link. | Add a task by typing a project alias; it lands in Queue/ with that project and workstream. |
| 6 | **Workstreams.** Tiles, filter, chips, "Needs a workstream". | Filter to one workstream; every section narrows consistently. |
| 7 | **Review.** Queue builder, the modal, keys, undo, archiveProject. Run it with trusted keys. | Archive a project and undo: the note moves out and back and links never break. |
| 8 | **Freshness lights,** status bar, Today, Brief hook. | Each light shows "off" before its feed exists, never red. |
| 9 | **Phase 2.** Create the brief task from a vault-connected session. | The Brief section renders today's file; delete a section from the prompt and your check notices. |
| 10 | **Phase 3.** Run the runbook interactively as a backfill; Triage the output; tune. Then build the runner (section 8) and schedule. | You can say, from the heartbeat, what each source returned and did not read. |
| 11 | **Workspaces + drift check** (section 9), once you have 3+ code projects. | The drift check exits 0. |

Tip: ask Claude Code to build step by step from this file, one step per session, running that step's check before moving on. Set the first session up by giving it this file and an empty vault folder.

First-week habits that make the dashboard worth opening:

- Put 2-3 projects in Now and Park the first time you switch away. The resume cards are only as good as what gets parked.
- Run Review once to clear the stale backlog (about 10 minutes).
- Add aliases for the names people actually use.

## 11. Test Plan

1. `npm run typecheck` and `npm run smoke` (COCKPIT_VAULT set). Smoke covers the parsers (nested and fenced Next Steps, scheduled-block filtering, whole-word alias matching, filename safety) and structural vault invariants. It reads the disk, so it cannot see Obsidian's cache (6.7 #3).
2. Live check through the Obsidian CLI (enable under Settings -> General -> Advanced): `plugin:reload id=cockpit`, `command id=cockpit:open`, turn dev debug on FIRST (or console capture returns nothing), `dev:screenshot`, `dev:dom selector=.cockpit total`, and `eval` to run JS in the app. Reload after every deploy.
3. Write actions are the risky part. Exercise each against a SCRATCH COPY of the vault, never your real one, and confirm every undo.
4. Keyboard behaviour only with trusted CDP key events.
5. After any bulk scripted edit: reload Obsidian, then compare vault.read vs metadataCache for each changed file.
6. Bases formulas must be verified on screen; there is no evaluator outside Obsidian. Two formula bugs once parsed cleanly, passed a property audit and were still wrong.

## 12. Adaptation Points

| What | Where | Change when |
|---|---|---|
| Scheduled-task title regex | `SCHEDULED_TITLE` in parse.ts | your scheduled tasks log Daily blocks under other names |
| Excluded folders | `isExcluded` in model.ts and `PERSONAL` in main.ts | your private folder is not Personal/ |
| The Admin workstream | Admin.md; `isAdmin` regex in model.ts | you want another catch-all |
| Workstream colours | styles.css `.ws-0` to `.ws-4`, `.ws-admin` | more than five workstreams |
| Session block format | `parseDailySessions` | you log sessions differently (the "last worked" logic depends on it) |
| Thresholds | settings tab | your rhythm differs from 7/21/30/21 |
| Sources in the runbook | Capture Runbook section 2 and heartbeat/runner source keys | you do not use the same email, chat or ticket tools |
| Bulk-import date | settings | you imported old notes and want them held out of Pick up |
| Workstream names | Workstreams/ | always: choose your own 4-6 from your real areas |

Workstream discovery tip: rather than inventing them, cluster your existing projects (note links, shared sessions, shared people, TF-IDF vocabulary, greedy-modularity communities) and name the clusters. It produced six clusters that matched intuition; every project and nearly every task mapped to one.

## 13. Known Gaps And Open Questions

- **G-1** Phase 2 (the brief) is specified, not built; the dashboard hook exists.
- **G-2** No prep generator (section 7).
- **G-3** Undo is partial for pause/complete and archive: it restores status (or moves the note back and clears archived) but not reviewed or focus.
- **G-4** Desktop only, not always-on, no mobile.
- **G-5** Transcript retention in the agent tool (default ~30 days) deletes the only way to audit what a session did but did not log. Raising it also retains every credential ever pasted in a session; decide deliberately.
- **G-6** Projects.base and People.base were written fresh and not verified on screen in a clean vault; check formulas on first load.
- **G-7** A one-time local-only vault has no off-machine backup unless you build one (4.5).
- **O-1** Does the dashboard get opened daily? That is the whole bet. A previous check-off surface closed about five items a day for 27 days and died the day it stopped being opened. Measure it rather than assume it.
- **O-2** What belongs in the brief's per-project resume digest once the dashboard shows what the notes alone cannot?
- **O-3** Whether to add a second capture channel (a browser extension or form feeding a queue) for intake that is not email, chat or tickets.

## Appendix A - Field Ownership

| Field | Where | Written by |
|---|---|---|
| status, status-detail, workstream, aliases, parent, owner | Project | you / agents (Review writes pause/complete) |
| focus, resume*, resume-via, reviewed, archived | Project | the plugin (agents may write resume, resume-next; never remove focus without asking) |
| status, person, due, slipped, project, workstream, notes, raised, added | Task | you / sweeps (notes and status also from the dashboard) |
| snooze, reviewed, resolution | Task | the plugin |
| closed | Task | **only** the sweeper |
| source, confidence, origin, ## Evidence | Capture | the sweep |
| last-swept | Waiting On Me | the sweeper, when it archives closed tasks |
| heartbeat files | _System/Sweeps | the sweep |
| brief files | _System/Brief | the brief task |

## Appendix B - Glossary

| Term | Meaning |
|---|---|
| **Now** | Up to three focus projects. |
| **Park** | Record where you left off and what to do first, when switching. |
| **Pick up** | Live projects not in Now, banded by time since real work. |
| **Capture** | A task filed by a sweep as status: verify. |
| **Triage** | Confirming or dropping captures. |
| **Review** | One-key clearing of the stale backlog. |
| **Heartbeat** | The file every sweep writes, even with zero results. |
| **Standing context** | Dated steering lines in Cockpit.md that outrank a brief's own inference. |
| **Workstream** | Top-level area; every project, task and meeting belongs to one. |

## Appendix C - Starter Prompt For Claude Code

(Use this if you start without the kit's `vault/` folder; otherwise use the prompt in the README.)

```text
Read the attached build spec in full. Build it in the order in section 10,
one step per session. Start with step 0 in an empty folder I'll name: create
the vault structure, the four templates, CLAUDE.md, Vault Map, Waiting On Me
and Context Pack, and a Projects.base. Ask me for my 4-6 workstream names and
3-5 real projects. Do not start step 1 until I confirm the step-0 check
passes. Never read or write a folder named Personal/. At the end of every
session, log to the three targets in section 5.2.
```

---

*End of spec. Free to use and adapt. If it saves you a rebuild, I'd like to hear about it: [tylerweed.com](https://tylerweed.com).*
