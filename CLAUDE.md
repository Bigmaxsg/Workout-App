# CLAUDE.md — Workout-App AI Assistant Guide

## Project Overview

**Workout Tracker Pro** is a production-ready fitness tracking Progressive Web App (PWA) built as a single-file HTML5 application. It requires no build step, no backend, and no external dependencies beyond CDN-loaded libraries. All state is persisted in the browser's localStorage.

---

## Repository Structure

```
Workout-App/
├── index.html          # Entire application (HTML + CSS + JS, ~9000 lines)
├── manifest.json       # PWA manifest (app name, theme, icons)
├── site.webmanifest    # Alternative PWA manifest
├── favicon.ico / .svg  # Favicon assets
├── apple-touch-icon.png
├── android-chrome-*.png
├── web-app-manifest-*.png
├── images/             # Exercise demonstration GIFs (89 files)
└── CLAUDE.md           # This file
```

The entire application lives in **`index.html`**. There are no separate JS/CSS files.

---

## Technology Stack

| Layer | Technology |
|---|---|
| Language | HTML5, CSS3, Vanilla JavaScript (ES6+) |
| Architecture | Single-file SPA with client-side section switching |
| Data Persistence | Browser `localStorage` |
| AI Coaching | Google Gemini API (`gemini-2.5-flash`) |
| QR Codes | QRCode.js v1.0.0 + jsQR v1.4.0 (CDN) |
| Compression | Pako v2.1.0 (CDN) |
| Build System | None — open `index.html` directly |
| PWA | Standalone installable (iOS + Android) |

---

## index.html Internal Structure

The file is organized into three top-level sections:

### 1. `<head>` — Metadata and PWA configuration
### 2. `<style>` (lines ~36–1222) — All CSS
- BEM-inspired class naming: `.btn`, `.btn-secondary`, `.exercise-card`
- Color palette: Navy `#001f3f`, Gold `#d4af37`, Tan `#d9c7a5` / `#e8dcc4`
- Responsive layout via flexbox and CSS grid
- CSS variables for user-customizable theme colors

### 3. `<body>` — HTML sections + embedded `<script>`

#### Application Sections (navigated via `showSection(id)`)

| Section ID | Purpose |
|---|---|
| `ai-coach` | Chat interface for Gemini AI fitness coaching |
| `workout` | Active workout session (timer, sets, reps, weight) |
| `exercises` | Browsable exercise database |
| `recommend` | AI-generated workout recommendations |
| `progress` | PRs, streaks, and workout stats |
| `history` | Past workout sessions log |
| `calendar` | Month/week/day calendar of workouts |
| `body-stats` | Body measurements tracker |
| `share-workout` | QR code export/import for sharing workouts |
| `settings` | Data export/import, theme customization |

#### JavaScript Structure (inside `<script>`, starts ~line 2100)

- **Global state variables** declared at the top
- **Embedded exercise database** at ~line 2053 (100+ exercises)
- Code organized by feature domain with section headers:
  ```js
  // ==================== SECTION NAME ====================
  ```

---

## Key JavaScript Conventions

### Naming
- Functions: `camelCase` verb+noun format — `startTimer()`, `updateExerciseValue()`, `deleteCustomWorkout()`
- DOM IDs: descriptive, prefixed by feature — `#timerDisplay`, `#workoutContainer`, `#apiKey`
- CSS classes: kebab-case — `.exercise-card`, `.drag-over`, `.detail-input`
- localStorage keys: `camelCase` — `workoutHistory`, `personalRecords`, `geminiKey`

### State Management
- All mutable state is global JavaScript variables
- Persisted to `localStorage` after each relevant mutation
- Read with fallback defaults:
  ```js
  JSON.parse(localStorage.getItem('key') || '{}')
  JSON.parse(localStorage.getItem('key') || '[]')
  ```

### DOM Interaction
- Direct `document.getElementById()` queries (no virtual DOM)
- Event handlers via inline `onclick` attributes in HTML
- Content updated by setting `.innerHTML` or `.textContent`

### Async / API
- `async/await` for Gemini API calls
- `try/catch` for all API error handling
- API key stored in `localStorage` as `geminiKey`, validated before every call

### Audio
- Web Audio API for timer beeps
- `AudioContext` initialized with vendor prefix fallback:
  ```js
  window.AudioContext || window.webkitAudioContext
  ```
