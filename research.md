---
name: obsidian-god:research
description: Generates an interconnected research cluster into the existing Obsidian vault under Research/<topic>/. Triggered by /obsidian-god:research. Requires an explicit effort level: quick, explore, or full.
---

# obsidian-god:research

Generates a research cluster directly into `Research/<topic>/` in your vault. Shares vault path, YAML conventions, and `_index.md` protocol with obsidian-god.

## Invocation

```
/obsidian-god:research [topic] effort:[quick|explore|full]
```

## Effort Levels (hardcoded — apply exactly as specified, no inference)

| Level   | Pages | Batch size | WebSearch/page | Min wikilinks/page | Min page types |
|---------|-------|------------|----------------|--------------------|----------------|
| quick   | 12    | 5          | 1              | 5                  | 4              |
| explore | 30    | 5          | 2              | 7                  | 6              |
| full    | 55    | 5          | 3              | 10                 | 8              |

Read the effort parameter from the invocation. Apply the corresponding row exactly. Do not adjust values based on topic complexity or any other factor.

Available page types: concept, theory, person, controversy, method, debate, term, school
No single type may exceed 50% of total pages.

---

## Output Structure

```
Research/<topic>/
├── _index.md                  ← generated at completion (obsidian-god protocol)
├── _MOC.md                    ← Map of Content hub page
├── <page-slug>.md             ← N pages per effort level
└── 99-Meta/
    ├── Plan.md                ← compact page plan table
    ├── Fact-Check-Log.md      ← verified single source of truth
    └── Audit.md               ← compliance and consistency report
```

YAML frontmatter required on every page:
```yaml
---
title: <page title>
type: <page type>
cluster: <topic>
status: draft
tags: []
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

---

## Phase 0 — Bootstrap

1. Read `_vault-config.md` from vault root. If missing, use defaults.
2. Check if `Research/<topic>/` already exists. If it does, warn the user and ask: continue (append) or abort?
3. Create `Research/<topic>/99-Meta/` folder structure.
4. State the effort level parameters you will apply (repeat the table row).

---

## Phase 1 — Page Plan

Generate `99-Meta/Plan.md` as a compact table:

```
| ID | Title | Type |
|----|-------|------|
| 01 | ...   | ...  |
```

Rules:
- Exact page count from effort table
- Meet minimum page type count for this effort level
- No single type >50% of total pages
- IDs are zero-padded integers (01, 02, ...)

**CHECKPOINT 1** — Stop. Output:
> "Plan written to `99-Meta/Plan.md`. Review it and reply `GO` to begin writing, or `EDIT: [instructions]` to revise."

Wait for user reply before proceeding.

---

## Phase 2 — First Batch (5 pages)

Write pages 01–05 from the plan:

For each page:
1. Run WebSearch (N queries per effort table) to fact-check key claims
2. Write the page with minimum wikilinks per effort table
3. Append verified findings to `Fact-Check-Log.md`:
   - Format: `[page-slug] | [claim] | [verified/disputed/unverifiable] | [source URL or "none"]`
   - Only log after confirming or denying — no speculative entries

After all 5 pages in the batch: output a one-line status:

Compute total batch count as `ceil(page_count / 5)` and use that value as N in all status lines.

> "Batch 1/N complete (5 pages written). Continuing..."

**CHECKPOINT 2** — Stop after first batch. Output:
> "First 5 pages written. Review any of them and reply `GO` to continue, `ADJUST: [instructions]` to change approach, or `STOP` to end here."

Wait for user reply before proceeding.

---

## Phase 3 — Remaining Batches

Write remaining pages in batches of 5 using the same per-page protocol as Phase 2.

After each batch output a one-line status: `"Batch N/Total complete (X/Y pages written). Continuing..."`

No further checkpoints. User may interrupt at any time by sending a message.

The final batch may contain fewer than 5 pages; write all remaining pages and report the actual count in the status line.

---

## Phase 4 — MOC

Write `_MOC.md`:

```markdown
---
title: <topic> — Map of Content
type: moc
cluster: <topic>
status: active
tags: [moc]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---

# <topic> — Map of Content

## <Type Group 1>
- [[page-slug|Page Title]] — one-line description

## <Type Group 2>
- [[page-slug|Page Title]] — one-line description
```

Group pages by type. Every page must appear exactly once.

---

## Phase 5 — Audit

Check all pages and write `99-Meta/Audit.md` with findings:

- YAML frontmatter: all required fields present on every page?

Note: `_MOC.md` uses type `moc` and is exempt from page-type distribution and minimum-type checks.

- Orphaned pages: any page with zero wikilinks pointing to it?
- Wikilink targets: do all `[[wikilinks]]` resolve to a real page in the cluster?
- Contradictions: flag any pages that make conflicting factual claims
- Fact-Check-Log: any `unverifiable` entries that should be flagged in the page itself?

Format:
```
## YAML
- [PASS/FAIL] <page-slug>: <issue if any>

## Orphaned pages
- <page-slug> (no inbound links)

## Broken wikilinks
- [[target]] in <page-slug> — target not found

## Contradictions
- <page-slug-A> vs <page-slug-B>: <description>

## Unverifiable claims in text
- <page-slug>: <claim>
```

---

## Phase 6 — Index

1. Generate `Research/<topic>/_index.md` using obsidian-god's folder index format:

```
# /Research/<topic> — Folder Index
updated: <YYYY-MM-DD>

## Markdown files
| File | Title | Status | Tags |
|------|-------|--------|------|
| _MOC.md | <topic> — Map of Content | active | [moc] |
| <page-slug>.md | <title> | draft | [...] |
```

2. Check if `Research/_index.md` exists. If yes, append a row for this cluster. If no, create it with this cluster as the first entry.

3. Write `Research/<topic>/99-Meta/_index.md` listing Plan.md, Fact-Check-Log.md, and Audit.md in the standard folder index format.

---

## Token Efficiency Rules

- Plan table is written once and referenced by ID — never re-read during writing
- Pages written in batches of 5 — no per-page round-trips
- `_index.md` generated once at Phase 6, not after each page
- No word count targets — content density is natural
- WebSearch results used immediately per page — not stored for batch processing
