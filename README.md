# Vault + Cockpit starter kit

From [tylerweed.com/vault-kit](https://tylerweed.com/vault-kit). Free to use and adapt.

This is the setup I'd have wanted before I rebuilt my own system several times: a plain-markdown
vault that Claude works inside, the instructions that make every session leave it better than it
found it, and the full spec for **Cockpit**, an Obsidian dashboard plugin that answers "where did I
leave this?" without any AI in the loop.

## What's in here

| Path | What it is |
|---|---|
| `BUILD-SPEC.md` | The full build spec: 26 design rules with their reasons, the vault contract, the plugin contract, optional brief and capture sweeps, and a 12-step build order with a check per step. Written to hand to Claude Code. |
| `global-CLAUDE.md` | A template for your user-level instructions (`~/.claude/CLAUDE.md`), which apply to every Claude Code session on the machine. Fill in the `<...>` placeholders here and in `vault/CLAUDE.md`. |
| `vault/` | Most of step 0, done for you: the folder structure, `CLAUDE.md` for the vault, the four templates, the `_System` notes and an Admin workstream. The `.base` view files, `core-plugins.json` and `snippets/cockpit.css` are created in your first session; section 4.1 of the spec lists them. |

**Requirements:** Obsidian 1.9+ (for Bases) and Claude Code.

There is no plugin source code in this kit. The spec builds it, one step per session.

## Three ways to start

**1. Build the whole thing with Claude Code (recommended).**
Copy `vault/` to a local folder that is *not* cloud-synced, open it in Obsidian, and turn on the
**Bases** core plugin. Then start Claude Code in that folder and paste:

```text
Read BUILD-SPEC.md in full (it's at <path to this kit>). Step 0 is already
done in this folder: check it against section 10 and tell me what's missing.
Then ask me for my 4-6 workstream names and 3-5 real projects, and convert
them into notes. Do not start step 1 until I confirm the step-0 check passes.
Never read or write a folder named Personal/. At the end of every session,
log to the three targets in section 5.2.
```

**2. Just the vault.** Use `vault/` and `global-CLAUDE.md` without the plugin. The folder
contract, templates and session-logging rules work on their own, and Obsidian's Bases can stand in
for the dashboard (section 4.6 of the spec lists the views).

**3. Just read it.** Section 3 of the spec (design decisions) and section 6.7 (Obsidian hazards) are
the parts most worth stealing even if you never build any of it.

The ` ```base ` blocks in the templates are starting points. Bases formulas can't be checked
outside Obsidian, so look at them on screen the first time (spec section 4.6 lists the traps).

## Adapting it

The spec is opinionated on purpose. Section 12 lists what to change for your own setup (private
folder name, scheduled-task names, thresholds, sources). Every rule carries the reason it exists,
so you can tell which reasons apply to you.

## License

MIT (see `LICENSE`). Built by [Tyler Weed](https://tylerweed.com).
