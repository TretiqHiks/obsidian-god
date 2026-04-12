# Canvas Manifest Format

## Purpose
Sidecar files that allow Claude to navigate .canvas files without reading them.
File naming: `<canvas-name>.canvas.manifest.md`

## Full format

```
# <canvas-name>.canvas — Manifest
nodes: <total count> | edges: <total count> | updated: <YYYY-MM-DD>

## Nodes
| ID | Type | Match anchor (first unique line) | Approx line |
|---|---|---|---|
| title | text | # EstateVisio — Buyer Personas | L3 |
| nikolay-header | text | ## 🏃 Николай — Sales Agent | L5 |
| nikolay-profile | text | **Age:** 28-40 \| **Experience:** 3-8 years | L6 |
| nikolay-psychology | text | ### Psychology\n- **Cares about:** Making his one listing | L7 |
| comparison-table | text | \| Factor \| Николай \| Петър | L25 |
| personas-file | file | Projects/estatevisio/sales/buyer-personas.md | L27 |

## Edges
title → nikolay-header, petar-header, maria-header, georgi-header
nikolay-math → comparison-header (color: 4)
petar-math → comparison-header (color: 2)
comparison-table → signals
```

## Node types

| Canvas type | Manifest treatment |
|---|---|
| `text` | Extract first unique line(s) as match anchor |
| `file` | Record the `file` field value (vault-relative path) |
| `group` | Record label field as match anchor |
| `link` | Record `url` field |

## Match anchor rules

1. Use the first line of the node's `text` field (JSON-unescape `\n` to get the actual first line)
2. If the first line appears more than once in the canvas, use first line + `\n` + second line as the anchor
3. For `file` nodes: the anchor IS the file path — use it directly in old_string
4. Escape pipe characters in table cells: `\|`

## How to generate from a .canvas file

The canvas JSON structure is:
```json
{
  "nodes": [
    {"id": "...", "type": "text", "text": "line1\nline2\n...", "x": N, "y": N, "width": N, "height": N},
    {"id": "...", "type": "file", "file": "path/to/file.md", "x": N, "y": N, ...}
  ],
  "edges": [
    {"id": "...", "fromNode": "...", "fromSide": "bottom", "toNode": "...", "toSide": "top"}
  ]
}
```

For each node: extract `id`, `type`, and the match anchor. Approximate line = JSON line number of that node's opening `{`.
For edges: group by `fromNode`, list all `toNode` values.

## JSON match anchor — critical note

Obsidian canvas stores node text as a JSON string value. Newlines within a node's text are stored as literal `\n` (two characters: backslash + n) in the JSON file on disk. When using the Edit tool's `old_string` to match across two lines of a node, use `\n` (the two-character sequence), NOT an actual newline.

Example — to match a node starting with `### Revenue Math` then `**Rev/client:**`:
- In the JSON file it reads: `"text":"### Revenue Math\n**Rev/client:**..."`
- Edit tool old_string: `### Revenue Math\n**Rev/client:**`
