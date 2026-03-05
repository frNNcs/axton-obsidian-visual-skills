---
name: excalidraw-diagram
description: Generate Excalidraw diagrams from text content. Supports three output modes - Obsidian (.md), Standard (.excalidraw), and Animated (.excalidraw with animation order). Triggers on "Excalidraw", "draw diagram", "flowchart", "mind map", "visualization", "diagram", "standard Excalidraw", "standard excalidraw", "Excalidraw animation", "animated diagram", "animate".
metadata:
  version: 1.2.1
---

# Excalidraw Diagram Generator

Create Excalidraw diagrams from text content with multiple output formats.

## Output Modes

Select output mode based on user's trigger words:

| Trigger Words | Output Mode | File Format | Usage |
|--------|----------|----------|------|
| `Excalidraw`, `draw diagram`, `flowchart`, `mind map` | **Obsidian** (default) | `.md` | Open directly in Obsidian |
| `standard Excalidraw`, `standard excalidraw` | **Standard** | `.excalidraw` | Open/edit/share on excalidraw.com |
| `Excalidraw animation`, `animated diagram`, `animate` | **Animated** | `.excalidraw` | Drag to excalidraw-animate to generate animations |

## Workflow

1. **Detect output mode** from trigger words (see Output Modes table above)
2. Analyze content - identify concepts, relationships, hierarchy
3. Choose diagram type (see Diagram Types below)
4. Generate Excalidraw JSON (add animation order if Animated mode)
5. Output in correct format based on mode
6. **Automatically save to current working directory**
7. Notify user with file path and usage instructions

## Output Formats

### Mode 1: Obsidian Format (Default)

**Strictly output the following structure with no modifications:**

```markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'

# Excalidraw Data

## Text Elements
%%
## Drawing
\`\`\`json
{Complete JSON data}
\`\`\`
%%
```

**Key points:**
- Frontmatter must include `tags: [excalidraw]`
- Warning message must be complete
- JSON must be surrounded by `%%` markers
- Cannot use other frontmatter settings besides `excalidraw-plugin: parsed`
- **File extension**: `.md`

### Mode 2: Standard Excalidraw Format

Output a pure JSON file directly, can be opened at excalidraw.com:

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [...],
  "appState": {
    "gridSize": null,
    "viewBackgroundColor": "#ffffff"
  },
  "files": {}
}
```

**Key points:**
- `source` uses `https://excalidraw.com` (not Obsidian plugin)
- Pure JSON, no Markdown wrapper
- **File extension**: `.excalidraw`

### Mode 3: Animated Excalidraw Format

Same as Standard format, but each element adds a `customData.animate` field to control animation order:

```json
{
  "id": "element-1",
  "type": "rectangle",
  "customData": {
    "animate": {
      "order": 1,
      "duration": 500
    }
  },
  ...other standard fields
}
```

**Animation order rules:**
- `order`: Animation playback order (1, 2, 3...), smaller numbers appear first
- `duration`: Drawing duration for the element (milliseconds), default 500
- Elements with the same `order` appear simultaneously
- Recommended order: Title → Main frames → Connecting lines → Detail text

**How to use:**
1. Generate the `.excalidraw` file
2. Drag to https://dai-shi.github.io/excalidraw-animate/
3. Click Animate to preview, then export SVG or WebM

**File extension**: `.excalidraw`

---

## Diagram Types & Selection Guide

Choose the appropriate diagram form to enhance understanding and visual appeal.

| Type | Use Case | Approach |
|------|---------|------|
| **Flowchart** | Step-by-step instructions, workflows, task execution sequences | Connect steps with arrows, clearly express flow direction |
| **Mind Map** | Concept expansion, topic categorization, brainstorming | Radiate outward from center as core, radial structure |
| **Hierarchy** | Org charts, content hierarchy, system decomposition | Build hierarchical nodes top-down or left-to-right |
| **Relationship** | Influences, dependencies, interactions between elements | Show associations between shapes with connecting lines, arrows and labels |
| **Comparison** | Side-by-side analysis of two or more approaches or viewpoints | Left-right columns or table format, mark comparison dimensions |
| **Timeline** | Event development, project progress, model evolution | Use time as axis, mark key time points and events |
| **Matrix** | Two-dimensional categorization, task priority, positioning | Establish X and Y two dimensions, place on coordinate plane |
| **Freeform** | Scattered content, idea capture, initial information collection | No structural restrictions, freely place blocks and arrows |

## Design Rules

### Text & Format
- **All text elements must use** `fontFamily: 5` (Excalifont handwriting font)
- **Double quotes replacement rule in text**: replace `"` with `『』`
- **Parentheses replacement rule in text**: replace `()` with `「」`
- **Font size rules** (hard minimums, values below these are unreadable at normal zoom):
  - Title: 20-28px (minimum 20px)
  - Subtitle: 18-20px
  - Body/labels: 16-18px (minimum 16px)
  - Secondary annotations: 14px (use sparingly, only for unimportant auxiliary notes)
  - **Absolutely prohibited below 14px**
