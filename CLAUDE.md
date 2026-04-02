# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build process. Open `nepal-assessment-v2.html` directly in any modern browser. No npm/bundler dependencies, but fonts load from Google Fonts CDN (Zen Antique Soft, Noto Sans JP) — internet access required for correct rendering.

## Architecture

The entire application is a single HTML file (`nepal-assessment-v2.html`, ~1,057 lines) with:
- **~700 lines of embedded CSS** — all styles, animations, responsive layout using CSS variables
- **~435 lines of embedded JavaScript** — question database, state, and all UI logic

### App Flow (3-screen SPA)

Navigation is handled by `showScreen(id)` which toggles CSS visibility.

1. **Start screen (`s-start`)** — User enters name/village/email, selects topic categories
2. **Question screen (`s-question`)** — One question at a time; answer locks screen until "Next" is clicked
3. **Results screen (`s-results`)** — Scores, category breakdown, personalized advice, optional mailto export

### State Object (`S`)

All mutable state lives in a single global `S`:
```js
{ name, village, email, selCats[], allQs[], idx, answers[], lock }
```
`lock` prevents double-answers while feedback is displayed.

### Question Database (`DB`)

10 categories (soil, seed, planting, fertilizer, pest, hilling, harvest, storage, marketing, learning), 3 questions each = 30 total. Each question:
```js
{ lv: 1|2|3, q: "text", opts: [4 strings], ans: 0-3, exp: "explanation" }
```
`lv` values: 1 = 基礎 (basic), 2 = 応用 (applied), 3 = 上級 (advanced). All text is in Japanese; the app targets Nepalese potato farmers in mid-hill regions. The layout is mobile-first.

### Scoring

Total score is 0–100. Levels:
- 90–100: Outstanding, 80–89: Excellent, 70–79: Good, 60–69: Needs Improvement, <60: Unsatisfactory

The "← もどる" back button (visible during the quiz) resets state and returns to the start screen.

### Key Functions

| Function | Purpose |
|---|---|
| `startAssessment()` | Validates inputs, shuffles questions, starts quiz |
| `renderQ()` | Renders current question from `S.allQs[S.idx]` |
| `selectAns(idx, btn)` | Records answer, shows feedback, sets `S.lock = true` |
| `nextQ()` | Advances index or calls `showResults()` |
| `showResults()` | Calculates per-category scores, renders report |
| `buildWeakAdvice()` | Generates personalized tips for weak categories |
| `sendMail()` | Builds a `mailto:` link with score summary (opens mail app; does not send automatically) |
