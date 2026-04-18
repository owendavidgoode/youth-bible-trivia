# Bible Trivia App — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` admin-run team bible trivia game with 10 hardcoded hard questions, deployed to Cloudflare Pages at betterquizthanmichael.com.

**Architecture:** Single HTML file with inline CSS and vanilla JS. Game state lives in a JS object; UI is fully re-rendered via innerHTML on every state change. No build step, no dependencies.

**Tech Stack:** Vanilla HTML/CSS/JS, Cloudflare Pages.

---

### Task 1: HTML shell + dark theme CSS

**Files:**
- Create: `index.html`

- [ ] Create `index.html` with full structure and CSS:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Bible Trivia</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg: #0d0d1a;
      --surface: #1a1a2e;
      --text: #f0f0f0;
      --muted: #8888aa;
      --team1: #3b82f6;
      --team2: #ef4444;
      --correct: #22c55e;
      --wrong: #ef4444;
      --option: #252540;
    }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: 'Georgia', serif;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
    }

    #app { flex: 1; display: flex; flex-direction: column; min-height: 100vh; }

    /* Scoreboard */
    .scoreboard {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 16px 40px;
      background: var(--surface);
      font-size: 1.5rem;
      font-weight: bold;
    }
    .score-team1 { color: var(--team1); text-align: left; }
    .score-team2 { color: var(--team2); text-align: right; }
    .score-label { font-size: 0.85rem; color: var(--muted); font-weight: normal; }
    .scoreboard-center { color: var(--muted); font-size: 0.95rem; font-weight: normal; }

    /* Setup */
    .setup-screen {
      flex: 1; display: flex; flex-direction: column;
      align-items: center; justify-content: center;
      gap: 28px; padding: 40px;
    }
    .setup-screen h1 { font-size: 3rem; text-align: center; }
    .setup-screen p { color: var(--muted); font-size: 1.1rem; }
    .team-inputs { display: flex; gap: 28px; flex-wrap: wrap; justify-content: center; }
    .team-input-group { display: flex; flex-direction: column; gap: 8px; }
    .team-input-group label { font-size: 0.95rem; color: var(--muted); }
    .team-input-group input {
      background: var(--surface);
      border: 2px solid var(--option);
      color: var(--text);
      padding: 12px 20px;
      font-size: 1.3rem;
      border-radius: 8px;
      outline: none;
      width: 230px;
      font-family: inherit;
    }
    .team-input-group input:focus { border-color: var(--team1); }

    /* Shared button */
    .btn-primary {
      background: var(--team1); color: white; border: none;
      padding: 16px 52px; font-size: 1.4rem; border-radius: 10px;
      cursor: pointer; font-family: inherit; transition: opacity 0.15s;
    }
    .btn-primary:hover { opacity: 0.85; }

    /* Question screen */
    .question-screen {
      flex: 1; display: flex; flex-direction: column; padding: 32px; gap: 24px;
    }
    .question-text {
      font-size: 2rem; text-align: center; line-height: 1.45;
      flex: 1; display: flex; align-items: center; justify-content: center;
      padding: 0 32px;
    }
    .options-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
    .option-btn {
      background: var(--option); color: var(--text);
      border: 2px solid transparent;
      padding: 20px 24px; font-size: 1.15rem;
      border-radius: 10px; text-align: left; font-family: inherit;
    }
    .option-label { font-weight: bold; margin-right: 12px; color: var(--muted); }
    .option-btn.correct {
      background: rgba(34,197,94,0.2);
      border-color: var(--correct);
      color: var(--correct);
    }

    /* Buzz buttons */
    .buzz-area { display: flex; gap: 20px; margin-top: 8px; }
    .buzz-btn {
      flex: 1; padding: 26px; font-size: 1.5rem; font-weight: bold;
      border: none; border-radius: 12px; cursor: pointer;
      font-family: inherit; transition: transform 0.1s, opacity 0.15s;
    }
    .buzz-btn:active { transform: scale(0.97); }
    .buzz-btn:hover { opacity: 0.85; }
    .buzz-btn.team1 { background: var(--team1); color: white; }
    .buzz-btn.team2 { background: var(--team2); color: white; }

    /* Reveal */
    .reveal-area {
      display: flex; flex-direction: column;
      align-items: center; gap: 14px; margin-top: 8px;
    }
    .buzzed-label { font-size: 1.1rem; color: var(--muted); }
    .buzzed-name { font-weight: bold; color: var(--text); font-size: 1.35rem; }
    .result-buttons { display: flex; gap: 20px; }
    .btn-correct {
      background: var(--correct); color: white; border: none;
      padding: 18px 44px; font-size: 1.3rem; border-radius: 10px;
      cursor: pointer; font-family: inherit;
    }
    .btn-wrong {
      background: var(--wrong); color: white; border: none;
      padding: 18px 44px; font-size: 1.3rem; border-radius: 10px;
      cursor: pointer; font-family: inherit;
    }

    /* End */
    .end-screen {
      flex: 1; display: flex; flex-direction: column;
      align-items: center; justify-content: center;
      gap: 32px; padding: 40px; text-align: center;
    }
    .end-label { font-size: 1.1rem; color: var(--muted); letter-spacing: 2px; text-transform: uppercase; }
    .winner-name { font-size: 3.5rem; font-weight: bold; }
    .final-scores { display: flex; gap: 56px; font-size: 1.4rem; }
    .final-score-item { display: flex; flex-direction: column; align-items: center; gap: 4px; }
    .final-score-number { font-size: 2.5rem; font-weight: bold; }
  </style>