- **Line height**: use `lineHeight: 1.25` for all text
- **Text centering estimation**: standalone text elements have no auto-centering, must manually calculate x coordinate:
  - Estimate text width: `estimatedWidth = text.length * fontSize * 0.5` (use `* 1.0` for CJK characters)
  - Centering formula: `x = centerX - estimatedWidth / 2`
  - Example: text "Hello" (5 chars, fontSize 20) centered at x=300 → `estimatedWidth = 5 * 20 * 0.5 = 50` → `x = 300 - 25 = 275`

### Layout & Design
- **Canvas range**: recommended all elements within 0-1200 x 0-800 area
- **Minimum shape size**: rectangles/ellipses with text no smaller than 120x60px
- **Element spacing**: minimum 20-30px spacing to prevent overlap
- **Clear hierarchy**: use different colors and shapes to distinguish different levels of information
- **Graphic elements**: appropriately use rectangles, circles, arrows, and other elements to organize information
- **No Emoji**: do not use any Emoji symbols in diagram text; use simple shapes (circles, squares, arrows) or color coding for visual markers

### Color Palette

**Text color (strokeColor for text):**

| Usage | Color | Description |
|------|------|------|
| Title | `#1e40af` | Dark blue |
| Subtitle/connecting lines | `#3b82f6` | Bright blue |
| Body text | `#374151` | Dark gray (lightest on white background: no lighter than `#757575`) |
| Emphasis/highlights | `#f59e0b` | Gold |

**Shape fill colors (backgroundColor, fillStyle: "solid"):**

| Color | Semantic | Use Case |
|------|------|---------|
| `#a5d8ff` | Light blue | Input, data sources, main nodes |
| `#b2f2bb` | Light green | Success, output, completed |
| `#ffd8a8` | Light orange | Warning, pending, external dependencies |
| `#d0bfff` | Light purple | Processing, middleware, special items |
| `#ffc9c9` | Light red | Error, critical, alert |
| `#fff3bf` | Light yellow | Notes, decisions, planning |
| `#c3fae8` | Light cyan | Storage, data, cache |
| `#eebefa` | Light pink | Analysis, metrics, statistics |

**Area background colors (large rectangle + opacity: 30, for layered diagrams):**

| Color | Semantic |
|------|------|
| `#dbe4ff` | Frontend/UI layer |
| `#e5dbff` | Logic/processing layer |
| `#d3f9d8` | Data/tools layer |

**Contrast rules:**
- Text on white background no lighter than `#757575`, otherwise unreadable
- Use dark variant text on light fills (e.g., light green background use `#15803d`, not `#22c55e`)
- Avoid light gray text (`#b0b0b0`, `#999`) on white backgrounds

Reference: [references/excalidraw-schema.md](references/excalidraw-schema.md)

## JSON Structure

**Obsidian mode:**
```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://github.com/zsviczian/obsidian-excalidraw-plugin",
  "elements": [...],
  "appState": { "gridSize": null, "viewBackgroundColor": "#ffffff" },
  "files": {}
}
```

**Standard / Animated mode:**
```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [...],
  "appState": { "gridSize": null, "viewBackgroundColor": "#ffffff" },
  "files": {}
}
```

## Element Template

Each element requires these fields (do NOT add extra fields like `frameId`, `index`, `versionNonce`, `rawText` -- they may cause issues on excalidraw.com. `boundElements` must be `null` not `[]`, `updated` must be `1` not timestamps):

```json
{
  "id": "unique-id",
  "type": "rectangle",
  "x": 100, "y": 100,
  "width": 200, "height": 50,
  "angle": 0,
  "strokeColor": "#1e1e1e",
  "backgroundColor": "transparent",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid",
  "roughness": 1,
  "opacity": 100,
  "groupIds": [],
  "roundness": {"type": 3},
  "seed": 123456789,
  "version": 1,
  "isDeleted": false,
  "boundElements": null,
  "updated": 1,
  "link": null,
  "locked": false
}
```

`strokeStyle` values: `"solid"` (solid line, default) | `"dashed"` (dashed line) | `"dotted"` (dotted line). Dashed lines are suitable for optional paths, async flows, weak associations, etc.

Text elements add:
```json
{
  "text": "display text",
  "fontSize": 20,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "containerId": null,
  "originalText": "display text",
  "autoResize": true,
  "lineHeight": 1.25
}
```

**Animated mode additionally adds** `customData` field:
```json
{
  "id": "title-1",
  "type": "text",
  "customData": {
    "animate": {
      "order": 1,
      "duration": 500
    }
  },
  ...other fields
}
```

See [references/excalidraw-schema.md](references/excalidraw-schema.md) for all element types.

---

## Additional Technical Requirements

### Text Elements Processing
- The `## Text Elements` section in Markdown **must be left empty**, only `%%` is used as a delimiter
- The Obsidian ExcaliDraw plugin will **automatically populate text elements** based on JSON data
- No need to manually list all text content

### Coordinates and Layout
- **Coordinate system**: origin at top-left corner (0,0)
- **Recommended range**: all elements within 0-1200 x 0-800 pixel range
- **Element IDs**: each element needs a unique `id` (can be a string, e.g., "title", "box1", etc.)

