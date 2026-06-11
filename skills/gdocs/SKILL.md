---
name: gdocs
description: "Create and edit Google Docs with structured content using MCP tools. Supports bold/italic text, code blocks, and visual hierarchy via ALL CAPS headers. Use when users want to create documentation, technical guides, reports, memos, or any structured text document. Trigger when user mentions Google Docs, Google Document, creating a doc, writing documentation, or wants to produce a formatted text document."
version: 2.0.0
author: Paul
---

# Google Docs Skill

Create well-structured Google Docs documents using MCP tools.

## When to Use This Skill

- Creating documentation, guides, or reports in Google Docs
- Writing structured text with sections, code blocks, and bold labels
- Editing existing Google Docs content
- When you need to add content at specific positions

## Critical Limitations

### 1. `formatText` is Character-Level ONLY

**`docs_formatText` does NOT apply paragraph-level heading styles.**

When you pass `heading1`, `heading2`, etc., it applies visual styling (font size/weight) but NOT native headings. Results:
- No document outline integration
- Wrong paragraph spacing
- Formatting breaks on edits
- **431 paragraphs corrupted in one incident**

**Use `formatText` ONLY for:**
- `bold` - Bold text
- `italic` - Italic text
- `code` - Monospace font
- `link` - Clickable hyperlinks

**For visual hierarchy, use:**
- ALL CAPS for section headers
- Bold for subsection headers
- Bullets and numbered lists

### 2. `replaceText` Strips ALL Formatting

After any `replaceText` or `writeText` call, ALL formatting in the affected region is lost.

**Fix:** Re-apply formatting after every edit.

### 3. Index Calculation is Unreliable

Calculating indices from `getText` output accumulates drift from:
- Unicode characters (emojis are 2 UTF-16 units)
- Newlines and paragraph breaks
- Split text runs

**Fix:** Always fetch fresh text with `getText` before calculating indices.

## Pitfalls Reference

### Empty Doc Write Failure
**Problem:** `writeText` fails on empty docs.
**Fix:** Seed with placeholder first, then `replaceText`.

### Large Block Replace Appends
**Problem:** `replaceText` on large blocks appends instead of replacing.
**Fix:** For full rewrites, create NEW doc → seed → replace.

### Empty String Replace Fails
**Problem:** `replaceText` with empty string fails.
**Fix:** Match surrounding context + replace with minimal non-empty content.

### Doc Tabs Invisible
**Problem:** `getText` only returns first/default tab content.
**Fix:** No MCP tool for multi-tab docs. Ask user to consolidate.

### ASCII/Pipe Tables Render Poorly
**Problem:** Box dividers and pipe tables look broken in Google Docs.
**Fix:** Use bullets, em-dashes, and short paragraphs instead.

### Multiple Replaces Cause Corruption
**Problem:** Cascading replacements create artifacts like "chwitz-birkenauHEAD".
**Fix:** After all replacements, verify with `getText` and fix artifacts.

### Hyperlinks Need Special Handling
**Problem:** `replaceText` only produces plain text URLs.
**Fix:** Write URL label first, then apply `style: "link"` with `url` field.

## Content Design Rules

Since we can't apply native heading styles, use these patterns:

### DO Use:
- **ALL CAPS** for section headers (e.g., "OVERVIEW", "SETUP INSTRUCTIONS")
- **Bold text** for subsection headers and labels
- Bullet points and numbered lists
- Em-dash (`—`) separators between sections
- Short paragraphs with line breaks
- `code` style for commands and file paths

### NEVER Use:
- ASCII box dividers (`════════════`, `────────────`)
- Pipe-delimited tables (`| Col1 | Col2 |`)
- Fixed-width alignment

### Example Structure:
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

## Workflow

### Step 1: Create the Document

```javascript
google-workspace_docs_create({
  title: "Document Title",
  content: "Initial content here"
})
```

### Step 2: Calculate Formatting Indices

**CRITICAL**: Always fetch fresh text first.

```python
# After getting doc ID, fetch text
text = """fetched content"""  # From getText
offset = 1  # Google Docs body starts at index 1

# Find text to format
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
google-workspace_docs_formatText({
  documentId: "your-doc-id",
  formats: [
    {"startIndex": 1, "endIndex": 8, "style": "bold"},
    {"startIndex": 50, "endIndex": 65, "style": "code"}
  ]
})
```

### Step 4: Add Additional Content

```javascript
google-workspace_docs_writeText({
  documentId: "your-doc-id",
  text: "Additional content",
  position: 1234
})
```

## Formatting Styles Available

### Works Well:
- `bold` - Bold text
- `italic` - Italic text
- `code` - Monospace font for commands
- `strikethrough` - Strikethrough text
- `underline` - Underlined text
- `link` - Clickable hyperlinks (needs `url` field)

### Avoid:
- `heading1/2/3` - Character-level only, NOT native headings

## Best Practices

### DO:
1. **Always fetch fresh text** before calculating indices
2. **Batch all formatting** in one call
3. **Use ALL CAPS or bold** for section headers
4. **Re-apply formatting** after every edit
5. **Verify with getText** after formatting

### DON'T:
1. **Never use heading styles** - use bold/ALL CAPS instead
2. **Never guess indices** - always calculate
3. **Never edit without re-formatting** - edits strip formatting
4. **Never use pipe tables** - use bullets instead
5. **Never use ASCII dividers** - use em-dashes

## Index Calculation Template

```python
def calculate_formatting_indices(text, offset=1):
    """Calculate formatting indices for Google Docs."""
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

## Tools Reference

| Tool | Use For | Notes |
|------|---------|-------|
| `docs_create` | Create new doc | Returns documentId |
| `docs_getText` | Read content | Plain text only |
| `docs_formatText` | Bold, italic, code | NOT for headings! |
| `docs_writeText` | Insert text | Strips formatting |
| `docs_replaceText` | Replace text | Strips formatting! |

## Verification Workflow

After any edit or formatting:

1. `getText` to verify content
2. Check for artifacts from replaces
3. Re-apply formatting if needed
4. `getText` again to confirm

## Version History

- v2.0.0: Updated with validated pitfalls and MCP-only workarounds
- v1.0.0: Initial release