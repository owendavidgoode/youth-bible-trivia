# Bible Trivia App v2 — Design Spec
**Date:** 2026-04-18
**Domain:** betterquizthanmichael.com
**Hosting:** Cloudflare Pages (Bridgepoint Bible Church account)

## Overview
A visually impressive, single-screen admin-run bible trivia game for Bridgepoint Bible Church youth group. Non-technical audience — CSS visuals should wow. Two teams compete to 10 points. Admin controls everything on one shared big-screen display.

## Tech Stack
Single `index.html`, inline CSS + vanilla JS. No build step, no dependencies. Deployed via Wrangler to Cloudflare Pages.

---

## Game Flow

### Phase: setup
- Admin selects difficulty: **Hard** / **Really Hard** / **Bob Stanberry**
- Admin enters Team 1 and Team 2 names (defaults if blank)
- "Start Game" button

### Phase: category-board
- 6 category cards displayed in a 2×3 grid
- Header shows: scores, whose turn it is to pick ("Team A's pick")
- Teams alternate turns each round
- Admin clicks a category card → loads a random unused question from that category+difficulty
- If a category is exhausted at current difficulty, it's visually greyed out
- Battlefield bar always visible

### Phase: buzz
- Question text + 4 shuffled options (A/B/C/D) displayed
- Scoreboard + battlefield bar visible
- Two large team buzz buttons at bottom
- Admin clicks whichever team physically buzzed first

