---
type: project
status: active
status-detail: 
workstream: "[[Admin]]"
aliases: []
owner: 
started: {{date}}
last-updated: {{date}}
tags: [project]
---

<!-- FILL RULES (read, don't execute):
- One note per project you actually engage Claude on. Work you only support stays a task.
- status: active | blocked | winding-down | paused | complete. Anything else hides the note.
- aliases: every name people use for it (tool, repo, nickname). Never a term two projects share.
- The plugin writes focus, resume, resume-next, resume-date, resume-via, reviewed, archived.
- Next Steps are "- [ ]" bullets. Put the first thing to do under a "### Resume here" heading.
- Delete this comment when you fill the note in. -->

# {{title}}

## Overview

## Key Decisions

## Open Questions

## Next Steps

### Resume here

- [ ] 

## Open asks

```base
filters:
  and:
    - file.hasLink(this.file)
    - note.type == "ask"
```

## Systems / Tools

## Related

## Resume log
