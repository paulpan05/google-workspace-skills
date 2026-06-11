---
name: docs
description: Create, format, and edit Google Docs documents with proper structure and formatting. Use when users want to create documentation, write technical guides, create reports, memos, or any structured text document in Google Docs. Supports headings, code blocks, bold text, and multi-section layouts. Trigger when user mentions Google Docs, Google Document, creating a doc, writing documentation, or wants to produce a formatted text document.
---

# Google Docs Skill

Create well-formatted Google Docs documents with proper structure, headings, code blocks, and text formatting.

## Overview

This skill helps create professional Google Docs with:
- Proper heading hierarchy (H1, H2, H3)
- Code blocks for commands and configurations
- Bold text for labels and emphasis
- Multi-section layouts
- Proper content organization

## Workflow

### Step 1: Create the Document

Use the `google-workspace_docs_create` tool to create a new Google Doc:

```javascript
google-workspace_docs_create({
  title: "Document Title",
  content: "Initial content here"
})
```

### Step 2: Calculate Formatting Indices

**CRITICAL**: Before applying any formatting, you MUST calculate exact character indices. Formatting applied at wrong positions will break the document.

Use Python to calculate indices:

```python
text = """Your document content here"""
offset = 1  # Google Docs body starts at index 1

# Find headings
headings = ["Section 1", "Section 2", "Subsection"]
for h in headings:
    idx = text.find(h)
    print(f'{{"startIndex": {idx+offset}, "endIndex": {idx+len(h)+offset}, "style": "heading2"}}')
```

### Step 3: Apply Formatting

Use `google-workspace_docs_formatText` with calculated indices:

```javascript
google-workspace_docs_formatText({
  documentId: "your-doc-id",
  formats: [
    {"startIndex": 1, "endIndex": 10, "style": "heading1"},
    {"startIndex": 50, "endIndex": 65, "style": "heading2"},
    {"startIndex": 100, "endIndex": 120, "style": "code"}
  ]
})
```

### Step 4: Add Additional Content

Use `google-workspace_docs_writeText` to append or insert content:

```javascript
google-workspace_docs_writeText({
  documentId: "your-doc-id",
  text: "Additional content",
  position: 1234  // Character index where to insert
})
```

## Formatting Styles

### Headings
- `heading1` - Main title
- `heading2` - Section headers (1., 2., 3., etc.)
- `heading3` - Subsection headers

### Text Formatting
- `code` - Monospace font for commands and code
- `bold` - Bold text for labels

## Best Practices

### DO:
1. **Always calculate indices** before applying formatting
2. **Create content first**, then format
3. **Use Python** to find exact character positions
4. **Apply all formatting in one call** when possible
5. **Verify indices** match the actual text positions

### DON'T:
1. **Never guess indices** - this breaks formatting
2. **Never apply formatting multiple times** - creates conflicts
3. **Never use replaceText** for formatting changes
4. **Never skip index calculation** - even for "simple" docs

## Index Calculation Template

```python
def calculate_indices(text, offset=1):
    """Calculate formatting indices for Google Docs."""
    
    # Headings
    h1 = ["Overview"]
    h2 = ["1. Section One", "2. Section Two"]
    h3 = ["Subsection A", "Subsection B"]
    
    # Code blocks
    code_blocks = [
        "command --option value",
        "config_file_path"
    ]
    
    # Bold labels
    bold_labels = ["Label:", "Server:"]
    
    results = {"h1": [], "h2": [], "h3": [], "code": [], "bold": []}
    
    for h in h1:
        idx = text.find(h)
        if idx >= 0:
            results["h1"].append({"startIndex": idx+offset, "endIndex": idx+len(h)+offset})
    
    for h in h2:
        idx = text.find(h)
        if idx >= 0:
            results["h2"].append({"startIndex": idx+offset, "endIndex": idx+len(h)+offset})
    
    # ... similar for h3, code, bold
    
    return results
```

## Example: Technical Documentation

```markdown
# Document Title

## Overview
Description text here.

## 1. Setup
Configuration steps:

[Config block here]

## 2. Commands
Run these commands:

sudo command --option
```

## Troubleshooting

### Formatting at Wrong Positions
- **Cause**: Indices calculated incorrectly
- **Fix**: Re-calculate using Python, verify with `text.find()`

### Content Not Appearing
- **Cause**: Position index out of range
- **Fix**: Use `google-workspace_docs_getText` to get current content length

### Formatting Conflicts
- **Cause**: Multiple formatting calls overlapping
- **Fix**: Create fresh document, apply formatting once

## Tools Reference

- `google-workspace_docs_create` - Create new document
- `google-workspace_docs_getText` - Read document content
- `google-workspace_docs_formatText` - Apply formatting
- `google-workspace_docs_writeText` - Add/insert text
- `google-workspace_docs_replaceText` - Replace text (use sparingly)

## Important Notes

1. **Index calculation is mandatory** - Never skip this step
2. **One formatting call** - Apply all styles in a single call
3. **Verify before applying** - Check indices match text positions
4. **Test with simple docs first** - Build complexity gradually