### Phase: reveal
- Correct answer highlighted green
- Buzzed team name displayed
- Admin clicks **Correct** (+1 point) or **Wrong** (trigger second-chance)
- If Correct: check for win (10 pts) → if won go to `win` phase, else back to `category-board` (other team's turn)

### Phase: second-chance
- Same question still shown, correct answer still highlighted
- Banner: "[Other team] gets a chance!"
- Only the other team's buzz button is active (first team's button is greyed/disabled)
- Admin clicks correct/wrong
- Correct: +1 for other team, check win, back to category-board (other team's turn — they benefitted)
- Wrong: 0 points, back to category-board (original picking team's turn continues — they don't lose turn for other team's failure)

### Phase: win
- Full-screen celebration
- Winner name large, in team color
- CSS confetti animation
- Final scores displayed
- Play Again button → back to setup

---

## Turn Logic
- `pickingTeam` flips after every fully resolved question (both teams answered or question ended)
- "Resolved" means: a correct answer was given OR both teams answered incorrectly
- Simple, fair, no exceptions

---

## Scoring
- First team to reach 10 points wins
- 1 point per correct answer
- 0 points for wrong answer

---

## Question Bank Structure
```javascript
QUESTION_BANK[category][difficulty] = [ { q, options, answer }, ... ]
```

### Categories (6)
1. **Old Testament** — Genesis through Malachi events, people, places
2. **New Testament** — Acts through Revelation (non-gospel)
3. **Kings & Prophets** — Saul, David, Solomon, Isaiah, Elijah, etc.
4. **Jesus & Gospels** — Matthew, Mark, Luke, John specific events
5. **Books of the Bible** — Authorship, order, content, chapter counts
6. **Numbers & Facts** — How many, how long, specific quantities

### Difficulties
- **Hard** — a serious Bible student knows it
- **Really Hard** — seminary-level trivia
- **Bob Stanberry** — only for those who read the Septuagint for fun

### Bank Size
~6–8 questions per category per difficulty = ~108–144 questions total
Questions drawn randomly per game, no repeats within a session

---

## Visual Design

### Theme
Dark, dramatic, vivid. Gradient backgrounds. Glowing accents. Impressive to a non-technical audience.

### Color Palette
```css
--bg-from: #0a0a1a;
--bg-to: #1a0a2e;
--surface: rgba(255,255,255,0.05);
--surface-border: rgba(255,255,255,0.1);
--team1: #3b82f6;        /* blue */
--team2: #ef4444;        /* red */
--correct: #22c55e;
--wrong: #ef4444;
--gold: #f59e0b;
--text: #f0f0f0;
--muted: #8888aa;
```

Background: animated radial gradient (slow pulse between dark navy and deep purple).

### Typography
- Title: large serif, letter-spacing, subtle text-shadow glow
- Scores: huge (4–5rem), bold, team-colored with drop-shadow glow
- Question text: 2.2rem, centered, max-width for readability
- Options: 1.2rem on frosted-glass cards

### Scoreboard
- Fixed at top
- Team names + scores in team colors with glow
- Battlefield bar below scores (always visible during game)

### Battlefield Bar
- Full-width horizontal bar
- Left half = team1 color, right half = team2 color
- Divider position = team1_score / (team1_score + team2_score) or 50% if tied
- CSS transition: smooth slide on score change
- Animated shimmer/pulse when a point is scored
- Shows score ratio visually

### Category Board
- 6 cards in 2×3 grid with emoji icons per category
- Frosted glass cards with border glow on hover
- Hover: card lifts (translateY), border brightens
- Used/exhausted categories: greyed out, "No more questions" overlay
- Animated entrance (fade+slide up)
- Category icons:
  - Old Testament: 📜
  - New Testament: ✉️
  - Kings & Prophets: 👑
  - Jesus & Gospels: ✝️
  - Books of the Bible: 📖
  - Numbers & Facts: 🔢

### Question Screen
- Question text in a large frosted-glass card, centered
- Options in 2×2 grid, frosted glass buttons
- Hover: subtle lift + border glow
- On reveal: correct answer glows green with animated border
- Wrong team's selection: red shake animation

### Buzz Buttons
- Very large (height: 120px+)
- Vivid team colors with gradient
- On hover: glow pulse
- On click: flash + scale animation
- On wrong answer: shake animation

### Win Screen
- Full dark overlay
- Winner name in large glowing text
- CSS confetti: 50+ divs falling with keyframe animation (different colors, sizes, rotation speeds)
- "Even Michael couldn't argue with that score" tagline
- Play Again button with pulsing glow

### Animations Summary
- Background: slow animated gradient (`@keyframes bgPulse`)
- Score increment: number bounces + brief glow (`@keyframes scorePop`)
- Battlefield shift: CSS transition on width (0.6s ease)
- Buzz click: scale up flash (`@keyframes buzzFlash`)
- Wrong answer: horizontal shake (`@keyframes shake`)
- Category card hover: translateY(-4px) + box-shadow intensify
- Win confetti: `@keyframes confettiFall` with random delays/positions
- Phase transitions: fade in (`@keyframes fadeIn`)
- Correct answer: green border pulse (`@keyframes correctPulse`)

---

## Branding

### Bridgepoint Bible Church
- Logo fetched from: search for Bridgepoint Bible Church logo URL at build time, embed as img tag
- Fallback: text "Bridgepoint Bible Church" if logo unavailable
- Position: top-left corner of header, small (40px height)
- Subtle, doesn't distract from gameplay

### Snarky Michael Content
- Setup tagline below title: *"The quiz that's definitively better than Michael's"*
- Title hover tooltip: *"Yes, it IS better than Michael's. We checked."*
- Bob Stanberry difficulty tooltip: *"Only for those who read the Septuagint for fun. Bob would finish this before you finish reading the question."*
- Win screen subtext: *"Even Michael couldn't argue with that score."*
- If score is 10–0: *"Michael is crying somewhere right now."*
- Wrong answer flavor text (rotating): 
  - *"Michael would have known that one."*
  - *"Ooh, tough one. Michael definitely got this in practice."*
  - *"Not your best moment. Michael is watching."*

---

## State Object
```javascript
const state = {
  phase: 'setup',           // setup | category-board | buzz | reveal | second-chance | win
  difficulty: null,         // 'hard' | 'really-hard' | 'bob-stanberry'
  team1: '',
  team2: '',
  scores: [0, 0],
  pickingTeam: 0,           // 0 or 1 — whose turn to pick category
  currentQuestion: null,    // { q, options, answer, shuffledOptions }
  currentCategory: null,
  buzzedTeam: null,         // 0 or 1 — who buzzed first
  usedQuestions: {},        // { [category]: Set of used indices }
  wrongFlavorIndex: 0,      // cycles through snarky wrong messages
};
```

---

## File Structure
```
index.html     ← entire app
CLAUDE.md      ← project context
docs/          ← specs and plans
```

---

## Deployment
- Wrangler CLI: `wrangler pages deploy . --project-name betterquizthanmichael`
- Account: Bridgepoint Bible Church (a81e254e5458a0b503e55c9371d4cf14)
- GitHub: owendavidgoode/youth-bible-trivia (push after each deploy)
