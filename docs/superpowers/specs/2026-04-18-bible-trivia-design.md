# Bible Trivia App — Design Spec
**Date:** 2026-04-18
**Domain:** betterquizthanmichael.com
**Hosting:** Cloudflare Pages

## Overview
A single-screen, admin-run bible trivia game for in-person youth group play. Two teams compete. Admin controls all interactions on one shared display screen.

## Tech Stack
- Single `index.html` with inline CSS and vanilla JS
- No build step, no dependencies
- Deploy by uploading folder to Cloudflare Pages

## Game Flow

1. **Setup Screen** — Admin enters Team 1 and Team 2 names → clicks Start
2. **Question Screen** — Shows current question (of 10) + 4 multiple-choice options in randomized order. Two large team buttons at bottom for admin to click whichever team physically buzzed first.
3. **Answer Reveal** — After admin selects a team, shows ✓ Correct / ✗ Wrong buttons. Admin clicks the result. Scores update. Next Question button appears.
4. **End Screen** — Winner announced, final scores displayed, Play Again resets to setup.

## Scoring
- 1 point for a correct answer
- 0 points for wrong answer
- Question ends after first buzz — no second chance for other team
- Max score: 10

## Questions
- 10 hardcoded hard bible knowledge questions
- Each question has exactly 4 answer options
- Answer options are shuffled into random order at game start (once per game, not per reveal)
- Correct answer index tracked separately from display order

## UI/Layout
- Dark background, large readable text for big-screen display
- Scoreboard (both team names + scores) always visible at top
- Current question number shown (e.g. "Question 3 of 10")
- Answer options labeled A, B, C, D
- Team buzz buttons are large and visually distinct

## File Structure
```
index.html   ← entire app
```

## Deployment
- Push to Cloudflare Pages
- Custom domain: betterquizthanmichael.com
