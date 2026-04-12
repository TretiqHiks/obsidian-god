# Folder Index Format

## Purpose
One `_index.md` per vault folder. Allows Claude to understand folder contents
without opening individual files.

## Full format

```
# /<relative-folder-path> — Folder Index
updated: <YYYY-MM-DD>

## Markdown files
| File | Title | Status | Tags |
|------|-------|--------|------|
| buyer-personas.md | Buyer Personas | active | sales, personas |
| sales-plan.md | Sales Plan | active | sales, strategy |

## Canvas files
| File | Nodes | Edges | Manifest |
|------|-------|-------|---------|
| buyer-personas.canvas | 27 | 9 | [[buyer-personas.canvas.manifest]] |
| sales-plan.canvas | 14 | 6 | [[sales-plan.canvas.manifest]] |
```

## Rules

- Title and Status come from the file's YAML frontmatter `title` and `status` fields
- Tags come from frontmatter `tags` field, joined with `, `
- If frontmatter is missing, use filename (without extension) as title, leave status and tags blank
- List files alphabetically within each section
- If a folder contains only subdirectories (no .md/.canvas), do not create _index.md
- Do NOT list `_index.md` itself, `_vault-config.md`, or `*.manifest.md` files in the index
- Do NOT recurse into subdirectories — each folder has its own _index.md

## When to create vs update

Create: when first writing any file to a folder that has no _index.md yet
Update: after adding, removing, or renaming a file in the folder
Full rebuild: when user says "update vault indexes" (scan all folders)