</head>
<body>
  <div id="app"></div>
  <script>
    // JS goes here in subsequent tasks
    document.getElementById('app').innerHTML = '<div class="setup-screen"><h1>Bible Trivia</h1></div>';
  </script>
</body>
</html>
```

- [ ] Open `index.html` in browser — dark background, "Bible Trivia" heading visible, no console errors
- [ ] Commit:
```bash
git init
git add index.html
git commit -m "feat: HTML shell and dark theme CSS"
```

---

### Task 2: Questions data + shuffle utility

**Files:**
- Modify: `index.html` (replace the `<script>` block)

- [ ] Replace the entire `<script>` block content with:

```javascript
const QUESTIONS = [
  {
    q: "Which book of the Bible does NOT contain the word 'God'?",
    options: ["Ruth", "Esther", "Song of Solomon", "Philemon"],
    answer: "Esther"
  },
  {
    q: "How many fish did the disciples catch in the miraculous catch recorded in John 21?",
    options: ["72", "120", "153", "276"],
    answer: "153"
  },
  {
    q: "Who was the father of Methuselah, the oldest person in the Bible?",
    options: ["Seth", "Lamech", "Enoch", "Noah"],
    answer: "Enoch"
  },
  {
    q: "For how many pieces of silver was Joseph sold to the Ishmaelites by his brothers?",
    options: ["15", "20", "30", "50"],
    answer: "20"
  },
  {
    q: "In what valley did David kill Goliath?",
    options: ["Valley of Jezreel", "Valley of Elah", "Valley of Hinnom", "Valley of Kidron"],
    answer: "Valley of Elah"
  },
  {
    q: "How many years did it take Solomon to build the Temple in Jerusalem?",
    options: ["3", "5", "7", "12"],
    answer: "7"
  },
  {
    q: "Who was the first person to see the risen Jesus according to John's Gospel?",
    options: ["Peter", "John", "Mary the mother of Jesus", "Mary Magdalene"],
    answer: "Mary Magdalene"
  },
  {
    q: "How many days was Lazarus in the tomb before Jesus raised him?",
    options: ["1", "2", "3", "4"],
    answer: "4"
  },
  {
    q: "What was the occupation of Zacchaeus, who climbed a tree to see Jesus?",
    options: ["Fisherman", "Carpenter", "Chief tax collector", "Tent-maker"],
    answer: "Chief tax collector"
  },
  {
    q: "On which island was the Apostle John when he received the visions recorded in Revelation?",
    options: ["Cyprus", "Crete", "Patmos", "Malta"],
    answer: "Patmos"
  }
];

