# Bible Trivia App

Single-file admin-run bible trivia game. One `index.html` — inline CSS + vanilla JS, no build step, no dependencies.

## Project Structure
```
index.html                              ← entire app (CSS + JS inline)
docs/superpowers/plans/2026-04-18-bible-trivia.md  ← implementation plan
docs/superpowers/specs/2026-04-18-bible-trivia-design.md ← design spec
```

## Architecture
- All game state in a single `state` object
- `render()` re-renders `#app` innerHTML on every state change
- `bindEvents()` re-attaches all event listeners after each render
- Phases: `setup` → `buzz` → `reveal` → `end` (→ `setup` on Play Again)

## Game Flow
1. Admin enters 2 team names → Start
2. Question shown with 4 shuffled multiple-choice options (A/B/C/D)
3. Admin clicks which team buzzed first
4. Admin marks Correct (+1 point) or Wrong (0 points, question ends)
5. After 10 questions → End screen with winner + Play Again

## Key Decisions
- Answer options shuffled once at game start (`state.shuffledOptions`), not per-reveal
- No backend, no WebSockets — pure static site
- Deploy target: Cloudflare Pages at betterquizthanmichael.com
