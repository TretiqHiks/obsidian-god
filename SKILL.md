---
name: obsidian-god
description: This skill should be used when working with any Obsidian vault — reading, editing, or creating .canvas files, navigating vault folders, or storing knowledge in an Obsidian vault. Activates on any reference to the vault path, .canvas files, or Obsidian .md files with wiki-link syntax.
---

# obsidian-god

Token-efficient Obsidian vault operations. Two-layer navigation, surgical canvas edits, optional vault-as-brain routing.

## Activation

Use this skill when:
- Working with any `.canvas` file
- Navigating or writing to the Obsidian vault at `C:\Users\tigge\Desktop\claude_hooks\knowledge-vault\`
- User mentions "vault", "obsidian", or "canvas"
- Storing research, decisions, or project notes

On activation, read `_vault-config.md` from the vault root. If missing, proceed with defaults (auto_index: true, canvas_manifests: true, brain_mode: disabled).

---

## Read Protocol

**Before opening any file in the vault:**

1. Check if `_index.md` exists in that folder.
2. If yes — read `_index.md` only. Do NOT open individual files unless you need to edit one.
3. If no `_index.md` — read files directly, then create `_index.md` afterward.

**Before touching any `.canvas` file:**

1. Check if `<filename>.canvas.manifest.md` exists alongside it.
2. If yes — read the manifest only. Identify your target node by ID and its match anchor.
3. If no manifest — read the full `.canvas` file once to generate the manifest, then proceed.

Never read a full `.canvas` file to understand structure when a manifest exists.

---

## Edit Protocol — Canvas Files

### Strategy A: Surgical Edit (primary)

1. Read `<filename>.canvas.manifest.md`
2. Find node by ID in the Nodes table
3. Use the Edit tool:
   - `old_string`: the node's match anchor (first 1-2 unique lines of its text content)
   - `new_string`: updated content
4. Regenerate the manifest after a successful edit (see Manifest Lifecycle)

**Token cost:** ~200 tokens for manifest read + Edit tool call. No full-file read needed.

**Critical:** Obsidian canvas files store node text as JSON strings. Newlines within a node's text are stored as literal `\n` (two characters: backslash + n) in the JSON file. Your `old_string` for the Edit tool must use `\n` notation when spanning multiple lines — NOT actual newlines. Example: to match a node starting with `### Revenue Math` followed by `**Rev/client:** ~€40`, use `old_string: ### Revenue Math\n**Rev/client:** ~€40`.

### Strategy B: Line-range Read (fallback)

Use when Strategy A fails (Edit tool: no match or ambiguous match).

1. Note the node's approximate line from the manifest (`Approx line` column)
2. Read `<filename>.canvas` with `offset: <line - 5>` and `limit: 20`
3. Compose a precise `old_string` from the actual JSON in that range
4. Retry the Edit tool
5. Rebuild the manifest after success

**Never:** load the full `.canvas` into context just to understand structure.

---

## Write Protocol — Canvas Files

When creating a new `.canvas` file, ALWAYS follow these conventions:

1. **Semantic node IDs** — Format: `<section>-<entity>-<aspect>` (e.g. `persona-nikolay-profile`). Never use `node-7`, UUIDs, or auto-generated hashes.
2. **Unique first line per node** — Every text node starts with a heading (`## Title`) or bold label (`**Label:**`) that is unique within the entire file. This is the match anchor.
3. **One concept per node** — Don't combine unrelated content into one node.
4. **No duplicate text snippets** — Guarantees Edit tool `old_string` hits exactly one location.
5. **Semantic edge IDs** — Format: `<fromNode>-to-<toNode>` (e.g. `title-to-nikolay-header`).

After writing a new canvas, immediately generate its manifest (see Manifest Lifecycle).

---

## Manifest Lifecycle

### Generate/Regenerate a canvas manifest

After any canvas write or edit, write `<filename>.canvas.manifest.md` alongside the canvas:

```
# <filename>.canvas — Manifest
nodes: <count> | edges: <count> | updated: <YYYY-MM-DD>

## Nodes
| ID | Type | Match anchor (first unique line) | Approx line |
|---|---|---|---|
| <id> | text | <first unique line of node text> | L<N> |
| <id> | file | <file path> | L<N> |

## Edges
<fromNode> → <toNode>, <toNode>
<fromNode> → <toNode>
```

Rules for match anchors:
- Use the first line of the node's `text` field (after JSON-unescaping `\n` → newline)
- If the first line is not unique (e.g. `### Psychology` appears multiple times), use first line + ` | ` + second line, keeping `\n` as separator
- For `file` nodes: the anchor IS the file path — use it directly in old_string
- Existing canvases created before these conventions: always use first + second line to be safe

### Update a folder index

After adding, removing, or renaming a file in a vault folder, update `_index.md` in that folder:

```
# /<folder path> — Folder Index
updated: <YYYY-MM-DD>

## Markdown files
| File | Title | Status | Tags |
|------|-------|--------|------|
| <file.md> | <frontmatter title> | <frontmatter status> | <frontmatter tags> |

## Canvas files
| File | Nodes | Edges | Manifest |
|------|-------|-------|---------|
| <file.canvas> | <N> | <N> | [[<file.canvas.manifest>]] |
```

If a folder has no `.md` or `.canvas` files, do not create `_index.md` for it.

---

## Rebuild Command

When user says **"update vault indexes"** or **"rebuild manifests"**:

1. Use Glob to find all `.canvas` files: `knowledge-vault/**/*.canvas`
2. For each canvas: read it, regenerate its sidecar manifest
3. Use Glob to find all folders containing `.md` or `.canvas` files
4. For each folder: regenerate its `_index.md`
5. Exclude folders listed in `_vault-config.md` → `exclude` (default: `Templates`, `.obsidian`)

---

## Brain Mode

When `_vault-config.md` has `brain_mode: enabled`:

**Instead of writing to `~/.claude/projects/.../memory/`**, route knowledge to the vault:

| Knowledge type | Route to |
|---|---|
| Research findings | `knowledge-vault/Research/<topic>.md` |
| Project notes | `knowledge-vault/Projects/<name>/<note>.md` |
| Decisions | `knowledge-vault/Decisions/index.md` (append entry) |
| Plans / next steps | `knowledge-vault/Projects/<name>/<note>.md` |

All vault notes require frontmatter (title, type, project, status, tags, created, updated) and a breadcrumb line under the title.

When brain mode is disabled (default), use standard `~/.claude/` memory files.

---

## Reference Files

For format details and full examples, read these references only when needed:

- `references/canvas-manifest-format.md` — full manifest format + annotated example
- `references/folder-index-format.md` — full `_index.md` format + annotated example
- `references/vault-config-schema.md` — all `_vault-config.md` fields with defaults

---

## Research Mode

To generate a new research cluster into the vault, use the research sub-skill:

```
/obsidian-god:research [topic] effort:[quick|explore|full]
```

| Level   | Pages | Batch size | WebSearch/page | Min wikilinks/page | Min page types |
|---------|-------|------------|----------------|--------------------|----------------|
| quick   | 12    | 5          | 1              | 5                  | 4              |
| explore | 30    | 5          | 2              | 7                  | 6              |
| full    | 55    | 5          | 3              | 10                 | 8              |

Output lands in `Research/<topic>/` in the existing vault. See `research.md` for the full protocol.
