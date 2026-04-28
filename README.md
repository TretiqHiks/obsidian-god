# obsidian-god

A Claude Code skill for token-efficient Obsidian vault operations — surgical canvas edits, two-layer folder navigation, and optional vault-as-brain memory routing.

## What it does

- Navigates vault folders using `_index.md` summaries instead of reading every file
- Edits `.canvas` files surgically via sidecar manifests — no full-file reads
- Keeps canvas files AI-editable by enforcing semantic node IDs and unique match anchors
- Optionally routes all Claude memory (research, decisions, project notes) into the vault instead of `~/.claude/`

## Installation

Place the skill in your Claude Code skills directory:

```
~/.claude/skills/obsidian-god/
├── SKILL.md
└── references/
    ├── canvas-manifest-format.md
    ├── folder-index-format.md
    └── vault-config-schema.md
```

Claude Code will auto-discover it. Invoke with the `Skill` tool:

```
Skill("obsidian-god")
```

## Activation triggers

The skill activates when:
- Working with any `.canvas` file
- Navigating or writing to an Obsidian vault
- User mentions "vault", "obsidian", or "canvas"
- Storing research, decisions, or project notes

## How it works

### Two-layer navigation

Before opening any vault folder, Claude checks for `_index.md`. If found, only the index is read — individual files are opened only when an edit is needed. This cuts token usage dramatically in large vaults.

### Canvas manifest system

Every `.canvas` file gets a sidecar `<filename>.canvas.manifest.md` that maps node IDs to their match anchors and approximate line numbers. Edits are made by:

1. Reading the manifest (~200 tokens)
2. Looking up the target node
3. Using the Edit tool with the match anchor as `old_string`

No full canvas load required.

### Canvas write conventions

New canvas files must follow these rules so future edits remain reliable:

| Convention | Rule |
|---|---|
| Node IDs | `<section>-<entity>-<aspect>` (e.g. `persona-user-profile`) |
| First line per node | Unique heading or bold label — this is the match anchor |
| One concept per node | Never combine unrelated content |
| Edge IDs | `<fromNode>-to-<toNode>` |

### Brain mode

When `brain_mode: enabled` is set in `_vault-config.md`, Claude routes all memory writes into the vault:

| Knowledge type | Destination |
|---|---|
| Research findings | `Research/<topic>.md` |
| Project notes | `Projects/<name>/<note>.md` |
| Decisions | `Decisions/index.md` |

Disabled by default — Claude uses `~/.claude/` memory files instead.

## Rebuild command

Say **"update vault indexes"** or **"rebuild manifests"** to regenerate all `_index.md` files and canvas manifests across the vault.

## Research mode

Generate an interconnected research cluster into `Research/<topic>/` in your vault:

```
/obsidian-god:research [topic] effort:[quick|explore|full]
```

### Effort levels

All levels produce the same quality. Effort controls scope and depth.

| Level   | Pages | Batch size | WebSearch/page | Min wikilinks/page | Min page types |
|---------|-------|------------|----------------|--------------------|----------------|
| quick   | 12    | 5          | 1              | 5                  | 4              |
| explore | 30    | 5          | 2              | 7                  | 6              |
| full    | 55    | 5          | 3              | 10                 | 8              |

### What gets generated

```
Research/<topic>/
├── _index.md              ← folder index (obsidian-god format)
├── _MOC.md                ← Map of Content hub page
├── <page-slug>.md         ← N research pages
└── 99-Meta/
    ├── Plan.md            ← page plan (reviewed at Checkpoint 1)
    ├── Fact-Check-Log.md  ← verified claim log
    └── Audit.md           ← YAML, orphan, and contradiction report
```

### Checkpoints

Two mandatory stops:
1. **After Plan.md** — review the page structure before any writing begins
2. **After first 5 pages** — validate quality before the full run continues

### Anti-hallucination

Every factual claim is WebSearch-verified before being written. Findings are logged in `Fact-Check-Log.md` as a verified single source of truth — no speculative entries.

## Reference files

| File | Purpose |
|---|---|
| `references/canvas-manifest-format.md` | Full manifest format with annotated example |
| `references/folder-index-format.md` | Full `_index.md` format with annotated example |
| `references/vault-config-schema.md` | All `_vault-config.md` fields and defaults |

## Configuration

Create `_vault-config.md` in the vault root to override defaults:

```yaml
auto_index: true          # auto-create _index.md after file changes
canvas_manifests: true    # auto-generate sidecar manifests
brain_mode: disabled      # set to "enabled" to route memory to vault
exclude:
  - Templates
  - .obsidian
```