- Context state checked and resumed (handles iOS suspension)
- Audio pre-triggered at the 4-second countdown mark

---

## localStorage Data Schema

| Key | Type | Purpose |
|---|---|---|
| `workoutHistory` | `Array` | All completed workout sessions |
| `customWorkouts` | `Object` | User-created workout templates |
| `customExercises` | `Object` | User-defined exercises |
| `exerciseDescriptions` | `Object` | AI-cached exercise descriptions |
| `personalRecords` | `Object` | Max weight per exercise |
| `workoutPreferences` | `Object` | Rest times, UI preferences |
| `workoutRestParams` | `Object` | Rest timing settings |
| `lastUsedExerciseValues` | `Object` | Cached last weight/reps per exercise |
| `geminiKey` | `String` | User's Gemini API key |
| `bodyStats` | `Array` | Weight, height, body fat over time |
| `colorPreferences` | `Object` | Custom theme color overrides |

---

## Workout Modes

| Mode | Description |
|---|---|
| **Traditional** | Sets × Reps with configurable rest between sets/exercises |
| **HIIT** | Work/Rest intervals with multiple rounds |
| **Circuit** | Exercises in sequence with rest after each circuit |
| **Cardio** | Timed cardio with interval support |
| **Custom** | User-defined workout templates |

---

## External API

**Google Gemini API**
- Endpoint: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key={API_KEY}`
- Method: `POST`
- Used for: workout generation, exercise recommendations, fitness coaching chat
- API key obtained from: https://ai.google.dev

---

## Running the App

```bash
# Option 1: Open directly
open index.html

# Option 2: Local server (avoids some browser restrictions)
python -m http.server 8000
# Then navigate to http://localhost:8000
```

No npm install, no build, no configuration required.

---

## Development Workflow

### Making Changes
All code lives in `index.html`. Edit it directly. No transpilation or compilation.

### Git Branch Convention
- Feature/fix branches follow the pattern: `claude/<description>-<id>`
- Main branch is `master` (also mirrored as `main` on remote)
- Commit messages describe the feature or fix concisely

### Testing
There is no automated test suite. Verification is manual:
1. Open `index.html` in a browser
2. Exercise the affected feature
3. Check `localStorage` via browser DevTools if debugging data persistence

---

## Key Functions Reference

| Function | Purpose |
|---|---|
| `showSection(id)` | Navigate between app sections |
| `startTimer()` / `pauseTimer()` / `resetTimer()` | Workout timer controls |
| `startGuidedWorkout(mode)` | Begin a workout session |
| `startHiitWorkout()` | Begin HIIT-specific session |
| `completeSet()` | Mark current set as done |
| `endWorkout()` | Save session to `workoutHistory` |
| `exportData()` | Export all data to JSON |
| `importData(event)` | Import data from JSON file |
| `sendMessage()` | Send message to Gemini AI coach |
| `saveApiKey()` | Persist Gemini API key to localStorage |
| `applyTheme(name)` | Apply a built-in color theme |
| `updateColors()` | Save custom color preferences |

---

## AI Assistant Guidelines

### What to Do
- Edit `index.html` only — this is the single source of truth
- Preserve the existing CSS variable system for colors; extend it if adding new theme-able elements
- Use `localStorage` for any new persistent state, following the existing read-with-fallback pattern
- Follow existing camelCase/verb+noun naming for new functions
- Add new section-header comments (`// ==================== SECTION ====================`) when introducing large new feature blocks
- Test audio-related code on mobile; always resume `AudioContext` before playing

### What to Avoid
- Do not introduce a build system, bundler, or npm dependencies unless explicitly requested
- Do not create new files for JS/CSS — keep everything in `index.html`
- Do not use frameworks (React, Vue, etc.) — the app is intentionally vanilla
- Do not store sensitive data beyond the user's own API key (which is their choice)
- Do not add a backend or service worker without explicit request
- Do not use `innerHTML` with unsanitized user input (XSS risk)

### Adding Exercises
The exercise database is embedded in `index.html` at ~line 2053. Each entry includes: equipment type, default sets/reps/weight, form instructions, image reference, and target muscles. Match this schema when adding new exercises.

### Adding New Sections
1. Add a `<div id="new-section-name">` block in the HTML body
2. Add a navigation entry if needed
3. Register in `showSection()` if it needs special initialization logic
4. Follow the existing CSS class conventions for styling
