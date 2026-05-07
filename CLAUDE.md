# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build process. Open `index.html` directly in any modern browser. No npm/bundler dependencies, but fonts load from Google Fonts CDN (Zen Antique Soft, Noto Sans JP) — internet access required for correct rendering.

## Architecture

The entire application is a single HTML file (`index.html`, ~1,057 lines) with:
- **~700 lines of embedded CSS** — all styles, animations, responsive layout using CSS variables
- **~435 lines of embedded JavaScript** — question database, state, and all UI logic

### App Flow (3-screen SPA)

Navigation is handled by `showScreen(id)` which toggles the `.active` CSS class.

1. **Start screen (`s-start`)** — User enters name/village/email, selects topic categories
2. **Question screen (`s-question`)** — One question at a time; answer locks screen until "Next" is clicked
3. **Results screen (`s-results`)** — Scores, category breakdown, personalized advice, optional mailto export

### State Object (`S`)

All mutable state lives in a single global `S`:
```js
{ name, village, email, selCats[], allQs[], idx, answers[], lock }
```
`lock` prevents double-answers while feedback is displayed. `restartAll()` resets `S` entirely and resets all form inputs.

### Question Database (`DB`)

10 categories (soil, seed, planting, fertilizer, pest, hilling, harvest, storage, marketing, learning), 3 questions each = 30 total. Each question:
```js
{ lv: 1|2|3, q: "text", opts: [4 strings], ans: 0-3, exp: "explanation" }
```
`lv` values: 1 = 基礎 (basic), 2 = 応用 (applied), 3 = 上級 (advanced). All text is in Japanese; the app targets Nepalese potato farmers in mid-hill regions. The layout is mobile-first.

Each category in `DB` also carries `name`, `icon`, and `color` fields used for rendering.

### Scoring

Total score is 0–100 (correct answers / total × 100, rounded). Levels:
- 90–100: Outstanding, 80–89: Excellent, 70–79: Good, 60–69: Needs Improvement, <60: Unsatisfactory

The "← 戻る" back button (visible during the quiz) triggers `confirmBack()`, which shows a native `confirm()` dialog before resetting state and returning to the start screen.

### CSS Design System

Colors are defined as CSS custom properties on `:root` — soil/earth tones (`--ink`, `--soil`, `--earth`, `--gold`, `--cream`, `--paper`) plus greens (`--green-dark`, `--green`, `--leaf`) and sky/mountain palette (`--sky-top`, `--mtn1–3`). Use these variables for any new UI elements.

Bar chart animations work via a deferred `setTimeout` after `showScreen('s-results')` fires — the `.bar-fill` widths start at `0%` then transition to the `data-w` percentage value 400ms later.

### Key Functions

| Function | Purpose |
|---|---|
| `startAssessment()` | Validates inputs, shuffles questions from selected categories, starts quiz |
| `renderQ()` | Renders current question from `S.allQs[S.idx]` |
| `selectAns(idx, btn)` | Records answer, shows feedback, sets `S.lock = true` |
| `nextQ()` | Advances index or calls `showResults()` |
| `showResults()` | Calculates per-category scores, renders full report |
| `buildWeakAdvice(catSc, maxN)` | Returns up to `maxN` advice objects for the weakest categories (<70%) |
| `getColor(pct)` | Returns green/amber/red hex based on score threshold |
| `sendMail()` | Builds a `mailto:` link with score summary (opens mail app; does not send automatically) |
| `confirmBack()` | Shows native confirm dialog; calls `showScreen('s-start')` if confirmed |
| `restartAll()` | Resets `S`, clears all inputs, resets email button state, returns to start screen |
| `toast(msg)` | Shows a fixed-position toast for 3 seconds |
