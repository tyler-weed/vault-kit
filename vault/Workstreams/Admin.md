---
type: workstream
aliases: []
keywords: []
---

# Admin

The catch-all workstream. It sorts last, gets a neutral colour, and shows no project count on its
dashboard tile. Add your other 4-5 workstreams next to it; name them after your real areas, or
cluster your existing projects and name the clusters (BUILD-SPEC section 12).

## Projects

```base
filters:
  and:
    - file.hasLink(this.file)
    - note.type == "project"
```

## Open tasks

```base
filters:
  and:
    - file.hasLink(this.file)
    - note.type == "task" || note.type == "ask"
```
