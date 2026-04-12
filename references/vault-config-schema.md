# Vault Config Schema

## File location
`_vault-config.md` at the vault root (e.g. `knowledge-vault/_vault-config.md`)

## Full schema with defaults

```yaml
---
claude_skill: obsidian-god
version: 1
---

# Claude Obsidian Config

vault_path: C:\Users\tigge\Desktop\claude_hooks\knowledge-vault

## Brain Mode
brain_mode: disabled
# Options: enabled | disabled
# enabled: Claude routes all knowledge output to this vault
#   research  → Research/<topic>.md
#   project notes → Projects/<name>/<note>.md
#   decisions → Decisions/index.md (appended)
# disabled (default): Claude uses ~/.claude/ memory files

## Index settings
auto_index: true
# true (default): regenerate _index.md after any file write in a folder
# false: only update on explicit "update vault indexes" command

canvas_manifests: true
# true (default): regenerate sidecar manifest after any canvas write/edit
# false: only update on explicit "rebuild manifests" command

## Excluded folders
exclude:
  - Templates
  - .obsidian
# These folders are never indexed and never have manifests generated
```

## Reading the config

Read `_vault-config.md` at skill activation. Parse the YAML frontmatter for `claude_skill`
and `version`, then parse the body as loose YAML-style key: value pairs.

If the file does not exist, use these defaults:
- brain_mode: disabled
- auto_index: true
- canvas_manifests: true
- exclude: [Templates, .obsidian]
