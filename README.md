# Parthiban Kanban

A single-page Kanban board demo/training tool for an internal IT PMO ("UOB IT PMO" is a neutral placeholder wordmark, not a real branded system). The entire application — HTML, CSS, and JavaScript — lives in one file: `index.html`. No framework, no build step, no bundler.

## Live demo

https://parthik-byte.github.io/Parthiban-Kanban/

## Run it locally

Just open `index.html` directly in a browser (double-click it, or `Start-Process index.html` on Windows). There's no server, no `npm install`, and no build step required.

## Features

- Four-column board (Backlog / In Progress / Blocked / Done) with drag-and-drop cards, plus a keyboard-accessible "Move ▸" dropdown on every card as a parity fallback.
- Add Task form with inline validation (no native browser dialogs).
- Filtering by project, assignee, and priority.
- Inline delete confirmation (no `confirm()` popups).
- Optimistic "new task" notification via a FormSubmit AJAX endpoint — the card is added to the board immediately, and any notification failure shows as a non-blocking toast without affecting the board.
- Responsive layout that collapses to a single column below 768px.

## Board state

Board state lives only in an in-memory JavaScript object. Refreshing the page resets the board back to its seed data — this is intended behavior, not a bug, and there is no `localStorage`/`sessionStorage`/cookie persistence by design.
