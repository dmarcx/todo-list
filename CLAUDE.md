# CLAUDE.md

## Project Overview

A single-page TODO list web application built with vanilla HTML, CSS, and JavaScript. No build tools, no bundler, no package manager. The entire app lives in `index.html` (1,243 lines) and is deployed via Firebase Hosting.

**Live URL:** `todo-list-app-2bcda.web.app`

## Repository Structure

```
todo-list/
├── index.html        # Entire application (HTML + CSS + JS in one file)
├── firebase.json     # Firebase Hosting config (serves "." as public root)
├── .firebaserc       # Firebase project reference (todo-list-app-2bcda)
├── logo.png          # MarcAI branding logo
├── background.png    # Page background image
└── CLAUDE.md         # This file
```

There is no `package.json`, no `node_modules`, no build step, no test framework.

## Technology Stack

- **Frontend:** Vanilla HTML5, CSS3, JavaScript (ES6)
- **Auth:** Firebase Authentication (Google Sign-In)
- **Database:** Cloud Firestore (per-user document at `users/{uid}`)
- **Storage:** localStorage as offline fallback
- **Hosting:** Firebase Hosting
- **External deps:** Firebase SDK v10.12.0 loaded from CDN (`gstatic.com`)

## Architecture (index.html)

The file is organized in three sections:

| Section | Lines | Description |
|---------|-------|-------------|
| CSS | 7-763 | All styles including dark mode, responsive design, glassmorphism |
| HTML | 764-822 | DOM structure: auth bar, input row, filters, todo list, status bar |
| JavaScript | 829-1241 | App logic: state, auth, Firestore sync, CRUD, render |

### Application State

```javascript
todos[]         // Array of todo objects (source of truth)
currentUser     // Firebase auth user or null
currentFilter   // 'active' | 'completed'
editingId       // ID of todo being edited, or null
currentSort     // 'name' | 'date' | 'priority' | null
sortAsc         // boolean, sort direction
```

### Todo Object Schema

```javascript
{
  id: number,            // Date.now() timestamp
  text: string,          // Task title (required)
  completed: boolean,    // Completion status
  description: string|null,
  dueDate: string|null,  // 'YYYY-MM-DD' format
  priority: 'none'|'low'|'medium'|'high',
  tags: string[]         // Parsed from comma-separated input
}
```

### Key Functions

| Function | Purpose |
|----------|---------|
| `handleAuth()` | Google Sign-In (popup with redirect fallback) |
| `loadFromCloud()` | Pull todos from Firestore |
| `pushToCloud()` | Save todos to Firestore |
| `save()` | Write to localStorage + push to cloud if signed in |
| `addTodo()` | Create a new todo from input fields |
| `toggleTodo(id)` | Toggle completed status |
| `deleteTodo(id)` | Remove a todo |
| `startEdit(id)` / `saveEdit(id)` / `cancelEdit()` | Inline editing |
| `setFilter(filter, btn)` | Switch between active/completed view |
| `setSort(sortBy)` | Sort by name, date, or priority (toggles direction) |
| `render()` | Re-render the entire todo list DOM |
| `exportTodos()` / `importTodos(event)` | JSON backup/restore |
| `toggleDarkMode()` | Toggle dark theme (persisted in localStorage) |

### Data Flow

1. User actions call CRUD functions (`addTodo`, `toggleTodo`, etc.)
2. CRUD functions mutate the `todos` array, then call `save()` + `render()`
3. `save()` writes to `localStorage` and calls `pushToCloud()` if authenticated
4. `render()` reads `todos`, applies filter/sort, and rebuilds `todoList.innerHTML`
5. Cloud sync: loads on auth, reloads on visibility change and every 30 seconds

## Development Workflow

### Running Locally

Open `index.html` directly in a browser. No server required for basic functionality. For Firebase auth to work, use Firebase's local emulator or serve from an allowed domain.

### Deploying

```bash
firebase deploy
```

Deploys the current directory to Firebase Hosting. The `firebase.json` config serves `.` as the public directory, ignoring `.git`, `.claude`, `.firebase`, `firebase-config.txt`, and `node_modules`.

### Testing

There are no automated tests. Manual testing in the browser is the only verification method. When making changes, verify:

- Adding, editing, completing, and deleting todos
- Filter switching (active/completed views)
- Sorting by name, date, and priority (ascending/descending)
- Dark mode toggle and persistence
- Export/import JSON functionality
- Cloud sync (sign in, verify data loads, modify and verify save)
- Mobile responsiveness

## Conventions and Guidelines

### Code Style

- All code is in a single `index.html` file (CSS in `<style>`, JS in `<script>`)
- No modules, no imports, no classes -- plain functions and global state
- DOM manipulation via `innerHTML` rebuilds in `render()`
- HTML escaping via `escapeHtml()` (creates a text node) and `escapeAttr()` (regex replace)
- Event handlers are inline (`onclick`, `onchange`, `onkeydown` attributes)

### CSS Conventions

- System font stack: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`
- Primary color: indigo (`#4f46e5`)
- Dark mode via `body.dark` class with CSS overrides
- Glassmorphism: `backdrop-filter: blur(10px)` on container
- Mobile-first responsive with `max-width: 520px` container

### When Making Changes

- Keep everything in `index.html`. Do not split into separate files.
- Maintain the existing pattern: mutate `todos` array, call `save()`, call `render()`.
- New features should follow the same global function + inline event handler pattern.
- Always escape user-provided text with `escapeHtml()` or `escapeAttr()` before inserting into the DOM.
- Test dark mode for any UI changes (check both `body` and `body.dark` styles).
- The Firestore document structure is a single `todos` array per user. Avoid changing this schema without migrating existing data.
- Firebase config (API keys, project ID) is embedded in the HTML. These are client-side keys and are safe to commit per Firebase's security model.

### Security Notes

- XSS prevention relies on `escapeHtml()` and `escapeAttr()` -- use them for all user input rendered in HTML.
- Firestore security rules (not in this repo) control data access server-side.
- Firebase API keys in `index.html` are public by design (restricted by Firebase security rules and domain allowlisting).
