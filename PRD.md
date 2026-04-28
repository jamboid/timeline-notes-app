# Timeline Notes App — PRD

## Overview

A client-side web app that reads a user-selected folder of markdown files and displays them as a chronological, fully-editable timeline. No backend. No install. No separate edit mode.

---

## Core Decisions

### Platform & Stack
- **Browser-based web app** using the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API)
- **Vue 3 + TypeScript + Vite**
- User grants folder access once; the grant persists across sessions
- No Electron, no server, no build-time file processing

### File Format
- Source of truth: a flat folder of `.md` files on the user's filesystem
- Files in a `_trash/` subfolder are ignored by the timeline
- Timestamp-based filenames on creation: `2026-04-28T143022.md`
- **Date field in frontmatter** determines timeline position; falls back to file last-modified date if absent

```yaml
---
date: 2026-04-28T14:30:00
---
```

---

## Features

### Timeline View
- Single-column, centered layout, ~720px max-width
- Fluid/responsive — desktop-first (File System Access API is not available on iOS Safari)
- Continuous vertical scroll; all notes rendered in full
- Notes grouped by **day** with a date section header
- Newest notes at the top

### Note Titles
- Derived from the first `# H1` in the markdown content
- Falls back to the filename (stripped of timestamp prefix and `.md` extension) if no H1 is present

### Inline Editing
- Notes render as HTML by default
- **Clicking a note** swaps it to a raw markdown textarea — no separate edit mode
- Clicking away (blur) or pressing **Cmd+S** saves the file immediately
- **Auto-save** on keystroke with a 1-second debounce
- No save button in the UI

### New Note Creation
- Fixed **"+" button** (top-right of UI)
- Creates a new `.md` file instantly with a timestamp filename
- Inserts the note at the top of today's group
- **Auto-focuses** the textarea so typing begins immediately

### Deletion
- Deleting a note **moves it to a `_trash/` subfolder** within the notes directory
- No permanent deletion from the UI
- The app ignores files in `_trash/` when building the timeline

### Search
- Full-text search bar, always visible
- **Cmd+F** focuses the search field
- In-memory filter — no indexing required
- Filters the timeline to notes whose content or title matches the query

### External Edit Detection
- The app **polls the notes directory** every few seconds
- If a file is added, modified, or removed externally (e.g. edited in VS Code), the timeline updates automatically

### Markdown Styling
- Rendered markdown output is styled via a **dedicated, user-editable CSS file** (`notes.css` or similar)
- Controls typography: font sizes, colors, spacing for headings, body text, code, blockquotes, etc.
- Intentionally separate from the app UI styles so the user can customize without touching component code

---

## Out of Scope (v1)

- Images / local file embeds
- Tags / categories
- Sidebar navigation
- Note reordering within a day group
- Mobile support (beyond fluid layout)
- Backend / sync / cloud storage

---

## Open Questions

- **Markdown rendering library**: `marked` vs `markdown-it` — recommend `markdown-it` for its plugin ecosystem if extensions (tables, footnotes) are added later
- **State management**: Pinia vs local Vue composables — composables likely sufficient for v1
- **Polling interval**: 3–5 seconds is a reasonable default; could be user-configurable later
