# ObsidianSync

[English](./README.md) | [简体中文](./README.zh-CN.md)

![license](https://img.shields.io/badge/license-MIT-blue)
![version](https://img.shields.io/badge/version-1.0.0-green)
![claude-code](https://img.shields.io/badge/Claude%20Code-plugin-purple)

> A Claude Code plugin that turns Claude into a thoughtful librarian for your local Obsidian knowledge base — PARA classification, auto-tagging, bidirectional links, health checks, summaries, URL import, and autonomous vault recall that grounds answers in your own notes — across **8 model-invoked Skills**. Every write is proposed first and committed only after you confirm.

ObsidianSync does **not** run a sync daemon or push to a remote. "Sync" means *keeping your vault in sync with itself*: when you save or import a note, Claude classifies it, tags it, links it to related notes, and writes it to the right PARA folder — always showing you the plan and waiting for a **yes** before touching disk.

---

## Table of Contents

- [Features](#features)
- [How It Works](#how-it-works)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Development Guide](#development-guide)
- [Migration from the Single-Skill Version](#migration-from-the-single-skill-version)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## Features

- **8 focused Skills** — `save`, `query`, `recall`, `link`, `maintain`, `tag`, `summary`, `import`. Claude picks the right one from your natural language; no commands to memorize.
- **Autonomous vault recall** — before answering, Claude proactively searches your vault and folds your relevant notes into its reply, so answers are grounded in what you've already written (read-only; self-invoked).
- **PARA organization** — every note is filed into `00_Inbox` / `10_Projects` / `20_Areas` / `30_Resources` / `40_Archives` with a transparent, ordered decision rule. Uncertain → Inbox, never a wrong archive.
- **Auto-tagging that respects what already exists** — new tags are checked against the vault's existing set first, so `机器学习` / `ML` / `machine-learning` don't all coexist.
- **Bidirectional links** — suggests `[[wikilinks]]` by matching note titles + aliases against your new note's body, ranked by relevance.
- **Write-before-confirm** — all mutations (create / edit / delete) show a proposal card and wait for your approval. Read-only operations return results directly.
- **Vault health checks** — broken links, orphan notes, malformed frontmatter, Inbox backlog, tag drift — reported and graded.
- **Summaries & MOC** — refresh a single note's summary, or generate a Map-of-Content index note for the whole vault.
- **URL import** — fetch a URL and run it through the full `save` pipeline into `30_Resources`.
- **Zero external dependencies** — uses only Claude Code's built-in tools plus the system `ripgrep` (`rg`).

## How It Works

Each Skill is short and self-contained: its own trigger conditions and flow in `skills/<name>/SKILL.md`. Shared rules — vault paths, the three iron rules, the confirmation card, PARA/tag/link logic — live once in `references/` and are pulled in via `${CLAUDE_PLUGIN_ROOT}/references/...`, so nothing is duplicated across the eight Skill files.

**The three iron rules** (every write operation obeys them):

1. **Confirm before writing** — never silently create, modify, or delete a file.
2. **Don't destroy** — without explicit consent: no deletions, no overwrites. Edits to existing notes use precise, minimal diffs.
3. **Stay traceable** — every step uses absolute paths and reports what changed and where.

When a Skill wants to write, you see a card like this and only your **confirm** lands it on disk:

```
📥 Storage plan
- Path:    20_Areas/self-attention.md
- Template: concept-note.md
- Tags:    #concept #deep-learning #attention
- Links:   [[Transformer]]
- Summary: Self-attention computes in-sequence relevance via Q/K/V matrices.
Write to disk? (adjust any field)
```

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) (plugin support).
- A local [Obsidian](https://obsidian.md) vault (the `.md` files on disk — the plugin reads/writes files, the Obsidian *app* is optional but recommended for browsing).
- `ripgrep` (`rg`) on your `PATH` (preinstalled on most systems; `brew install ripgrep` otherwise).

## Installation

ObsidianSync installs through a Claude Code **marketplace**. Point Claude Code at the repo; for local development you can point it at a directory instead.

### Option A — From the published marketplace (recommended)

```
/plugin marketplace add https://github.com/Wang-Shujie/ObsidianSync
/plugin install obsidiansync@obsidian-sync
```

Then restart Claude Code (or open a new session).

### Option B — From a local checkout (development / private use)

Clone the repo, then add its directory as a marketplace — `marketplace.json` lives at the repository root and its `source` points at `./ObsidianSync`:

```
git clone https://github.com/Wang-Shujie/ObsidianSync
/plugin marketplace add /absolute/path/to/ObsidianSync
/plugin install obsidiansync@obsidian-sync
```

### Option C — Pinned in `settings.json` (teams / persistence)

Add to `~/.claude/settings.json` (user) or `.claude/settings.json` (project):

```json
{
  "extraKnownMarketplaces": {
    "obsidian-sync": {
      "source": { "source": "file", "path": "/absolute/path/to/ObsidianSync" }
    }
  },
  "enabledPlugins": {
    "obsidiansync@obsidian-sync": true
  }
}
```

> The local-marketplace `source` shape varies across Claude Code versions (`"file"` + `path`, vs. a bare path). If Option C errors, fall back to Option A/B.

### Verify

After restarting, ask Claude to *"list available Skills"* — you should see `obsidiansync:save`, `obsidiansync:query`, … `obsidiansync:import`. Or test with *"use obsidiansync to save a note."*

## Configuration

Vault paths resolve from one place: the **`KB_ROOT`** rule at the top of [`ObsidianSync/references/config.md`](./ObsidianSync/references/config.md). `KB_ROOT` is resolved by precedence — first non-empty value wins:

1. Environment variable **`OBSIDIAN_KB_ROOT`** (recommended for persistence — set it in your shell profile or Claude Code env so it survives plugin reinstalls).
2. The **`KB_ROOT` default** line in `references/config.md` (edit it once to your vault root).
3. If neither is set, Claude **asks you** for the vault path, then asks **whether to save it** — confirm and it's written into the `KB_ROOT default` line in `references/config.md` (so rule 2 picks it up automatically from then on); decline and it's used for this session only.

```md
# references/config.md (excerpt)
- KB_ROOT (resolve by precedence): env OBSIDIAN_KB_ROOT → default below → ask in chat
- KB_ROOT default: <your-vault-root>          # e.g. ~/Obsidian/MyVault
- PARA dirs (under KB_ROOT): 00_Inbox  10_Projects  20_Areas  30_Resources  40_Archives
- Attachments: 90_Attachments
- Templates (KB_ROOT/80_Templates/): concept / project / resource / daily
```

| Setting | Where | Default / notes |
|---|---|---|
| Vault root (`KB_ROOT`) | `references/config.md` (or `OBSIDIAN_KB_ROOT` env) | Precedence: env var → config default → ask in chat (with option to save) |
| PARA folder names | `references/config.md` | `00_Inbox` `10_Projects` `20_Areas` `30_Resources` `40_Archives` |
| Classification / tag / link rules | `references/classification.md` | Edit freely; no Skill changes needed |
| Note templates | `<KB_ROOT>/80_Templates/*.md` | Register new templates in `classification.md` |

## Usage

Skills are **model-invoked** — Claude decides which to run from what you say. The `@name` prefixes below just help you be explicit; you don't have to type them.

| Skill | Say something like | What happens |
|---|---|---|
| `save` | "Save this: …" | Analyze → PARA classify → pick template → tag → link → **confirm** → write |
| `query` | "Find notes about X in the vault" | Multi-dimensional search: keyword / tag / date / folder (read-only) |
| `recall` | *(Claude self-invokes — you don't ask)* | Proactively searches the vault before answering and folds your notes in, so replies are grounded (read-only) |
| `link` | "Connect the orphan notes" | Find orphans, suggest `[[X]]` inserts, apply the ones you check |
| `maintain` | "Check vault health" | Broken links / orphans / frontmatter / Inbox backlog / tag drift, graded report |
| `tag` | "List tags, merge #ML and #机器学习" | Tag inventory, rename/merge/delete (confirm each step) |
| `summary` | "Summarize this note" / "Build a vault MOC" | Refresh one note's summary; or generate a vault index note |
| `import` | "Import this article: https://…" | Fetch URL → run `save` pipeline → file under `30_Resources` |

### Examples

```
# Save
"Save this: Self-attention uses Q/K/V matrices to weigh tokens in a sequence."

# Query (read-only, instant)
"What do I have tagged #deep-learning from last month?"

# Health check
"Run a vault health report — focus on broken links and the Inbox backlog."

# Import
"Import https://example.com/article into the vault."
```

Read-only Skills (`query`, the scan phase of `maintain`, the inventory phase of `tag`) return results directly. Every writing Skill shows the confirmation card first.

## Project Structure

The repository root is the **marketplace**; the plugin itself lives in `ObsidianSync/`:

```
ObsidianSync/                       # repository root (marketplace)
├── .claude-plugin/
│   └── marketplace.json            # marketplace manifest — source → ./ObsidianSync
├── ObsidianSync/                   # the plugin
│   ├── .claude-plugin/
│   │   └── plugin.json             # plugin manifest (name / version / description)
│   ├── skills/                     # 8 Skills, one directory each
│   │   ├── save/SKILL.md           # @save    smart store
│   │   ├── query/SKILL.md          # @query   multi-dimensional search
│   │   ├── recall/SKILL.md         # @recall  self-invoke to ground answers
│   │   ├── link/SKILL.md           # @link    bidirectional links
│   │   ├── maintain/SKILL.md       # @maintain health check
│   │   ├── tag/SKILL.md            # @tag     tag management
│   │   ├── summary/SKILL.md        # @summary summary / MOC
│   │   └── import/SKILL.md         # @import  URL import
│   └── references/                 # shared, via ${CLAUDE_PLUGIN_ROOT}/references/...
│       ├── config.md               # paths / tool conventions / iron rules / confirm card
│       └── classification.md       # PARA / tags / links / writing spec
├── LICENSE
├── README.md
└── README.zh-CN.md
```

> The inner `ObsidianSync/` directory is the plugin; the marketplace manifest at the repository root points to it via `"source": "./ObsidianSync"`.

## Development Guide

### Change the vault path
Edit `KB_ROOT` at the top of `ObsidianSync/references/config.md`. All Skills inherit it. (Or override per-session in chat.)

### Add a Skill
1. Create `skills/<name>/SKILL.md`. Frontmatter requires `name` (lowercase / digits / hyphens, ≤64 chars) and `description` (≤1024 chars — state *what it does + when to use it*; list trigger words in both EN and ZH).
2. Optionally pin tools via `allowed-tools` (e.g. read-only Skills: `Read, Grep, Glob, Bash`).
3. Put shared rules in `references/` and reference them as `${CLAUDE_PLUGIN_ROOT}/references/xxx.md`.
4. Reinstall / restart to activate.

### Change classification / tag / link rules
Edit `references/classification.md` directly — no Skill edits required.

### Add a template
Drop a `.md` into the vault's `80_Templates/` (with standard frontmatter and the `📝 Summary / 📖 Body / 🔗 Links / 📚 References / 🏷️ Tags` structure), then register it in the note-type → template table in `classification.md`.

### Dependencies
Only Claude Code's built-in tools (`Bash`, `Read`, `Write`, `Edit`, `WebFetch`, `Grep`, `Glob`) and system `rg`. No npm/pip packages.

### Local dev loop
```
/plugin marketplace add /absolute/path/to/ObsidianSync   # once
# edit SKILL.md / references/*.md
# in Claude Code: reinstall or restart to reload
```

## Migration from the Single-Skill Version

This plugin was refactored from an earlier single-file `ObsidianSync/SKILL.md` (now deleted). Its seven commands were split into seven independent Skills under `skills/` (the eighth, `recall`, was added later), with shared logic moved into `references/`. The old symlink install (`ln -s … ~/.claude/skills/obsidiansync`) no longer applies — use the [marketplace install](#installation) above.

## Contributing

Contributions are welcome — browse [issues](https://github.com/Wang-Shujie/ObsidianSync/issues) and [pull requests](https://github.com/Wang-Shujie/ObsidianSync/pulls).

1. Fork the repo and create a feature branch (`git checkout -b feat/<name>`).
2. Keep Skills short and self-contained; put anything reused into `references/`.
3. Respect the three iron rules — no Skill may write without a confirmation card.
4. If you add a Skill or change shared rules, update this README and `references/` together.
5. Open a Pull Request describing the change and a usage example.

**Good first issues:** new templates, additional languages for trigger words, new `query` dimensions, more `maintain` health checks.

Before submitting, please search existing [issues/PRs](https://github.com/Wang-Shujie/ObsidianSync/issues) to avoid duplicates.

## License

Released under the **MIT License**. See [`LICENSE`](./LICENSE).

## Acknowledgments

- [Obsidian](https://obsidian.md) — the knowledge base this plugin manages.
- [Claude Code](https://docs.claude.com/en/docs/claude-code) — the agent platform and plugin model.
- The **PARA method** (Tiago Forte) for the organization model.

**Author:** [Wang-Shujie](https://github.com/Wang-Shujie) · **Repo:** https://github.com/Wang-Shujie/ObsidianSync · **Issues:** https://github.com/Wang-Shujie/ObsidianSync/issues
