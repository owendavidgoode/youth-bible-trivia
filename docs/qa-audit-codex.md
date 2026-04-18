# QA Audit - Youth Bible Trivia

Audit target: `index.html`  
Audit date: 2026-04-18  
Verdict: `fix-then-ship`

Two high-risk flow issues can leave the live game inconsistent with the intended admin-run buzz/reveal mechanics or stuck with no remaining questions.

## Section 1 - Game Logic Correctness

Section 1 - 3 issues found.

### [HIGH] Required buzz/reveal flow is missing

Location: `index.html:1495` / `pickCategory`, `index.html:1607` / `render`, `index.html:1641` / `bindEvents`

Code: `state.phase = 'question';` / `case 'question': renderQuestion(); break;` / `if (state.phase === 'question') handleAnswer(teamIndex, optIndex);`

Issue: The app no longer follows the documented `buzz -> reveal -> second-chance` flow. Admins now click a team's A/B/C/D answer directly, and a correct answer immediately scores and returns to the board without a reveal/confirmation phase.

Fix: Restore explicit `buzz` and `reveal` phases: category pick should enter `buzz`, team buzz should enter `reveal`, and scoring should happen only after admin clicks Correct/Wrong.

### [HIGH] All categories can exhaust before a team reaches 10

Location: `index.html:1507` / `getRandomQuestion`, `index.html:1631` / `bindEvents`

Code: `if (available.length === 0) return null;` / `document.querySelectorAll('.category-card:not(.exhausted)')`

Issue: If enough questions are missed, all 42 questions in a difficulty can be used before either team reaches `WIN_SCORE`; the board then has no clickable category and no terminal/reset path.

Fix: Detect all-categories-exhausted in `renderCategoryBoard()` and render a "No questions left" state with Play Again or a tie-breaker action.

### [LOW] Play Again does not reset game state immediately

Location: `index.html:1646` / Play Again handler

Code: `state.phase = 'setup'; render();`

Issue: Scores, used questions, `buzzedTeam`, `wrongAnswerOption`, `currentQuestion`, and `wrongFlavorIndex` remain stale until Start Game is clicked again.

Fix: Use a `resetGameState()` helper from both Play Again and Start Game, resetting gameplay fields before rendering setup.

## Section 2 - Security & Input Safety

Section 2 - 1 issue found.

### [LOW] `escapeHtml()` misses apostrophes

Location: `index.html:1129` / `escapeHtml`

Code: `return str.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');`

Issue: Team names are consistently escaped at current render sites, but the helper does not meet the required escaping set because `'` is not encoded.

Fix:

```js
return String(str)
  .replace(/&/g, '&amp;')
  .replace(/</g, '&lt;')
  .replace(/>/g, '&gt;')
  .replace(/"/g, '&quot;')
  .replace(/'/g, '&#39;');
```

## Section 3 - Audio System

Section 3 - 3 issues found.

### [MEDIUM] Unsupported Web Audio can crash answer clicks

Location: `index.html:1180` / `getAudioCtx`

Code: `if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();`

Issue: If neither constructor exists, or construction is blocked, every SFX-triggering click can throw and interrupt gameplay.

Fix: Wrap AudioContext creation in `try/catch`; return `null` when unavailable, and make `playTone()` no-op when `ctx` is missing.

### [LOW] Oscillator/gain nodes are never disconnected

Location: `index.html:1189` / `playTone`

Code: `osc.connect(gain); gain.connect(ctx.destination); ... osc.stop(ctx.currentTime + startDelay + duration + 0.05);`

Issue: Stopped nodes are not explicitly disconnected, which can retain audio graph references during long sessions.

Fix: Add `osc.onended = () => { osc.disconnect(); gain.disconnect(); };` before `osc.start(...)`.

### [LOW] Mute/unmute ignores AudioContext promise failures

Location: `index.html:1227` / `toggleMute`

Code: `state.muted ? audioCtx.suspend() : audioCtx.resume();`

Issue: `suspend()`/`resume()` return promises and can reject, especially if the context is closed.

Fix: Guard `audioCtx.state !== 'closed'` and call `(state.muted ? audioCtx.suspend() : audioCtx.resume()).catch(() => {});`.

