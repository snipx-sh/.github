# SnipX

**effortlessly discover, collect, use, learn, memorize, internalize, and turn any knowledge into muscle memory. Then speed run challenges designed to train you to be the fastest expert at anything.**

SnipX is a knowledge collection, curation, exploration, sharing, and management platform designed to take anyone from awareness of something's existence all the way to muscle memory that doesn't require referencing docs and syntax to speed run competitions, all naturally, fluently, and intuitively without the traditionally difficult burden of learning something new.

**snipx.sh · snipx.dev**

---

## Features

- **Snippets, docs, bookmarks** — three content types, one searchable library
- **Unified search** — full-text and tag search across everything
- **Shell integration** — tab-complete your entire library from the terminal
- **Keyboard-driven** — navigate and manage everything without touching a mouse
- **Import / Export** — bring your existing knowledge in, take it out in standard formats
- **Doc ingestion** — automatically pull in and index official documentation
- **Interactive tutorials** — type every code example from the docs instead of reading it
- **Speed-run challenges** — same content at increasing difficulty until it's muscle memory
- **Whole-file ghost text** — inline suggestions for the entire file, not the next token
- **Package registry** — community-contributed knowledge packs for tools and frameworks

---

## Components

- **Desktop app** — local-first interface for managing and exploring your library
- **Terminal module** — full access to your library from any shell prompt
- **Web portal** — browse and share public knowledge collections in the browser
- **API** — single source of truth that all interfaces talk to
- **Package system** — tiered community contributions from reference docs to full learning units

---

## Process

```
Collect → Curate → Retrieve → Practice → Drill → Execute
```

Three kinds of knowing: declarative (it exists), procedural (you can look it up), tacit (your hands know it). SnipX moves knowledge through all three.

---

## Resources and References

