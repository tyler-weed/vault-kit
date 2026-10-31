---
type: system
---

# Capture Runbook

Read verbatim by every capture sweep (BUILD-SPEC section 8). Tune it by what it gets wrong.

## 1. What a sweep does

For a time window, read email (inbox and sent), chat and the ticket tracker. Find **requests made of
me** and **promises I made**. File each as a `Queue/` note with `status: verify`, a verbatim
`## Evidence` quote, `source`, `confidence` and `origin`, plus one heartbeat file. Never edit an
existing note, never reply to anyone, never read `Personal/`.

## 2. Sources

| Source | How to search | Known limits |
|---|---|---|
| <!-- e.g. Email --> | | |

## 3. Rules

- Classify conversations, not messages. A question answered in-thread minutes later is closed.
- Low confidence is listed in the heartbeat but never filed.
- Never re-create something I dropped. Skip people who have left.
- At most 40 notes per run.
- Judge the home project from the thread and write a "Project because" line; I confirm in Triage.
- Set `due` from any stated deadline.
- Capture my notes-to-self as `type: task`, `origin: self`.
- Capture promises ("I'll...", "I can help with...") as well as requests.

## 4. Heartbeat

Write `_System/Sweeps/YYYY-MM-DD HHmm <run>.md` on **every** run, even with zero results: run name,
window-start, window-end, per-source status (ok | partial | failed, with reasons), counts (captured,
held back as low confidence, duplicates), and what each source did NOT read.
