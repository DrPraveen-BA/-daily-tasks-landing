# Daily Tasks Landing Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single self-contained `index.html` daily task dashboard with priority grouping, recurring task auto-reset, and localStorage persistence.

**Architecture:** One HTML file with an inline `<style>` block (Indigo Night theme) and an inline `<script>` block (vanilla JS). All state lives in `localStorage` under the key `"daily-tasks"`. On every page load, the script runs a daily reset pass before rendering, clearing `completedOn` for any recurring task not completed today.

**Tech Stack:** HTML5, CSS3, vanilla JavaScript (ES2020), localStorage API — no build step, no dependencies.

---

## File Structure

```
daily-tasks-landing/
  index.html       ← entire app: HTML skeleton + <style> + <script>
```

All logic is in `index.html`. The script is organized into four clearly-labeled sections via comments:
1. **State & Storage** — load/save tasks, `todayStr()`
2. **Daily Reset** — clears `completedOn` for stale recurring tasks
3. **Render** — builds DOM from state
4. **Actions** — add, toggle, delete, clear

---

## Task 1: HTML Skeleton + CSS (Indigo Night Theme)

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create `index.html` with full HTML structure and CSS**

Write the complete file. The `<script>` block can be empty for now — we're building the shell first.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Daily Tasks</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #0d0d1a;
      min-height: 100vh;
      display: flex;
      align-items: flex-start;
      justify-content: center;
      padding: 32px 16px 64px;
      color: #e0e7ff;
    }

    .app {
      width: 100%;
      max-width: 680px;
      background: linear-gradient(135deg, #1e1b4b 0%, #0f172a 60%, #14172b 100%);
      border-radius: 16px;
      border: 1px solid rgba(99,102,241,0.3);
      overflow: hidden;
      box-shadow: 0 25px 60px rgba(0,0,0,0.5), 0 0 0 1px rgba(99,102,241,0.1);
    }

    /* ── Header ── */
    .header {
      padding: 24px 28px 20px;
      border-bottom: 1px solid rgba(99,102,241,0.2);
      display: flex;
      align-items: flex-start;
      justify-content: space-between;
      gap: 16px;
    }
    .date-block .day-name {
      font-size: 12px;
      color: #818cf8;
      font-weight: 600;
      letter-spacing: 2px;
      text-transform: uppercase;
    }
    .date-block .full-date {
      font-size: 22px;
      color: #e0e7ff;
      font-weight: 700;
      margin-top: 2px;
    }
    .progress-block {
      display: flex;
      flex-direction: column;
      align-items: flex-end;
      gap: 6px;
      padding-top: 4px;
    }
    .progress-label {
      font-size: 11px;
      color: #34d399;
      font-weight: 600;
      letter-spacing: 0.5px;
    }
    .progress-bar-wrap {
      width: 120px;
      height: 6px;
      background: rgba(99,102,241,0.2);
      border-radius: 99px;
      overflow: hidden;
    }
    .progress-bar {
      height: 100%;
      background: linear-gradient(90deg, #6366f1, #34d399);
      border-radius: 99px;
      transition: width 0.3s ease;
    }

    /* ── Add task bar ── */
    .add-bar {
      padding: 14px 28px;
      border-bottom: 1px solid rgba(99,102,241,0.15);
      display: flex;
      gap: 8px;
      align-items: center;
      flex-wrap: wrap;
    }
    .add-bar input[type="text"],
    .add-bar select {
      background: rgba(99,102,241,0.1);
      border: 1px solid rgba(99,102,241,0.25);
      border-radius: 8px;
      padding: 8px 12px;
      color: #c7d2fe;
      font-size: 13px;
      outline: none;
      font-family: inherit;
    }
    .add-bar input[type="text"]::placeholder { color: #4338ca; }
    .add-bar input[type="text"]:focus,
    .add-bar select:focus {
      border-color: rgba(99,102,241,0.6);
    }
    #title-input { flex: 1; min-width: 140px; }
    #category-input { width: 110px; }
    #priority-select { width: 100px; }
    #type-select { width: 110px; }
    .add-bar select option { background: #1e1b4b; }
    .add-btn {
      background: #6366f1;
      border: none;
      border-radius: 8px;
      padding: 8px 16px;
      color: #fff;
      font-size: 13px;
      font-weight: 600;
      cursor: pointer;
      white-space: nowrap;
      transition: background 0.15s;
    }
    .add-btn:hover { background: #4f46e5; }

    /* ── Priority sections ── */
    .sections { padding: 8px 0; }
    .priority-section { padding: 12px 28px; }
    .priority-section + .priority-section {
      border-top: 1px solid rgba(99,102,241,0.12);
    }
    .section-header {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-bottom: 10px;
    }
    .section-label {
      font-size: 10px;
      font-weight: 700;
      letter-spacing: 2px;
      text-transform: uppercase;
    }
    .section-label.high  { color: #f87171; }
    .section-label.medium { color: #fbbf24; }
    .section-label.low   { color: #818cf8; }
    .section-count {
      font-size: 10px;
      color: #4338ca;
      background: rgba(99,102,241,0.15);
      padding: 1px 8px;
      border-radius: 99px;
    }

    /* ── Task items ── */
    .task-item {
      display: flex;
      align-items: center;
      gap: 10px;
      background: rgba(99,102,241,0.08);
      border: 1px solid rgba(99,102,241,0.15);
      border-radius: 8px;
      padding: 10px 12px;
      margin-bottom: 6px;
      transition: background 0.15s;
      position: relative;
    }
    .task-item:last-child { margin-bottom: 0; }
    .task-item:hover { background: rgba(99,102,241,0.13); }
    .task-item.done { opacity: 0.5; }

    .task-checkbox {
      width: 16px;
      height: 16px;
      border: 1.5px solid #4338ca;
      border-radius: 4px;
      flex-shrink: 0;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background 0.15s, border-color 0.15s;
      user-select: none;
    }
    .task-checkbox.checked {
      background: #34d399;
      border-color: #34d399;
    }
    .task-checkbox.checked::after {
      content: '';
      display: block;
      width: 8px;
      height: 5px;
      border-left: 2px solid #fff;
      border-bottom: 2px solid #fff;
      transform: rotate(-45deg) translateY(-1px);
    }

    .task-title {
      font-size: 13px;
      color: #c7d2fe;
      flex: 1;
    }
    .task-item.done .task-title {
      text-decoration: line-through;
      color: #4338ca;
    }

    .category-badge {
      font-size: 10px;
      padding: 2px 8px;
      border-radius: 99px;
      background: rgba(99,102,241,0.2);
      color: #818cf8;
      flex-shrink: 0;
      white-space: nowrap;
    }
    .recurring-badge {
      font-size: 9px;
      padding: 1px 6px;
      border-radius: 99px;
      background: rgba(52,211,153,0.1);
      color: #34d399;
      border: 1px solid rgba(52,211,153,0.25);
      flex-shrink: 0;
      white-space: nowrap;
    }
    .delete-btn {
      background: none;
      border: none;
      color: #4338ca;
      cursor: pointer;
      font-size: 14px;
      line-height: 1;
      padding: 2px 4px;
      border-radius: 4px;
      opacity: 0;
      transition: opacity 0.15s, color 0.15s;
      flex-shrink: 0;
    }
    .task-item:hover .delete-btn { opacity: 1; }
    .delete-btn:hover { color: #f87171; }

    /* ── Empty state ── */
    .empty-state {
      text-align: center;
      padding: 40px 28px;
      color: #312e81;
      font-size: 13px;
    }

    /* ── Footer ── */
    .footer {
      padding: 12px 28px;
      border-top: 1px solid rgba(99,102,241,0.15);
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
      flex-wrap: wrap;
    }
    .footer-hint {
      font-size: 11px;
      color: #312e81;
    }
    .clear-btn {
      font-size: 11px;
      color: #4338ca;
      cursor: pointer;
      background: none;
      border: none;
      font-family: inherit;
      transition: color 0.15s;
    }
    .clear-btn:hover { color: #818cf8; }
  </style>
</head>
<body>
  <div class="app">
    <!-- Header -->
    <div class="header">
      <div class="date-block">
        <div class="day-name" id="day-name"></div>
        <div class="full-date" id="full-date"></div>
      </div>
      <div class="progress-block">
        <div class="progress-label" id="progress-label">0 of 0 done</div>
        <div class="progress-bar-wrap">
          <div class="progress-bar" id="progress-bar" style="width:0%"></div>
        </div>
      </div>
    </div>

    <!-- Add task bar -->
    <div class="add-bar">
      <input type="text" id="title-input" placeholder="Add a task..." autocomplete="off">
      <input type="text" id="category-input" placeholder="Category" list="category-list" autocomplete="off">
      <datalist id="category-list"></datalist>
      <select id="priority-select">
        <option value="high">High</option>
        <option value="medium" selected>Medium</option>
        <option value="low">Low</option>
      </select>
      <select id="type-select">
        <option value="one-off" selected>One-off</option>
        <option value="recurring">Recurring</option>
      </select>
      <button class="add-btn" id="add-btn">+ Add</button>
    </div>

    <!-- Priority sections injected here by JS -->
    <div class="sections" id="sections"></div>

    <!-- Footer -->
    <div class="footer">
      <span class="footer-hint">Recurring tasks reset at midnight &middot; One-off tasks carry over</span>
      <button class="clear-btn" id="clear-btn">Clear completed</button>
    </div>
  </div>

  <script>
    // Script filled in by subsequent tasks
  </script>
</body>
</html>
```

- [ ] **Step 2: Open `index.html` in browser and verify the shell renders**

Open the file directly (`File > Open` or drag into browser). Expected:
- Dark indigo gradient page with the app card centered
- Header area (date placeholders are blank — JS not wired yet)
- Empty add-bar with all four inputs visible
- Empty sections area
- Footer with hint text and "Clear completed" button
- No console errors

- [ ] **Step 3: Commit**

```bash
cd daily-tasks-landing
git add index.html
git commit -m "feat: HTML skeleton and Indigo Night CSS theme"
```

---

## Task 2: State, Storage, and Daily Reset Logic

**Files:**
- Modify: `index.html` — replace the empty `<script>` block

- [ ] **Step 1: Write the State & Storage section of the script**

Replace `// Script filled in by subsequent tasks` with:

```javascript
// ── State & Storage ──────────────────────────────────────────────────────────

const STORAGE_KEY = 'daily-tasks';

function todayStr() {
  return new Date().toISOString().slice(0, 10); // "YYYY-MM-DD"
}

function loadTasks() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
  } catch {
    return [];
  }
}

function saveTasks(tasks) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function generateId() {
  return Date.now().toString(36) + Math.random().toString(36).slice(2);
}

// ── Daily Reset ───────────────────────────────────────────────────────────────

function applyDailyReset(tasks) {
  const today = todayStr();
  return tasks.map(task => {
    if (task.type === 'recurring' && task.completedOn !== today) {
      return { ...task, completedOn: null };
    }
    return task;
  });
}

// ── Bootstrap ─────────────────────────────────────────────────────────────────

let tasks = applyDailyReset(loadTasks());
saveTasks(tasks); // persist reset immediately
```

- [ ] **Step 2: Verify reset logic manually in browser console**

Open `index.html` in browser. Open DevTools console and run:

```javascript
// Seed a recurring task completed yesterday
localStorage.setItem('daily-tasks', JSON.stringify([
  { id: 'test1', title: 'Morning standup', priority: 'low', category: 'Work',
    type: 'recurring', completedOn: '2020-01-01' },
  { id: 'test2', title: 'Fix bug', priority: 'high', category: 'Work',
    type: 'one-off', completedOn: '2020-01-01' }
]));
```

Reload the page. Then in console:

```javascript
JSON.parse(localStorage.getItem('daily-tasks'))
```

Expected output:
- `test1` (`recurring`): `completedOn` is `null` (reset because 2020-01-01 ≠ today)
- `test2` (`one-off`): `completedOn` is still `"2020-01-01"` (not reset)

Clear localStorage after test:
```javascript
localStorage.removeItem('daily-tasks');
```
Reload to start fresh.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: state storage and daily reset logic"
```

---

## Task 3: Render — Header and Progress Bar

**Files:**
- Modify: `index.html` — add render functions to script after bootstrap block

- [ ] **Step 1: Add `renderHeader()` and `render()` skeleton to script**

Append after the bootstrap block:

```javascript
// ── Render ────────────────────────────────────────────────────────────────────

const DAYS = ['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'];
const MONTHS = ['January','February','March','April','May','June',
                'July','August','September','October','November','December'];

function renderHeader() {
  const now = new Date();
  document.getElementById('day-name').textContent = DAYS[now.getDay()];
  document.getElementById('full-date').textContent =
    `${MONTHS[now.getMonth()]} ${now.getDate()}, ${now.getFullYear()}`;

  const today = todayStr();
  const total = tasks.length;
  const done = tasks.filter(t => t.completedOn === today).length;

  document.getElementById('progress-label').textContent =
    `${done} of ${total} done`;
  const pct = total === 0 ? 0 : Math.round((done / total) * 100);
  document.getElementById('progress-bar').style.width = `${pct}%`;
}

function render() {
  renderHeader();
  // sections rendered in Task 4
}

// Initial render
render();
```

- [ ] **Step 2: Verify header renders in browser**

Open `index.html`. Expected:
- Day name shows current day (e.g., "Thursday")
- Full date shows today's date (e.g., "June 5, 2026")
- Progress shows "0 of 0 done" with progress bar at 0%
- No console errors

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: render header with date and progress bar"
```

---

## Task 4: Render — Priority Sections and Task Items

**Files:**
- Modify: `index.html` — add `renderSections()` and `renderTask()`, wire into `render()`

- [ ] **Step 1: Add section and task rendering functions**

Add these functions after `renderHeader()`, before `render()`:

```javascript
const PRIORITY_CONFIG = [
  { key: 'high',   label: '⬆ High Priority',   cls: 'high'   },
  { key: 'medium', label: '→ Medium Priority',  cls: 'medium' },
  { key: 'low',    label: '↓ Low Priority',     cls: 'low'    },
];

function renderTask(task) {
  const today = todayStr();
  const isDone = task.completedOn === today;
  const el = document.createElement('div');
  el.className = `task-item${isDone ? ' done' : ''}`;
  el.dataset.id = task.id;

  const checkbox = document.createElement('div');
  checkbox.className = `task-checkbox${isDone ? ' checked' : ''}`;
  checkbox.addEventListener('click', () => toggleTask(task.id));

  const title = document.createElement('span');
  title.className = 'task-title';
  title.textContent = task.title;

  const catBadge = document.createElement('span');
  catBadge.className = 'category-badge';
  catBadge.textContent = task.category;

  const delBtn = document.createElement('button');
  delBtn.className = 'delete-btn';
  delBtn.textContent = '×';
  delBtn.title = 'Delete task';
  delBtn.addEventListener('click', () => deleteTask(task.id));

  el.appendChild(checkbox);
  el.appendChild(title);
  el.appendChild(catBadge);

  if (task.type === 'recurring') {
    const recBadge = document.createElement('span');
    recBadge.className = 'recurring-badge';
    recBadge.textContent = '↺ daily';
    el.appendChild(recBadge);
  }

  el.appendChild(delBtn);
  return el;
}

function renderSections() {
  const container = document.getElementById('sections');
  container.innerHTML = '';

  if (tasks.length === 0) {
    const empty = document.createElement('div');
    empty.className = 'empty-state';
    empty.textContent = 'No tasks yet — add one above';
    container.appendChild(empty);
    return;
  }

  const today = todayStr();

  PRIORITY_CONFIG.forEach(({ key, label, cls }) => {
    const group = tasks.filter(t => t.priority === key);
    if (group.length === 0) return;

    const section = document.createElement('div');
    section.className = 'priority-section';

    const remaining = group.filter(t => t.completedOn !== today).length;

    const header = document.createElement('div');
    header.className = 'section-header';
    header.innerHTML = `
      <span class="section-label ${cls}">${label}</span>
      <span class="section-count">${remaining} remaining</span>
    `;
    section.appendChild(header);

    // Done tasks go to bottom of their section
    const sorted = [
      ...group.filter(t => t.completedOn !== today),
      ...group.filter(t => t.completedOn === today),
    ];
    sorted.forEach(task => section.appendChild(renderTask(task)));
    container.appendChild(section);
  });
}
```

- [ ] **Step 2: Update `render()` to call `renderSections()`**

Replace:
```javascript
function render() {
  renderHeader();
  // sections rendered in Task 4
}
```

With:
```javascript
function render() {
  renderHeader();
  renderSections();
}
```

- [ ] **Step 3: Verify sections render with seeded data**

In browser DevTools console:
```javascript
localStorage.setItem('daily-tasks', JSON.stringify([
  { id:'a', title:'Review PRs', priority:'high', category:'Work', type:'one-off', completedOn: null },
  { id:'b', title:'Morning standup', priority:'low', category:'Work', type:'recurring', completedOn: null },
  { id:'c', title:'Take vitamins', priority:'low', category:'Health', type:'recurring', completedOn: null }
]));
```
Reload page. Expected:
- "High Priority" section shows "Review PRs" with a Work badge
- "Low Priority" section shows two tasks each with `↺ daily` badge
- Progress shows "0 of 3 done"
- No "Medium Priority" section (no tasks at that priority)

Clean up:
```javascript
localStorage.removeItem('daily-tasks');
```
Reload.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: render priority sections and task items"
```

---

## Task 5: Add Task Action

**Files:**
- Modify: `index.html` — add Actions section, wire add button and Enter key

- [ ] **Step 1: Add the Actions section with `addTask()` and category datalist update**

Add after `render()` call, before closing `</script>`:

```javascript
// ── Actions ───────────────────────────────────────────────────────────────────

function updateCategoryDatalist() {
  const defaults = ['Work', 'Personal', 'Health'];
  const used = tasks.map(t => t.category).filter(Boolean);
  const all = [...new Set([...defaults, ...used])];
  const list = document.getElementById('category-list');
  list.innerHTML = all.map(c => `<option value="${c}"></option>`).join('');
}

function addTask() {
  const titleInput = document.getElementById('title-input');
  const title = titleInput.value.trim();
  if (!title) {
    titleInput.focus();
    return;
  }
  const category = document.getElementById('category-input').value.trim() || 'General';
  const priority = document.getElementById('priority-select').value;
  const type     = document.getElementById('type-select').value;

  const task = {
    id: generateId(),
    title,
    priority,
    category,
    type,
    completedOn: null,
  };

  tasks = [...tasks, task];
  saveTasks(tasks);
  titleInput.value = '';
  render();
  updateCategoryDatalist();
}

document.getElementById('add-btn').addEventListener('click', addTask);
document.getElementById('title-input').addEventListener('keydown', e => {
  if (e.key === 'Enter') addTask();
});

// Populate datalist on load
updateCategoryDatalist();
```

- [ ] **Step 2: Verify adding tasks in browser**

Open `index.html`. Test these cases:

1. Type a task title, pick "Work" category, "High" priority, "One-off" → click Add
   - Task appears in High Priority section
   - Input clears
   - Progress updates to "0 of 1 done"

2. Type another task, press Enter (no click needed)
   - Task is added

3. Click Add with empty title
   - Nothing happens, focus returns to input

4. Type a custom category not in defaults (e.g., "Finance")
   - After adding, type "F" in category field → "Finance" appears as suggestion

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add task action with Enter key and dynamic category datalist"
```

---

## Task 6: Toggle and Delete Actions

**Files:**
- Modify: `index.html` — add `toggleTask()` and `deleteTask()` to Actions section

- [ ] **Step 1: Add `toggleTask()` and `deleteTask()`**

Add after `updateCategoryDatalist()` definition (before the event listeners):

```javascript
function toggleTask(id) {
  const today = todayStr();
  tasks = tasks.map(task => {
    if (task.id !== id) return task;
    return {
      ...task,
      completedOn: task.completedOn === today ? null : today,
    };
  });
  saveTasks(tasks);
  render();
}

function deleteTask(id) {
  tasks = tasks.filter(task => task.id !== id);
  saveTasks(tasks);
  render();
  updateCategoryDatalist();
}
```

- [ ] **Step 2: Verify toggle and delete in browser**

Add two or three tasks. Then:

1. Click a checkbox → task gets struck-through, opacity drops, moves to bottom of its section, progress count increases
2. Click the same checkbox again → task unchecks, moves back to top, progress count decreases
3. Hover a task → delete button (×) appears on the right
4. Click × → task removed, sections update, no console errors
5. Reload page → state is persisted (checked tasks still checked, deleted tasks gone)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: toggle task completion and delete task"
```

---

## Task 7: Clear Completed and Final Polish

**Files:**
- Modify: `index.html` — wire Clear Completed button, verify full flow

- [ ] **Step 1: Add `clearCompleted()` and wire the footer button**

Add after `deleteTask()`:

```javascript
function clearCompleted() {
  const today = todayStr();
  // Remove completed one-off tasks; leave recurring (they reset tomorrow)
  tasks = tasks.filter(task => {
    if (task.type === 'one-off' && task.completedOn === today) return false;
    return true;
  });
  saveTasks(tasks);
  render();
  updateCategoryDatalist();
}

document.getElementById('clear-btn').addEventListener('click', clearCompleted);
```

- [ ] **Step 2: Verify Clear Completed behavior**

In browser:
1. Add a one-off task, check it off
2. Add a recurring task, check it off
3. Click "Clear completed"
   - The completed one-off task is removed
   - The completed recurring task stays (it'll reset tomorrow)
   - Progress bar updates

- [ ] **Step 3: Verify full daily-reset flow**

In browser DevTools console, simulate "yesterday's" completed tasks:

```javascript
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);
const yStr = yesterday.toISOString().slice(0, 10);

localStorage.setItem('daily-tasks', JSON.stringify([
  { id:'r1', title:'Morning standup', priority:'low', category:'Work',
    type:'recurring', completedOn: yStr },
  { id:'o1', title:'Fix urgent bug', priority:'high', category:'Work',
    type:'one-off', completedOn: yStr }
]));
```

Reload page. Expected:
- `r1` (recurring): unchecked — `completedOn` reset to null
- `o1` (one-off): stays checked — `completedOn` preserved as yesterday's date

Note: one-off tasks show as "done" only if `completedOn === today`. Since it's yesterday, it renders as not-done. This is correct carry-over behavior — the task remains visible and unchecked for the user to act on today.

Clean up: `localStorage.removeItem('daily-tasks')` then reload.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: clear completed one-off tasks, verified full daily reset flow"
```

---

## Task 8: End-to-End Verification

**Files:**
- No changes — verification only

- [ ] **Step 1: Full golden-path walkthrough**

Open `index.html` in browser. Walk through this sequence:

1. Page loads showing today's date and "0 of 0 done"
2. Add "Review PRs" → Work → High → One-off → click Add
3. Add "Morning standup" → Work → Low → Recurring → press Enter
4. Add "Take vitamins" → Health → Low → Recurring → press Enter
5. Add "Send weekly update" → Work → Medium → One-off → click Add
6. Verify: three priority sections visible (High, Medium, Low), "0 of 4 done"
7. Check "Morning standup" → section count drops, progress becomes "1 of 4 done"
8. Check "Review PRs" → "2 of 4 done", progress bar ~50%
9. Click "Clear completed" → only "Review PRs" removed (one-off); "Morning standup" stays checked
10. Reload page → state persists; "Morning standup" still checked
11. Hover any task → delete × appears; click it → task removed

- [ ] **Step 2: Verify empty state**

Delete all tasks. Expected: sections area shows "No tasks yet — add one above" centered text.

- [ ] **Step 3: Verify category datalist**

Type "W" in category field → "Work" suggestion appears. Type a brand-new category (e.g., "Finance"), add a task, then type "F" again → "Finance" now appears as suggestion.

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "chore: verified end-to-end — daily tasks landing page complete"
```

---

## Self-Review Checklist

- [x] **Add tasks** — Task 5
- [x] **Check off / uncheck tasks** — Task 6
- [x] **Delete individual tasks** — Task 6
- [x] **Clear completed one-off tasks (bulk)** — Task 7
- [x] **Daily auto-reset for recurring tasks** — Task 2 (logic) + Task 7 (verified)
- [x] **Progress bar and count update live** — Task 3 + Task 4
- [x] **All state persisted to localStorage** — Task 2
- [x] **Priority sections hidden when empty** — Task 4 (`renderSections` skips empty groups)
- [x] **Category datalist updates dynamically** — Task 5
- [x] **↺ daily badge on recurring tasks** — Task 4
- [x] **Delete button appears on hover** — Task 1 (CSS), Task 4 (button rendered)
- [x] **Completed tasks sorted to bottom of section** — Task 4
- [x] **Empty state when no tasks** — Task 4
