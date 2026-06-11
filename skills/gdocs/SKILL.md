---
name: gdocs
description: "Create and edit Google Docs with structured content using MCP tools. Supports bold/italic text, code blocks, and visual hierarchy. Use when users want to create documentation, technical guides, reports, memos, or any structured text document. Trigger when user mentions Google Docs, Google Document, creating a doc, writing documentation, or wants to produce a formatted text document."
version: 2.1.0
---

# Google Docs Skill

Create well-structured Google Docs documents using MCP tools.

## When to Use This Skill

- Creating documentation, guides, or reports in Google Docs
- Writing structured text with sections, code blocks, and bold labels
- Editing existing Google Docs content
- When you need to add content at specific positions

## Tools Reference

| Tool | Use For | Notes |
|------|---------|-------|
| `docs_create` | Create new doc | Returns documentId |
| `docs_getText` | Read content | Plain text only |
| `docs_formatText` | Apply formatting | bold, italic, code, link |
| `docs_writeText` | Insert text | Need position index |
| `docs_replaceText` | Replace text | Strips formatting |

## Known Behaviors

### `replaceText` Strips Formatting

After any `replaceText` or `writeText` call, formatting in the affected region may be lost.

**Fix:** Re-apply formatting after every edit.

### Index Calculation

Always fetch fresh text with `getText` before calculating indices. Characters like `→`, `—`, emojis are handled correctly by Python `str.find()`.

### Empty Docs

`writeText` may fail on empty docs. Seed with placeholder content first.

## Content Design Rules

For reliable visual hierarchy that works across all scenarios:

### Recommended:
- **ALL CAPS** for section headers (e.g., "OVERVIEW", "SETUP INSTRUCTIONS")
- **Bold text** for subsection headers and labels
- Bullet points and numbered lists for structured data
- Em-dash (`—`) or en-dash separators between sections
- Short paragraphs with line breaks
- `code` style for commands and file paths

### Avoid:
- ASCII box dividers (`════════════`, `────────────`) - render poorly
- Pipe-delimited tables (`| Col1 | Col2 |`) - don't render as tables
- Fixed-width alignment - breaks at different zoom sizes

## Workflow

### Step 1: Create the Document

```javascript
docs_create({
  title: "Document Title",
  content: "Initial content here"
})
```

### Step 2: Calculate Formatting Indices

Fetch fresh text, then calculate:

```python
text = """fetched content"""  # From docs_getText
offset = 1  # Google Docs body starts at index 1

items = [
    ("OVERVIEW", "bold"),
    ("COMMANDS", "bold"),
    ("sudo command", "code"),
]

for text_to_find, style in items:
    idx = text.find(text_to_find)
    if idx >= 0:
        print(f'{{"startIndex": {idx+offset}, "endIndex": {idx+len(text_to_find)+offset}, "style": "{style}"}}')
```

### Step 3: Apply Formatting

```javascript
docs_formatText({
  documentId: "your-doc-id",
  formats: [
    {"startIndex": 1, "endIndex": 8, "style": "bold"},
    {"startIndex": 50, "endIndex": 65, "style": "code"}
  ]
})
```

### Step 4: Add Additional Content

```javascript
docs_writeText({
  documentId: "your-doc-id",
  text: "Additional content",
  position: 1234
})
```

## Formatting Styles

### Reliable:
- `bold` - Bold text
- `italic` - Italic text
- `code` - Monospace font for commands
- `strikethrough` - Strikethrough text
- `underline` - Underlined text
- `link` - Clickable hyperlinks (needs `url` field)

### Heading Styles:
- `heading1`, `heading2`, `heading3` - May apply visual styling
- For guaranteed structure, use ALL CAPS + bold instead

## Best Practices

1. **Always fetch fresh text** before calculating indices
2. **Batch all formatting** in one call
3. **Re-apply formatting** after every edit
4. **Use ALL CAPS or bold** for section headers
5. **Verify with getText** after formatting

## Index Calculation Template

```python
def calculate_formatting_indices(text, offset=1):
    results = []
    
    # Bold labels (ALL CAPS headers)
    bold_items = ["OVERVIEW", "SETUP", "COMMANDS", "TROUBLESHOOTING"]
    for item in bold_items:
        idx = text.find(item)
        if idx >= 0:
            results.append({
                "startIndex": idx + offset,
                "endIndex": idx + len(item) + offset,
                "style": "bold"
            })
    
    # Code blocks (commands)
    code_items = ["sudo ", "command --", "/path/to/"]
    for item in code_items:
        start = 0
        while True:
            idx = text.find(item, start)
            if idx == -1:
                break
            end = text.find("\n", idx)
            if end == -1:
                end = len(text)
            results.append({
                "startIndex": idx + offset,
                "endIndex": end + offset,
                "style": "code"
            })
            start = end
    
    return results
```

## Example Structure

```
OVERVIEW
Description text here.

SETUP INSTRUCTIONS
Step 1: Do this
Step 2: Do that

COMMANDS
sudo command --option value
another-command --flag
```

## Version History

- v2.1.0: Removed unverified claims, focused on tested behaviors
- v2.0.0: Added limitations and workarounds
- v1.0.0: Initial release