function shuffle(arr) {
  const a = [...arr];
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

document.getElementById('app').innerHTML = '<div class="setup-screen"><h1>Bible Trivia</h1></div>';
```

- [ ] Open browser console and run: `QUESTIONS.length` — expect `10`; `shuffle([1,2,3,4])` — expect a shuffled array
- [ ] Commit:
```bash
git add index.html
git commit -m "feat: 10 hardcoded bible questions and shuffle utility"
```

---

### Task 3: Game state + render scaffold

**Files:**
- Modify: `index.html` (JS section — append after shuffle function, replace dummy innerHTML line)

- [ ] Replace `document.getElementById('app').innerHTML = ...` with the full state + render scaffold:

```javascript
const state = {
  phase: 'setup',       // 'setup' | 'buzz' | 'reveal' | 'end'
  team1: '',
  team2: '',
  scores: [0, 0],
  questionIndex: 0,
  shuffledOptions: [],  // per-question shuffled option arrays, set on game start
  buzzedTeam: null,     // 0 or 1
};

function render() {
  const app = document.getElementById('app');
  switch (state.phase) {
    case 'setup':  app.innerHTML = renderSetup();    break;
    case 'buzz':   app.innerHTML = renderQuestion(); break;
    case 'reveal': app.innerHTML = renderReveal();   break;
    case 'end':    app.innerHTML = renderEnd();      break;
  }
  bindEvents();
}

function renderScoreboard() {
  return `
    <div class="scoreboard">
      <div class="score-team1">
        <div class="score-label">${state.team1}</div>
        ${state.scores[0]}
      </div>
      <div class="scoreboard-center">Question ${state.questionIndex + 1} of ${QUESTIONS.length}</div>
      <div class="score-team2">
        <div class="score-label">${state.team2}</div>
        ${state.scores[1]}
      </div>
    </div>
  `;
}

function renderSetup()    { return '<div class="setup-screen"><h1>Bible Trivia</h1></div>'; }
function renderQuestion() { return '<div class="question-screen"><p>Question screen</p></div>'; }
function renderReveal()   { return '<div class="question-screen"><p>Reveal screen</p></div>'; }
function renderEnd()      { return '<div class="end-screen"><p>End screen</p></div>'; }
function bindEvents()     {}

render();
```

- [ ] Open browser — "Bible Trivia" heading visible, no errors
- [ ] Commit:
```bash
git add index.html
git commit -m "feat: game state object and render scaffold"
```

---

### Task 4: Setup screen

**Files:**
- Modify: `index.html` (JS section)

- [ ] Replace `renderSetup()` and `bindEvents()` with:

```javascript
function renderSetup() {
  return `
    <div class="setup-screen">
      <h1>Bible Trivia</h1>
      <p>Enter team names to begin</p>
      <div class="team-inputs">
        <div class="team-input-group">
          <label>Team 1 Name</label>
          <input id="team1-input" type="text" placeholder="Team 1" maxlength="20" />
        </div>
        <div class="team-input-group">
          <label>Team 2 Name</label>
          <input id="team2-input" type="text" placeholder="Team 2" maxlength="20" />
        </div>
      </div>
      <button class="btn-primary" id="start-btn">Start Game</button>
    </div>
  `;
}

function startGame() {
  state.team1 = document.getElementById('team1-input').value.trim() || 'Team 1';
  state.team2 = document.getElementById('team2-input').value.trim() || 'Team 2';
  state.scores = [0, 0];
  state.questionIndex = 0;
  state.shuffledOptions = QUESTIONS.map(q => shuffle(q.options));
  state.buzzedTeam = null;
  state.phase = 'buzz';
  render();
}

function bindEvents() {
  document.getElementById('start-btn')?.addEventListener('click', startGame);
}
```

- [ ] Open browser: setup screen shows both inputs and Start button. Enter names, click Start — transitions to "Question screen" placeholder
- [ ] Commit:
```bash
git add index.html
git commit -m "feat: setup screen with team name inputs and game start"
```

---

### Task 5: Question (buzz) screen

**Files:**
- Modify: `index.html` (JS section)

- [ ] Replace `renderQuestion()` with:

```javascript
function renderQuestion() {
  const q = QUESTIONS[state.questionIndex];
  const opts = state.shuffledOptions[state.questionIndex];
  const labels = ['A', 'B', 'C', 'D'];

  const optionsHtml = opts.map((opt, i) => `
    <div class="option-btn">
      <span class="option-label">${labels[i]}</span>${opt}
    </div>
  `).join('');

  return `
    ${renderScoreboard()}
    <div class="question-screen">
      <div class="question-text">${q.q}</div>
      <div class="options-grid">${optionsHtml}</div>
      <div class="buzz-area">
        <button class="buzz-btn team1" data-team="0">${state.team1} buzzed!</button>
        <button class="buzz-btn team2" data-team="1">${state.team2} buzzed!</button>
      </div>
    </div>
  `;
}
```

- [ ] Update `bindEvents()` to handle buzz buttons:

```javascript
function bindEvents() {
  document.getElementById('start-btn')?.addEventListener('click', startGame);

  document.querySelectorAll('.buzz-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      state.buzzedTeam = parseInt(btn.dataset.team);
      state.phase = 'reveal';
      render();
    });
  });
}
```

- [ ] Open browser: start game, verify scoreboard shows team names + scores + question number. Question text and A/B/C/D options display. Click a buzz button — transitions to "Reveal screen" placeholder
- [ ] Commit:
```bash
git add index.html
git commit -m "feat: question screen with shuffled options and buzz buttons"
```

---

### Task 6: Reveal screen + scoring

**Files:**
- Modify: `index.html` (JS section)

- [ ] Replace `renderReveal()` with:

```javascript
function renderReveal() {
  const q = QUESTIONS[state.questionIndex];
  const opts = state.shuffledOptions[state.questionIndex];
  const labels = ['A', 'B', 'C', 'D'];
  const buzzedName = state.buzzedTeam === 0 ? state.team1 : state.team2;

  const optionsHtml = opts.map((opt, i) => {
    const isAnswer = opt === q.answer;
    return `
      <div class="option-btn ${isAnswer ? 'correct' : ''}">
        <span class="option-label">${labels[i]}</span>${opt}
      </div>
    `;
  }).join('');

  return `
    ${renderScoreboard()}
    <div class="question-screen">
      <div class="question-text">${q.q}</div>
      <div class="options-grid">${optionsHtml}</div>
      <div class="reveal-area">
        <div class="buzzed-label">Buzzed in: <span class="buzzed-name">${buzzedName}</span></div>
        <div class="result-buttons">
          <button class="btn-correct" id="correct-btn">Correct</button>
          <button class="btn-wrong" id="wrong-btn">Wrong</button>
        </div>
      </div>
    </div>
  `;
}

