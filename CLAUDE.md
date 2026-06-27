# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

This is a **single-file web app** — the entire application lives in `index.html`. There is no build step, no package manager, and no separate source files.

### How the bundler works

`index.html` contains three `<script>` tags with custom types that act as a self-contained bundle:

- `__bundler/manifest` — a JSON map of UUIDs to base64-encoded binary assets (fonts, the React runtime, etc.)
- `__bundler/template` — a JSON-encoded string of the full app HTML (including its own `<head>`, styles, and component markup)
- `__bundler/ext_resources` — external resource declarations (currently empty)

On load, the bundler script unpacks the manifest assets into Blob URLs, substitutes them into the template string, parses the result with `DOMParser`, then calls `document.documentElement.replaceWith(doc.documentElement)` to swap the entire page. This means the outer `<head>` (including `<title>`) is discarded — any tags that need to survive must be inside the `__bundler/template` HTML string.

### App logic

The template embeds a `<script type="text/x-dc">` block containing a `Component` class that extends `DCLogic` (a reactive UI framework loaded as a bundled asset). State is persisted to `localStorage` under the key `routineBoard.v1`.

Key state shape:
- `kids` — array of `{id, name, accent, accentDeep, accentSoft}`
- `tasks` — per-kid object keyed by category (`morning`, `evening`, `chore`), each an array of task objects
- `lists` — custom task lists with kid assignments
- `done` — per-kid map of `"modeKey:taskId" → true`
- `viewMode`, `activeCat`, `showCompleted` — UI state per kid

## Development

There are no build commands. Edit `index.html` directly and open it in a browser (or push to trigger a Vercel deploy).

**Deployed on Vercel** — every push to `main` triggers an automatic redeploy.

## Key editing constraints

- The `__bundler/template` value is a single JSON-encoded string on one line. When editing template HTML (markup, styles, title), the content must remain valid JSON — escape `"` as `\"`, `<\/script>` tags as `<\/script>`, and newlines as `\n`.
- To change the browser tab title, update the `<title>` tag **inside the template string** (not the outer `<head>`), since the bundler replaces `documentElement` at runtime.