- [snipx.sh](https://snipx.sh) — project home
- [snipx.dev](https://snipx.dev) — web app and docs
- [snipx-sh/snipx.sh](https://github.com/snipx-sh/snipx.sh) — core source
- [snipx-sh/snipx.dev](https://github.com/snipx-sh/snipx.dev) — web app source
- [snipx-sh/snipx](https://github.com/snipx-sh/snipx) — package registry

---

# snipx

**snipx.sh · snipx.dev**

A local-first developer knowledge system. Snippets, docs, bookmarks, and interactive learning — searchable from your shell, extensible by the community, and designed to move knowledge from collection into muscle memory.

---

## The Problem

Learning is becoming increasingly difficult as the rate of technology accelerates. Every new language, framework, and CLI comes with its own DSL, schema, flags, and grammar, increasing both the surface area and cognitive burden beyond what any individual can absorb.

The instinctive response is hoarding — bookmarks, stars, open tabs — an accumulation of declarative knowledge that never converts into capability. You end up familiar with tools without having internalized them.

There are three distinct problems here, and they're worth keeping separate:

**Collection without absorption.** Recognition is cheap. Reproduction is what's missing. The gap between "I know this exists" and "I can reach for it without thinking" is enormous, and nothing in a typical developer's workflow bridges it.

**Availability with too much friction.** Even things genuinely learned are hard to reach in the moment. The retrieval cost is just high enough that you muddle through instead, and every slow retrieval is a missed chance to reinforce what you already know.

**No pipeline from syntax to muscle memory.** Reading docs doesn't build it. Bookmarking doesn't. Even using something occasionally doesn't get you there. The right kind of repetition, in context, at the right time, for developer tooling specifically — Vim motions, shell patterns, CLI flags, API idioms — doesn't exist yet.

snipx is an attempt to close that gap. v1 is a knowledge manager and retrieval tool. The longer arc is a system that moves knowledge from collection through procedural familiarity into tacit, automatic execution.

---

## Repositories

| Repo | Purpose |
|------|---------|
| [`snipx-sh/snipx.sh`](https://github.com/snipx-sh/snipx.sh) | Core — Tauri desktop app, HTTP API, Nushell module, dev scripts |
| [`snipx-sh/snipx.dev`](https://github.com/snipx-sh/snipx.dev) | Web app — the hosted interface, deployed on Cloudflare |
| [`snipx-sh/snipx`](https://github.com/snipx-sh/snipx) | Official knowledge base — community-maintained knowledge for tools and frameworks |

Personal content repos follow the convention `github.com/[user]/snipx` and are served automatically at `https://snipx.dev/[username]`.

---

## What snipx Is

### Desktop App + CLI

A three-pane interface — sidebar, list, detail — for managing code snippets, documentation references, and bookmarks. Tokyo Night color system. Keyboard-driven. No cloud, no account, no subscription required to use the local app.

Everything is stored in a single SQLite file at `~/.local/share/snipx/snipx.db`. Portable, backupable, exportable.

### HTTP API

A Bun + Hono API running on `localhost:7878` as a Tauri sidecar. It is the single source of truth — the desktop app, the CLI, and the Nushell module all talk to it. The app works headlessly: the API can run as a systemd user service with no GUI.

### Nushell Module

`use snipx.nu *` brings the entire library into the shell with tab-completion backed by the live API. Snippets are pipeable:

```nushell
snipx get nu-pipeline | save my-script.nu
snipx list --lang rust | where fav == true | get id | each { snipx get $in }
snipx search "axum handler"
```

### REPL

An in-app terminal panel with command history, arrow-key navigation, and the full command set: `list`, `docs`, `bookmarks`, `show`, `search`, `copy`, `fav`, `run`, `open`, `tags`, `clear`. Drag-to-resize. Toggleable via keyboard.

### snipx.dev

The web interface is served at `https://snipx.dev`. No account is required to view a public repo:

```
https://snipx.dev/danielbodnar
  → serves github.com/danielbodnar/snipx (public, read-only, anonymous)

https://snipx.dev/danielbodnar/private-knowledge
  → requires snipx.dev account + repo access grant
```

The convention assumes `[user]/snipx` as the public repo name, but authenticated users can point snipx.dev at any repo they own.

---

## The Knowledge System

A knowledge entry — `bun`, `cloudflare/containers`, `dagger` — is a directory that simultaneously satisfies multiple contracts so it works across different contexts without duplication.

### The Structure

```
[tool]/
│
│   # Claude Code skill (agentskills.io open standard)
├── SKILL.md                  # frontmatter + instructions
├── agents/                   # subagent configs: tutor.md, reviewer.md, ingest.md
├── references/               # loaded on-demand into claude context
│   ├── overview.md           # curated summary of the tool
│   ├── api.md                # key APIs, flags, schemas
│   ├── patterns.md           # idioms, anti-patterns, gotchas
│   └── grammar.md            # from tree-sitter/pest grammar where available
├── scripts/                  # executable — only output enters context, not source
│   ├── fetch-docs.ts         # fetch + curate official docs into references/
│   ├── gen-completions.nu    # generate completions from live API/CLI
│   └── validate.nu           # smoke-test the overlay
├── assets/                   # templates, starter kits, icons
│
│   # Nushell overlay (nupm-compatible module)
├── nupm.nuon                 # { name, type, version, description, license }
├── mod.nu                    # overlay entry point — export-env + re-exports
├── completions/              # custom completions, private by default
├── commands/                 # exported commands — thin wrappers + help text
├── hooks/                    # env hooks: detect project, set SNIPX_[TOOL]_*
│
│   # snipx learning unit
├── snipx.nuon                # registry manifest — extends nupm.nuon
├── snippets/                 # tagged, searchable code snippets
│   └── index.nuon            # catalog: { id, title, lang, tags, file }
├── tutorials/                # typed examples sequenced by complexity
│   └── index.nuon            # sequence metadata: { id, title, level, deps }
│
│   # MCP server (optional, generated)
└── mcp/
    └── server.ts             # exposes this tool's knowledge as MCP tools
```

No Python. Scripts are TypeScript (Bun) for anything needing HTTP or parsing, Nushell for anything shell-native or overlay-integrated.

### Contribution Tiers

Knowledge entries can be contributed at any level of completeness:

| Tier | What's included | What you get |
|------|----------------|--------------|
| 1 | `SKILL.md` + `references/` + `scripts/` | Claude context for this tool |
| 2 | Tier 1 + `mod.nu` + `completions/` + `hooks/` | Full Nushell overlay |
| 3 | Tier 2 + `snippets/` + `tutorials/` + `snipx.nuon` | Learning unit + MCP |

`snipx add <tool>` checks which tier is available and activates accordingly.

### The Manifest

`snipx.nuon` is what tells snipx what an entry provides:

```nuon
{
  name:        "bun"
  version:     "1.2.0"
  type:        "module"
  license:     "MIT"
  description: "Bun runtime — scripts, tests, bundler, package manager"
  topics:      ["runtime", "typescript", "bundler", "testing"]
  deps:        []
  snippets:    "snippets/index.nuon"
  tutorials:   "tutorials/index.nuon"
  skill:       "SKILL.md"
  overlay:     "mod.nu"
  mcp:         "mcp/server.ts"
  difficulty:  1
}
```

---

## Adding Knowledge

Skills follow XDG + find-up resolution. When installing, snipx walks up from the current directory looking for a git root, then decides where to place the skill based on context. Both agent conventions are supported simultaneously via symlinks.

```
# Search order (first match wins):
./.agents/skills/[tool]/          # project-local (agentskills.io)
./.claude/skills/[tool]/          # project-local (claude-code)
                                  # ... walk up to fs root
~/.agents/skills/[tool]/          # user-level
~/.claude/skills/[tool]/          # user-level claude-code
~/.snipx/skills/[tool]/           # user-level snipx canonical
```

The skill directory at any of these locations is always thin — `SKILL.md` plus symlinks back to `~/.snipx/packages/[tool]/`. Content lives in exactly one place.

```
# Adding knowledge:
snipx add dagger
snipx add cloudflare/containers
snipx add danielbodnar/neovim
bunx snipx@latest add dagger
bunx skills add snipx-sh/snipx --skill dagger -a claude-code -y
```

---

## File Locations

```
~/.snipx/                         # SNIPX_HOME
  packages/                       # installed package content
  skills/                         # user-level skill dirs (thin + symlinked)
  data/                           # snipx.db, vector index, cache

~/.config/snipx/                  # SNIPX_CONFIG_DIR
  config.toml                     # SNIPX_CONFIG_FILE
  mcp.json
  activations.nuon
  packages.nuon
```

Each path is independently overridable:

```
SNIPX_HOME           overrides ~/.snipx/
SNIPX_CONFIG_DIR     overrides ~/.config/snipx/
SNIPX_CONFIG_FILE    overrides ~/.config/snipx/config.toml
SNIPX_CONFIG         alias for SNIPX_CONFIG_FILE
```

---

## The Longer Arc

snipx v1 is a knowledge manager and retrieval tool. The longer arc is a system that moves knowledge from collection through to muscle memory — three additional layers that build on the foundation:

**Doc corpus ingestion.** Automatic downloading, parsing, and vectorization of official docs, API schemas, and tree-sitter grammars into a local index. This feeds search, completions, and the tutorial layer — and doubles as the training corpus for whole-file suggestions.

**Auto-generated interactive tutorials.** Extract and sequence every code example from an ingested doc corpus into a typing interface where the learner produces each example rather than reads it. Ghost text provides scaffolding. Completions take the edge off the purely mechanical parts. When you finish typing an example, snipx saves it as a tagged snippet. Learning and building the personal library happen at the same time.

**Progressive speed-run challenges.** The same tutorial content, run at increasing difficulty, stripping away scaffolding with each pass until the learner can execute from a blank file at speed with no assistance. Scoring on speed, accuracy, and scaffold usage. Applies to Vim motions, shell patterns, CLI flags, Nushell pipelines, Emmet abbreviations — anything with a learnable motor pattern.

```
Level 1 — full ghost text, unlimited completions, no time pressure
Level 2 — partial ghost text, limited completions, relaxed par time
Level 3 — first line only, no completions, normal par time
Level 4 — blank file, no hints, tight par time
Level 5 — blank file, no hints, no errors, fast par time
```

A level 5 pass means it is genuinely second nature. That is the threshold the system is working toward.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Desktop shell | Tauri v2 |
| Frontend | React 19, Vite, TypeScript strict |
| Styling | Inline styles, Tokyo Night palette |
| API server | Bun + Hono |
| Validation | Zod |
| Database | SQLite (`bun:sqlite`) |
| CLI / shell | Nushell module (`snipx.nu`) |
| Package format | nupm-compatible (`nupm.nuon` + `mod.nu`) |
| Skill format | agentskills.io open standard (`SKILL.md`) |
| Web hosting | Cloudflare (Pages, Workers, R2, KV) |
| Content sync | Cloudflare R2 + user GitHub repos |
| Package registry | `github.com/snipx-sh/snipx` + `bunx snipx@latest` |

---

## Philosophy

**Sane Defaults.** Be thin glue. Align with the upstream platform. Never compete with it. Data lives in native formats. The app is a viewer and organizer, not a walled garden.

**Local-first.** Everything works offline. The API is `127.0.0.1` only. No telemetry. No account required for the desktop app or the CLI.

**Files as the interface.** References, snippets, tutorials, overlays, skill manifests — everything is a file. Vectorizable, searchable, git-trackable, R2-syncable, human-readable without tooling.

**Compounding knowledge.** Some knowledge makes other knowledge cheaper to acquire. The package dependency graph (`deps` in `snipx.nuon`) encodes this — learning Nushell before jq, Unix pipes before Docker, touch typing before Vim. The order matters.

---

## Contributing

Knowledge entries live in [`snipx-sh/snipx`](https://github.com/snipx-sh/snipx) — or in your own `github.com/[user]/snipx`. Any tier of contribution is welcome. A well-curated `references/` and a good `SKILL.md` is already useful. A full tier-3 entry with tutorials and speed runs is the goal.

---

*snipx.sh · snipx.dev · MIT*