### Required Fields for All Elements

**IMPORTANT**: Do NOT include `frameId`, `index`, `versionNonce`, or `rawText` fields. Use `boundElements: null` (not `[]`), and `updated: 1` (not timestamps).

```json
{
  "id": "unique-identifier",
  "type": "rectangle|text|arrow|ellipse|diamond",
  "x": 100, "y": 100,
  "width": 200, "height": 50,
  "angle": 0,
  "strokeColor": "#color-hex",
  "backgroundColor": "transparent|#color-hex",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid|dashed|dotted",
  "roughness": 1,
  "opacity": 100,
  "groupIds": [],
  "roundness": {"type": 3},
  "seed": 123456789,
  "version": 1,
  "isDeleted": false,
  "boundElements": null,
  "updated": 1,
  "link": null,
  "locked": false
}
```

### Text-Specific Properties
Text elements (type: "text") require additional properties (do NOT include `rawText`):
```json
{
  "text": "display text",
  "fontSize": 20,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "containerId": null,
  "originalText": "display text",
  "autoResize": true,
  "lineHeight": 1.25
}
```

### appState Configuration
```json
"appState": {
  "gridSize": null,
  "viewBackgroundColor": "#ffffff"
}
```

### files Field
```json
"files": {}
```

## Common Mistakes to Avoid

- **Text offset** — The `x` of a standalone text element is the left edge, not the center. Must manually calculate using the centering formula, otherwise text will shift to one side
- **Element overlap** — Elements with similar y coordinates tend to stack. Before placing a new element, check that there is at least 20px spacing from surrounding elements
- **Insufficient canvas margins** — Content should not be flush against canvas edges. Leave 50-80px padding on all sides
- **Title not centered on diagram** — Title should be centered on the overall width of the diagram below, not fixed at x=0
- **Arrow label overflow** — Long text labels (e.g., "ATP + NADPH") will extend beyond short arrows. Keep labels brief or increase arrow length
- **Insufficient contrast** — Light-colored text on white background is nearly invisible. Text color no lighter than `#757575`, colored text use dark variants
- **Font size too small** — Below 14px is unreadable at normal zoom, body text minimum 16px

## Implementation Notes

### Auto-save & File Generation Workflow

When generating an Excalidraw diagram, **the following steps must be executed automatically**:

#### 1. Select the appropriate diagram type
- Based on the characteristics of the content provided by the user, refer to the "Diagram Types & Selection Guide" table above
- Analyze the core requirements of the content, select the most appropriate visualization form

#### 2. Generate a meaningful filename

Select the file extension based on the output mode:

| Mode | Filename Format | Example |
|------|-----------|------|
| Obsidian | `[topic].[type].md` | `business-model.relationship.md` |
| Standard | `[topic].[type].excalidraw` | `business-model.relationship.excalidraw` |
| Animated | `[topic].[type].animate.excalidraw` | `business-model.relationship.animate.excalidraw` |

- Prioritize clarity and descriptiveness in the filename (use English names)

#### 3. Use the Write tool to auto-save the file
- **Save location**: current working directory (auto-detect environment variables)
- **Full path**: `{current_directory}/[filename].md`
- This enables flexible migration without hardcoding paths

#### 4. Ensure Markdown structure is completely correct
**Must generate in the following format** (no modifications allowed):

```markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'

# Excalidraw Data

## Text Elements
%%
## Drawing
\`\`\`json
{Complete JSON data}
\`\`\`
%%
```

#### 5. JSON data requirements
- Include complete Excalidraw JSON structure
- All text elements use `fontFamily: 5`
- Replace `"` in text with `『』`
- Replace `()` in text with `「」`
- JSON format must be valid, pass syntax check
- All elements have unique `id`
- Include `appState` and `files: {}` fields

#### 6. User feedback and confirmation
Report to user:
- Diagram has been generated
- Exact save location
- How to view in Obsidian
- Explanation of design choices (what type of diagram was chosen and why)
- Whether adjustments or modifications are needed

### Example Output Messages

**Obsidian mode:**
```
Excalidraw diagram generated!

Saved to: business-model.relationship.md

How to use:
1. Open this file in Obsidian
2. Click the MORE OPTIONS menu in the top right
3. Select Switch to EXCALIDRAW VIEW
```

**Standard mode:**
```
Excalidraw diagram generated!

Saved to: business-model.relationship.excalidraw

How to use:
1. Open https://excalidraw.com
2. Click the top-left menu → Open → Select this file
3. Or drag the file directly to the excalidraw.com page
```

**Animated mode:**
```
Excalidraw animated diagram generated!

Saved to: business-model.relationship.animate.excalidraw

Animation order: Title(1) → Main frames(2-4) → Connecting lines(5-7) → Detail text(8-10)

Generate animation:
1. Open https://dai-shi.github.io/excalidraw-animate/
2. Click Load File to select this file
3. Preview the animation
4. Click Export to export SVG or WebM
```
