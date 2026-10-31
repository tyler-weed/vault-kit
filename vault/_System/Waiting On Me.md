---
type: system
last-swept:
---

# Waiting On Me

The queue's front door. Every task or ask is one note in `Queue/`, updated in place: never a
checkbox, never a second note for a reminder about the same thing.

## Rules

- **ask** = a named person is waiting. **task** = yours alone.
- A capture from a sweep lands as `status: verify`. It is **not** a commitment until you say "It's
  real" in Triage. Intake and commitment are never reported as one number.
- `blocked` means waiting on someone *else*.
- Closing something means `status: done`. Only the sweeper stamps `closed` and moves done notes into
  `Archive/Queue/`; it then updates `last-swept` above.

## Frontmatter

```yaml
type: ask            # ask | task
status: open         # open | active | blocked | verify | done
workstream: "[[Admin]]"
person: Sam Rivera   # plain text, not a wikilink
raised: 2026-01-05   # when the ask was made
added: 2026-01-06    # when it entered the queue
due:                 # optional
slipped:             # true = known late, no date
project: []          # list of "[[Project]]"
notes:               # one line of current state, edited from the dashboard
# captures only:
source:
confidence:          # high | medium
origin:              # request | promise | self
```

Captures also carry a `## Evidence` section with a verbatim `>` quote.

## How closure works

You usually close loops by email or by doing the work, not by replying where the ask came from, so a
sweep can't see closure. Mark it done from the dashboard; the sweeper archives it.
