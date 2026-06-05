# Daily Tasks Landing Page — Design Spec

**Date:** 2026-06-05  
**Status:** Approved

---

## Overview

A single self-contained `index.html` file that opens directly in the browser (no server required). Serves as a personal daily task dashboard — open it each morning to see and manage the day's work. All data persists in localStorage.

---

## Layout & Theme

**Layout:** Priority Dashboard — tasks grouped into three sections (High / Medium / Low priority), each showing a remaining-count badge. A progress bar at the top shows today's overall completion.

**Theme:** Indigo Night — deep indigo-to-navy gradient background, indigo borders and accents, green progress indicators, red/amber/indigo priority labels.

---

## Data Model

Each task is a JSON object stored as an array in `localStorage` under the key `daily-tasks`:

```json
{
  "id": "uuid-v4",
  "title": "Review pull requests",
  "priority": "high",
  "category": "Work",
  "type": "recurring" | "one-off",
  "completedOn": "2026-06-05" | null
}
```

- **priority:** `"high"` | `"medium"` | `"low"`
- **type:** `"recurring"` tasks have their `completedOn` cleared at midnight; `"one-off"` tasks keep `completedOn` until the task is deleted
- **completedOn:** ISO date string (`YYYY-MM-DD`) of when the task was last checked off; `null` if not completed

A task is considered "done today" when `completedOn === today's date`.

---

## Daily Reset Logic

On every page load, the app compares each recurring task's `completedOn` date to today's date. If `completedOn` is not today, it is set to `null` (unchecked). This runs before render — no cron job, no service worker needed.

One-off tasks are never auto-reset. They carry over until the user manually deletes them.

---

## UI Components

### Header
- Left: Day of week (e.g., "Thursday") + full date ("June 5, 2026")
- Right: Progress label ("4 of 9 done") + gradient progress bar

### Add Task Bar
- Text input: task title (required)
- Category input: plain text field with a `<datalist>` showing all categories currently in use plus the three defaults (Work, Personal, Health); user types freely or picks from suggestions
- Priority dropdown: High / Medium / Low
- Type dropdown: One-off (default) / Recurring
- Add button: appends task, clears input

### Priority Sections (×3)
Each section has:
- Section header: label (e.g., "⬆ High Priority") + remaining count badge
- Task items (see below)
- Hidden when no tasks exist at that priority level

### Task Item
- Checkbox (click to toggle completion)
- Task title (struck-through when done)
- Category badge
- `↺ daily` badge for recurring tasks
- Delete button (×) on hover

### Footer
- Left: hint text — "Recurring tasks reset at midnight · One-off tasks carry over"
- Right: "Clear completed" button — removes all completed one-off tasks; leaves recurring tasks in place (they'll reset tomorrow)

---

## Features In Scope

- Add tasks with title, category, priority, type
- Check off / uncheck tasks
- Delete individual tasks
- Clear completed one-off tasks (bulk)
- Daily auto-reset for recurring tasks on page load
- Progress bar and count update live on every interaction
- All state persisted to localStorage

## Out of Scope

- Due dates or time-based reminders
- Drag-and-drop reordering
- Multiple lists or views
- Import/export
- Cloud sync

---

## Implementation

- **Single file:** `index.html` — HTML structure + `<style>` block + `<script>` block
- **No dependencies:** pure vanilla JS, no libraries, no build step
- **Entry point:** open `index.html` directly in any browser
- localStorage key: `"daily-tasks"`
- Categories are free-form strings; the dropdown pre-populates from all categories already used in tasks plus the three defaults

---

## File Structure

```
daily-tasks-landing/
  index.html          ← the entire app
  docs/
    superpowers/
      specs/
        2026-06-05-daily-tasks-design.md
```