## Section 4 - Rendering & State Management

Section 4 - 1 issue found.

### [LOW] `render()` has no default fallback

Location: `index.html:1602` / `render`

Code: `switch (state.phase) { ... case 'win': renderWin(); break; }`

Issue: Any invalid phase leaves the previous DOM in place with no error or recovery path.

Fix: Add `default: state.phase = 'setup'; renderSetup();` or throw a clear error in development.

## Section 5 - CSS Correctness

Section 5 - 1 issue found.

### [LOW] Win/confetti stacking is not robust

Location: `index.html:106`, `index.html:687`, `index.html:933`

Code: `#mute-btn { z-index: 1000; }` / `.win-screen { z-index: 200; }` / `.confetti-piece { z-index: 201; }`

Issue: CSS says the mute button stacks above the win screen if present, and confetti stacks above win content.

Fix: Set `.win-screen { z-index: 1100; }`, put `.confetti-piece { z-index: 0; }`, and give win content `position: relative; z-index: 1;`.

## Section 6 - Question Bank Integrity

Section 6 - 2 issues found.

Mechanical validation passed:

- Total questions: 126
- Answer/option exact-match mismatches: 0
- Option count issues: 0
- Duplicate question text within category/difficulty tier: 0

### [MEDIUM] John 19:20 answer omits Greek

Location: `index.html:1051` / question bank

Code: `answer: "Hebrew, Aramaic, Latin"`

Issue: John 19:20 names Hebrew/Aramaic, Latin, and Greek; the current answer omits Greek. Source: <https://www.biblegateway.com/verse/en/John_19%3A20>

Fix: Change the option and answer to `Hebrew/Aramaic, Latin, Greek`.

### [MEDIUM] Deutero-Pauline answer is disputed/ambiguous

Location: `index.html:998` / question bank

Code: `answer: "1 Timothy, 2 Timothy, Titus (Pastorals)"`

Issue: The Pastorals are commonly treated as their own group; sources also identify Ephesians, Colossians, and 2 Thessalonians as deutero-Pauline/disputed. Source: <https://www.britannica.com/topic/biblical-literature/The-Pauline-Letters>

Fix: Either change the answer to `Ephesians, Colossians, 2 Thessalonians`, or reword the question to ask specifically for the Pastoral Epistles.

## Section 7 - Performance & Reliability

Section 7 - 1 issue found.

### [LOW] `sessionStorage` can hard-crash initial render

Location: `index.html:1665` / kickoff, `index.html:1654` / splash handler

Code: `state.phase = sessionStorage.getItem('gameReady') ? 'setup' : 'splash';`

Issue: Browser privacy settings or embedded contexts can throw `SecurityError` on `sessionStorage`, preventing any render.

Fix: Wrap storage access in helpers: `safeGetGameReady()` and `safeSetGameReady()` with `try/catch` fallback.

## Section 8 - Code Quality

Section 8 - 2 issues found.

### [NITPICK] Stale buzz/reveal CSS remains after flow change

Location: `index.html:39`, `index.html:550`, `index.html:603`, `index.html:631`

Code: `@keyframes shake` / `.buzz-btn` / `.reveal-area` / `.verdict-buttons`

Issue: These styles are no longer referenced by rendered markup in the current answer-button flow.

Fix: Remove the dead CSS if the answer-button flow is intentional, or restore the missing buzz/reveal markup if the original flow is required.

### [NITPICK] Confetti count is a magic number

Location: `index.html:1438` / `renderWin`

Code: `Array.from({ length: 50 }, (_, i) => {`

Issue: `50` is behaviorally meaningful but unnamed.

Fix: Add `const CONFETTI_COUNT = 50;` near `WIN_SCORE` and use `Array.from({ length: CONFETTI_COUNT }, ...)`.

## Summary Table

| Severity | Count |
|----------|-------|
| critical | 0 |
| high | 2 |
| medium | 3 |
| low | 7 |
| nitpick | 2 |

## Top 3 Critical Fixes

No critical issues found.

## Overall Verdict

`fix-then-ship`

Two high-risk flow issues can leave the live game inconsistent with the intended admin-run buzz/reveal mechanics or stuck with no remaining questions.
