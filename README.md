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

Generates an interconnected research cluster directly into your existing vault under `Research/<topic>/`. Pages are WebSearch-verified, wikilinked, and organized into an Obsidian graph you can navigate immediately after the run.

### Usage

```
/obsidian-god:research "Cognitive Biases" effort:quick
/obsidian-god:research "Stoicism" effort:explore
/obsidian-god:research "Consensus Mechanisms" effort:full
```

Topic goes in quotes. Effort level is required.

### Effort levels

All levels produce the same quality. Effort controls scope and depth only.

| Level   | Pages | Batch size | WebSearch/page | Min wikilinks/page | Min page types |
|---------|-------|------------|----------------|--------------------|----------------|
| quick   | 12    | 5          | 1              | 5                  | 4              |
| explore | 30    | 5          | 2              | 7                  | 6              |
| full    | 55    | 5          | 3              | 10                 | 8              |

**quick** — a solid starter cluster. Good for exploring a new topic or validating scope before going deeper.

**explore** — a well-rounded wiki. Covers the major concepts, people, methods, and controversies with cross-links between them.

**full** — a comprehensive reference. Dense wikilinks, broad type coverage, full WebSearch verification on every claim.

### What happens when you run it

**Phase 0 — Bootstrap**
Claude confirms the vault path, reads your `_vault-config.md`, and states the exact parameters it will use for your chosen effort level.

**Phase 1 — Page plan**
Claude generates `99-Meta/Plan.md` — a compact table of all pages with their IDs and types (concept, theory, person, controversy, method, debate, term, school). Minimum 4–8 distinct types depending on effort level. No single type exceeds 50% of the total.

> **Checkpoint 1** — Claude stops here. Review the plan and reply:
> - `GO` to begin writing
> - `EDIT: [instructions]` to adjust the plan before writing starts

**Phase 2 — First batch**
Claude writes the first 5 pages. Each page is WebSearch-verified before being written. Findings are logged to `Fact-Check-Log.md` with verification status (`verified / disputed / unverifiable`) and source.

> **Checkpoint 2** — Claude stops after the first 5 pages. Review 1–2 of them and reply:
> - `GO` to continue the full run
> - `ADJUST: [instructions]` to change approach before continuing
> - `STOP` to end here with what's been written

**Phase 3 — Remaining batches**
Claude writes the remaining pages in batches of 5. A one-line status is output after each batch. No further checkpoints — you can interrupt at any time by sending a message.

**Phase 4 — MOC**
Claude writes `_MOC.md`, a Map of Content hub page that links every page in the cluster grouped by type.

**Phase 5 — Audit**
Claude checks all pages for YAML compliance, orphaned pages, broken wikilinks, contradictions, and unverifiable claims. Results written to `99-Meta/Audit.md`.

**Phase 6 — Index**
Claude generates `Research/<topic>/_index.md` and updates the parent `Research/_index.md` — both in the standard obsidian-god folder index format, ready for two-layer navigation.

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

Every page has required frontmatter (`title`, `type`, `cluster`, `status`, `tags`, `created`, `updated`) and a minimum number of outgoing wikilinks per effort level.

### Anti-hallucination

Every factual claim is WebSearch-verified before being written. All findings — including disputed and unverifiable claims — are logged in `Fact-Check-Log.md` as a verified single source of truth. No speculative entries are written.

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
