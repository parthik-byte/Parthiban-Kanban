# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page Kanban board for an internal IT PMO demo/training tool ("UOB IT PMO" — a neutral placeholder wordmark, not a real branded system). The entire application — HTML, CSS, and JavaScript — lives in one file: `index.html`.

## Commands

There is no build, lint, test, or package tooling in this repo (no `package.json`, no bundler, no framework).

- **Run it**: open `index.html` directly in a browser (double-click, or `Start-Process index.html` on Windows). No server, no `npm install`, no build step. It must keep working this way — do not introduce anything that requires a dev server or bundling.
- **Verify a change**: manually exercise the UI in a browser (drag a card, use the "Move ▸" select, submit the Add Task form with valid/invalid data, resize below 768px, refresh to confirm the board resets to seed data). There are no automated tests.

## Hard constraints (do not violate these when editing)

- **Vanilla only**: no React/Vue/jQuery/Tailwind, no build step, no bundler, no npm, no CDN scripts, no Google Fonts, no external image files. System font stack + inline SVG/Unicode glyphs only.
- **Single file**: all markup, one `<style>` block, and one `<script>` block stay in `index.html`. Don't split into separate `.css`/`.js` files.
- **No persistence, on purpose**: board state is an in-memory JS object only. Never add `localStorage`, `sessionStorage`, `IndexedDB`, or cookies — a page refresh resetting to seed data is intended behavior, not a bug, and the UI has a note saying so.
- **No native browser dialogs**: no `alert()`/`confirm()`. Delete confirmation and form validation errors are both inline UI (see the `delete-confirm` toggle and `.field-error` spans).
- **FormSubmit is the only backend call**: the Add Task flow posts to the FormSubmit AJAX endpoint (`formsubmit.co/ajax/...`) and nowhere else. It's optimistic: the card is added to the board and rendered immediately; `notifyNewTask()` runs in parallel and any failure is caught and shown as a non-blocking warning toast — it must never remove the card or block the UI.

## Architecture (all inside `index.html`)

**State**: a single `state = { tasks: [], filters: {...}, nextIdCounter, pendingDeleteId }` object is the sole source of truth. There is no other place task data lives. `seedTasks()` populates 8 demo tasks on load via `makeTask()`, which also mints IDs in the `UOB-ITPM-####` format from `state.nextIdCounter`.

**Render pipeline**: `renderBoard()` is the single entry point that rebuilds the whole board (columns + cards + summary strip) from `state` on every change — there is no fine-grained DOM patching. Any state mutation (`moveTask`, `deleteTask`, `addTask`, filter changes, toggling `pendingDeleteId` for the inline delete-confirm) ends by calling `renderBoard()` again. `renderCard()` builds one card's markup and wires its own event listeners (delete trigger, move `<select>`, drag handlers) each time it's called — don't try to mutate a card in place from outside these two functions.

**Filtering**: `getFilteredTasks()` derives the visible task list from `state.tasks` + `state.filters` (project, assignee substring, priority) — it never mutates `state.tasks` itself. `renderBoard()` renders columns from the filtered list, but per-column counts and the header summary strip (`renderSummary()`) are computed off the *unfiltered* `state.tasks`.

**Drag and drop**: native HTML5 DnD (`draggable`, `dragstart`/`dragover`/`dragleave`/`drop`) on cards and column drop zones, calling `moveTask(id, newStatus)`. Every card also renders a keyboard-accessible "Move ▸" `<select>` as a parity fallback that calls the same `moveTask()` — when changing how moves work, update both paths.

**Escaping**: all task-derived strings (title, description, assignee, etc.) go through `escapeHtml()` before being interpolated into template strings and assigned via `innerHTML`. Any new field that renders user-supplied text must go through it too — there's no other sanitization layer.

**FormSubmit integration**: `FORMSUBMIT_ENDPOINT` is a single named constant near the top of the `<script>` block (currently a placeholder address) — it's the one place to swap in a real destination. `notifyNewTask()` is always called inside a `try/catch` from the form submit handler, after the optimistic `addTask()` + `renderBoard()` has already happened. FormSubmit requires a one-time email confirmation click before it starts delivering to a new address — a failed/unconfirmed call still shouldn't affect the board, just the toast shown.

**CSS gotcha to remember**: the modal overlay is toggled via the `hidden` attribute, not inline styles or a class. Because `.modal-overlay { display: flex; ... }` is an author-stylesheet rule, it silently overrides the browser's built-in `[hidden] { display: none }` rule (author CSS wins over user-agent CSS regardless of specificity). There's an explicit `.modal-overlay[hidden] { display: none; }` rule to counter this — if you add other elements that are shown/hidden via the `hidden` attribute while also having their own `display` rule, they need the same `[hidden]` override or they'll never actually hide.

**Responsive layout**: the board is a 4-column CSS grid that collapses to a single column via `@media (max-width: 768px)`. Column order and count are fixed (Backlog / In Progress / Blocked / Done, defined in the `COLUMNS` array) — don't hardcode column-specific markup elsewhere; everything iterates over `COLUMNS`.
