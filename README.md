# Google Workspace Skills

A collection of skills for AI coding agents to interact with Google Workspace services.

## Install

```bash
npx skills add paulpan05/google-workspace-skills
```

## Available Skills

### gdocs

Create and edit Google Docs with structured content using MCP tools.

**Use when:** Creating documentation, technical guides, reports, memos, or any structured text document in Google Docs.

**Features:**
- Structured content with sections and code blocks
- Bold/italic text formatting
- Multi-section layouts
- Precise content positioning

**Limitations:**
- `formatText` applies character-level styling only (not native heading styles)
- `replaceText`/`writeText` strip all formatting
- Use ALL CAPS or bold for visual hierarchy instead of headings

---

*More skills coming soon:*
- `gsheets` - Google Sheets operations
- `gslides` - Google Slides presentations
- `gdrive` - Google Drive file management

## Prerequisites

- Google Workspace access
- Relevant Google APIs enabled

## License

MIT