function advanceQuestion() {
  if (state.questionIndex < QUESTIONS.length - 1) {
    state.questionIndex++;
    state.phase = 'buzz';
  } else {
    state.phase = 'end';
  }
  render();
}
```

- [ ] Add correct/wrong handlers to `bindEvents()`:

```javascript
function bindEvents() {
  document.getElementById('start-btn')?.addEventListener('click', startGame);

  document.querySelectorAll('.buzz-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      state.buzzedTeam = parseInt(btn.dataset.team);
      state.phase = 'reveal';
      render();
    });
  });

  document.getElementById('correct-btn')?.addEventListener('click', () => {
    state.scores[state.buzzedTeam]++;
    advanceQuestion();
  });

  document.getElementById('wrong-btn')?.addEventListener('click', () => {
    advanceQuestion();
  });
}
```

- [ ] Open browser: buzz a team, verify correct answer highlighted green. Click Correct — score increments, advances to next question. Click Wrong — no score change, advances.
- [ ] Commit:
```bash
git add index.html
git commit -m "feat: reveal screen with answer highlight and scoring"
```

---

### Task 7: End screen + Play Again

**Files:**
- Modify: `index.html` (JS section)

- [ ] Replace `renderEnd()` with:

```javascript
function renderEnd() {
  const [s1, s2] = state.scores;
  let winnerHtml;
  if (s1 > s2) {
    winnerHtml = `<div class="winner-name" style="color:var(--team1)">${state.team1} wins!</div>`;
  } else if (s2 > s1) {
    winnerHtml = `<div class="winner-name" style="color:var(--team2)">${state.team2} wins!</div>`;
  } else {
    winnerHtml = `<div class="winner-name">It's a tie!</div>`;
  }

  return `
    <div class="end-screen">
      <div class="end-label">Game Over</div>
      ${winnerHtml}
      <div class="final-scores">
        <div class="final-score-item">
          <div style="color:var(--team1)">${state.team1}</div>
          <div class="final-score-number" style="color:var(--team1)">${s1}</div>
        </div>
        <div class="final-score-item">
          <div style="color:var(--team2)">${state.team2}</div>
          <div class="final-score-number" style="color:var(--team2)">${s2}</div>
        </div>
      </div>
      <button class="btn-primary" id="play-again-btn">Play Again</button>
    </div>
  `;
}
```

- [ ] Add play-again handler to `bindEvents()` (full final version):

```javascript
function bindEvents() {
  document.getElementById('start-btn')?.addEventListener('click', startGame);

  document.querySelectorAll('.buzz-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      state.buzzedTeam = parseInt(btn.dataset.team);
      state.phase = 'reveal';
      render();
    });
  });

  document.getElementById('correct-btn')?.addEventListener('click', () => {
    state.scores[state.buzzedTeam]++;
    advanceQuestion();
  });

  document.getElementById('wrong-btn')?.addEventListener('click', () => {
    advanceQuestion();
  });

  document.getElementById('play-again-btn')?.addEventListener('click', () => {
    state.phase = 'setup';
    render();
  });
}
```

- [ ] Play through all 10 questions end-to-end in browser. Verify: winner name appears in correct team color, final scores shown, Play Again resets to setup.
- [ ] Commit:
```bash
git add index.html
git commit -m "feat: end screen with winner announcement and play again"
```

---

### Task 8: Deploy to Cloudflare Pages

**Files:**
- Create: `.gitignore`

- [ ] Create `.gitignore`:
```
.DS_Store
```

- [ ] Commit:
```bash
git add .gitignore
git commit -m "chore: add gitignore"
```

- [ ] Push repo to GitHub (create a new GitHub repo named `youth-bible-trivia`, then):
```bash
git remote add origin https://github.com/<your-username>/youth-bible-trivia.git
git push -u origin main
```

- [ ] In Cloudflare Pages dashboard (pages.cloudflare.com):
  - Click "Create a project" → "Connect to Git"
  - Select the `youth-bible-trivia` repo
  - Framework preset: **None**
  - Build command: *(leave empty)*
  - Build output directory: `/` (root)
  - Click "Save and Deploy"

- [ ] Once deployed, go to Settings → Custom Domains → Add `betterquizthanmichael.com`
  - Cloudflare will auto-configure DNS since the domain is already on Cloudflare

- [ ] Visit `betterquizthanmichael.com` — full game works end-to-end in browser
