# Chat Archive

This folder stores exported conversation history related to TinyWebSDR work.

## Recommended export format

Use an **HTML export** from the chat UI when available.
- Reason: best preservation of original layout/styles.
- Keep a PDF copy as a secondary reference if needed.

## Naming convention

Use timestamped filenames:

- `YYYY-MM-DD-session-title.html`
- `YYYY-MM-DD-session-title.pdf` (optional)
- `YYYY-MM-DD-session-title.json` (optional raw export)

Example:

- `2026-03-06-tinywebsdr-build-log.html`
- `2026-03-06-tinywebsdr-build-log.pdf`

## Minimal metadata note (optional)

For each session, you can add a short markdown note:

- `YYYY-MM-DD-session-title.md`

Template:

```md
# Session Note

- Date: 2026-03-06
- Source: Chat UI export
- Scope: TinyWebSDR SDR pipeline + scheduler
- Related commits:
  - 5030953
  - 5911da0
  - 08f0ce7
```

## Git tips

- Commit archive files in a dedicated commit message like:
  - `docs: archive chat session 2026-03-06`
- Large binary exports (very big PDF/ZIP) are better tracked with Git LFS.
