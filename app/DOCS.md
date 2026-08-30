# 📖 My Japanese Companion — Technical Documentation

> Full reference for developers, contributors, and anyone who wants to extend or understand the codebase.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [File Reference](#2-file-reference)
3. [Application Screens & Navigation](#3-application-screens--navigation)
4. [App Module (JavaScript)](#4-app-module-javascript)
5. [Firebase & Data Schema](#5-firebase--data-schema)
6. [Question Bank Schema](#6-question-bank-schema)
7. [Flashcard Data Schema](#7-flashcard-data-schema)
8. [Adding Questions](#8-adding-questions)
9. [Adding Flashcards](#9-adding-flashcards)
10. [Adding a New JLPT Level (N2 / N1)](#10-adding-a-new-jlpt-level-n2--n1)
11. [CSS Design System](#11-css-design-system)
12. [Authentication & Security](#12-authentication--security)
13. [Flashcard Engine](#13-flashcard-engine)
14. [Leaderboard System](#14-leaderboard-system)
15. [Streak System](#15-streak-system)
16. [Practice Records System](#16-practice-records-system)
17. [Time Attack Mode](#17-time-attack-mode)
18. [Keyboard Shortcuts](#18-keyboard-shortcuts)
19. [Local Storage Keys](#19-local-storage-keys)
20. [Deploying to GitHub Pages](#20-deploying-to-github-pages)

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Browser (Client)                      │
│                                                         │
│  index.html ──loads──► questions-n5/n4/n3.js           │
│       │                flashcards.js                    │
│       │                                                 │
│       └──► App (IIFE module)                            │
│              ├── Login / Profile management             │
│              ├── Quiz engine (Practice/Exam/TimeAttack) │
│              ├── Flashcard engine                       │
│              ├── Leaderboard renderer                   │
│              └── Streak & Records tracker               │
│                          │                              │
│                          ▼                              │
│              Firebase Firestore (cloud)                 │
│              ├── jp_users        (profiles)             │
│              ├── jp_leaderboard  (scores)               │
│              ├── jp_sessions     (full session logs)    │
│              └── jp_records      (global records)       │
└─────────────────────────────────────────────────────────┘
```

The app is a **single-page application (SPA)** with no routing framework.  
Sections are shown/hidden via CSS classes (`hidden`). All logic lives inside a single IIFE (`App`) in `index.html`.

---

## 2. File Reference

| File | Purpose | Size |
|------|---------|------|
| `app/index.html` | Entire app — HTML structure, CSS styles, JavaScript logic | ~2,100 lines |
| `app/questions-n5.js` | N5 question bank | 170 questions |
| `app/questions-n4.js` | N4 question bank | 130 questions |
| `app/questions-n3.js` | N3 question bank | 80 questions |
| `app/flashcards.js` | Flashcard data for N5/N4/N3 | 130 cards |
| `README.md` | User-facing documentation | — |
| `app/DOCS.md` | This file — technical reference | — |

### How files are loaded

```html
<script src="questions-n5.js"></script>   <!-- sets window.QUESTIONS_N5 -->
<script src="questions-n4.js"></script>   <!-- sets window.QUESTIONS_N4 -->
<script src="questions-n3.js"></script>   <!-- sets window.QUESTIONS_N3 -->
<script src="flashcards.js"></script>     <!-- sets window.FLASHCARDS   -->
```

The `App` module reads these globals via `getBank(level)` and `window.FLASHCARDS`.

---

## 3. Application Screens & Navigation

### Screens

| Screen ID | Shown When |
|-----------|-----------|
| `#screen-username` | Before login |
| `#screen-main` | After login |

### Sections (inside `#screen-main`)

| Section ID | Nav label | Activated by |
|------------|-----------|-------------|
| `#section-home` | 🏠 Home | Default after login |
| `#section-flashcards` | 🃏 Flashcards | Nav click → `App.showSection('flashcards')` |
| `#section-quiz` | *(no nav — entered via Start Session)* | `App.startSession()` |
| `#section-results` | *(no nav — shown after quiz ends)* | `App.finishSession()` |
| `#section-leaderboard` | 🏆 Leaderboard | Nav click → `App.showSection('leaderboard')` |

### Navigation elements

- **Desktop**: `.nav-menu #main-nav` — horizontal tab row (hidden on mobile via `@media`)
- **Mobile**: `#mobile-bottom-nav` — fixed bottom bar with Home / Cards / Leaderboard tabs

---

## 4. App Module (JavaScript)

The entire application logic is encapsulated in a single **IIFE** that returns a public API:

```js
const App = (() => {
  // private state & functions
  return { /* public API */ };
})();
```

### State variables

| Variable | Type | Description |
|----------|------|-------------|
| `username` | string | Currently logged-in user |
| `activeLevel` | string | Current JLPT level (`'N5'`–`'N1'`) |
| `mode` | string | `'practice'` \| `'exam'` \| `'time-attack'` |
| `difficulty` | string | `'all'` \| `'easy'` \| `'medium'` \| `'hard'` \| `'extreme'` |
| `sectionFilter` | string | `'all'` \| `'Vocabulary'` \| `'Grammar'` \| `'Kanji'` \| `'Reading'` |
| `sessionQuestions` | array | The shuffled question set for the current session |
| `currentIndex` | number | Current question index (0-based) |
| `score` | number | Correct answer count |
| `sessionAnswers` | array | Per-question answer records |
| `taTimeLeft` | number | Time Attack seconds remaining |
| `taPoints` | number | Time Attack accumulated points |
| `taStreak` | number | Time Attack consecutive correct count |
| `_cachedUsers` | object | In-memory cache of Firestore user docs |

### Public API

| Method | Description |
|--------|-------------|
| `App.setLevel(level)` | Switch active JLPT level; updates all UI buttons |
| `App.handleLogin()` | Process login form (new user or returning user) |
| `App.selectReturningUser(name)` | Highlight a returning user chip and show PIN field |
| `App.cancelVerify()` | Reset the verify/forgot PIN flow |
| `App.showForgotFlow()` | Show the security question reset flow |
| `App.cancelForgot()` | Cancel the forgot-PIN flow |
| `App.submitForgot()` | Validate security answer and reset PIN |
| `App.signOut()` | Clear session and return to login screen |
| `App.openProfile()` | Open the profile modal |
| `App.closeProfile()` | Close the profile modal |
| `App.saveProfile()` | Save avatar change to Firestore |
| `App.showSection(sec)` | Navigate to a section by name |
| `App.selectMode(mode)` | Select quiz mode on Home screen |
| `App.setDiff(diff)` | Set difficulty filter |
| `App.setSection(sec)` | Set section filter for Practice mode |
| `App.startSession()` | Build question pool and start quiz |
| `App.nextQuestion()` | Advance to next question |
| `App.prevQuestion()` | Go back to previous question |
| `App.confirmQuit()` | Prompt user and quit current session |
| `App.finishSession()` | Score the session, save to Firestore, show results |
| `App.setLbMode(mode)` | Switch leaderboard mode tab |
| `App.setLbPeriod(period)` | Switch leaderboard time period tab |
| `App.setSidebarLbMode(mode)` | Switch sidebar top-5 leaderboard mode |
| `App.closeBanner()` | Dismiss the record-broken banner |
| `App.fcFlipCard()` | Flip the current flashcard |
| `App.fcRate(rating)` | Rate a flashcard (`'know'` or `'again'`) |
| `App.fcSetSection(sec)` | Filter flashcard deck by section |
| `App.fcRestart()` | Restart today's flashcard deck |

---

## 5. Firebase & Data Schema

### Configuration

The app uses **Firebase Firestore** (compat SDK v10.12.2).  
The project is `myistqbreviewer` (shared with the ISTQB reviewer — Japanese collections use the `jp_` prefix).

```js
firebase.initializeApp({
  apiKey: "...",
  authDomain: "myistqbreviewer.firebaseapp.com",
  projectId: "myistqbreviewer",
  ...
});
```

> ⚠️ To use your own Firebase project, replace the config object in `index.html` (lines 29–36) and create the four collections below.

### Firestore Collections

#### `jp_users` — one document per username

```
Document ID: <username>
Fields:
  entryIdHash    string   — hashed PIN
  secqKey        string   — security question key (e.g. 'pet', 'city')
  secqHash       string   — hashed security answer
  streak         number   — current day streak
  lastActiveDate string   — ISO date 'YYYY-MM-DD' of last activity
  avatar         string   — base64 data URL of profile photo (optional)
  following      array    — list of usernames this user follows (optional)
  lastRecordCheck number  — timestamp of last followed-user record check
```

#### `jp_leaderboard` — one document per completed session

```
Document ID: 'lb_<timestamp>_<random>'
Fields:
  username     string   — player name
  level        string   — 'N5' | 'N4' | 'N3' | 'N2' | 'N1'
  mode         string   — 'practice' | 'exam' | 'time-attack'
  score        number   — correct answers
  total        number   — total questions
  pct          number   — percentage (0–100)
  pass         boolean  — whether the session was a pass
  difficulty   string   — difficulty filter used
  section      string   — section filter used
  timeTaken    number   — seconds elapsed
  taPoints     number   — Time Attack points (null if not TA)
  ts           number   — Unix timestamp (ms)
  date         string   — formatted date string
  bank         string   — 'jp_n5' | 'jp_n4' | 'jp_n3' etc.
```

#### `jp_sessions` — full session logs (source of truth)

```
Document ID: 'sid_<timestamp>_<random>'
Fields:
  (all leaderboard fields above, plus:)
  sessionId    string   — matches document ID
  answers      array    — per-question answer records:
    [ { qId, question, chosen, correct, isCorrect }, ... ]
```

#### `jp_records` — global records (one document per record type)

```
Document ID: 'jp_n5_highest' | 'jp_n5_fastest' | 'jp_n4_highest' | ...
Fields:
  username     string
  pct          number
  score        number
  total        number
  timeTaken    number
  date         string
  ts           number
```

### Firestore helper functions (private)

| Function | Description |
|----------|-------------|
| `loadUsers()` | Fetch all user docs |
| `getUser(name)` | Fetch a single user doc |
| `upsertUser(name, patch)` | Create or merge a user doc |
| `loadLB()` | Fetch up to 2,000 recent leaderboard entries |
| `saveLBEntry(entry)` | Save a new leaderboard entry |
| `saveSession(record)` | Save a full session record |
| `loadRecord(id)` | Load a specific records doc |
| `saveRecord(id, data)` | Save/overwrite a records doc |

---

## 6. Question Bank Schema

Each question bank file (e.g. `questions-n5.js`) sets a global array:

```js
window.QUESTIONS_N5 = [ /* question objects */ ];
```

### Question object fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | number | ✅ | Unique within the file (1, 2, 3 …) |
| `level` | string | ✅ | `'N5'` \| `'N4'` \| `'N3'` \| `'N2'` \| `'N1'` |
| `section` | string | ✅ | `'Vocabulary'` \| `'Grammar'` \| `'Kanji'` \| `'Reading'` |
| `topic` | string | ✅ | Subtopic label (e.g. `'Particles'`, `'Colors'`) |
| `difficulty` | string | ✅ | `'easy'` \| `'medium'` \| `'hard'` \| `'extreme'` |
| `question` | string | ✅ | Question text (use 「」for Japanese) |
| `options` | string[] | ✅ | Exactly 4 options, each starting with `'A. '`, `'B. '`, `'C. '`, `'D. '` |
| `answer` | number | ✅ | **0-indexed** index of the correct option |
| `explanation` | string | ✅ | Shown after answering in Practice mode |

### Example

```js
{
  id: 1,
  level: "N5",
  section: "Vocabulary",
  topic: "Food",
  difficulty: "easy",
  question: "What does 「みず」(水) mean?",
  options: ["A. Tea", "B. Juice", "C. Milk", "D. Water"],
  answer: 3,
  explanation: "水 (みず) means 'water'. One of the most basic N5 words."
}
```

### Question pool selection logic

```
startSession()
  └─ getBank(activeLevel)         // returns window.QUESTIONS_N5 etc.
       ├─ filter by sectionFilter  (if not 'all')
       ├─ filter by difficulty     (if not 'all')
       ├─ shuffle (Fisher-Yates)
       └─ slice to session size    (20 practice, 40 exam, 20 TA)
```

---

## 7. Flashcard Data Schema

`flashcards.js` sets a single global array used by all levels:

```js
window.FLASHCARDS = [ /* card objects */ ];
```

### Flashcard object fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique ID, e.g. `'FC001'` |
| `level` | string | `'N5'` \| `'N4'` \| `'N3'` \| `'N2'` \| `'N1'` |
| `section` | string | `'Vocabulary'` \| `'Grammar'` \| `'Kanji'` |
| `type` | string | Word type shown on back (e.g. `'Noun'`, `'Verb (う)'`, `'Grammar pattern'`) |
| `front` | string | Japanese displayed on card front (kanji/kana/pattern) |
| `reading` | string | Furigana / reading in hiragana |
| `romaji` | string | Romanization |
| `meaning` | string | English meaning |
| `example` | string | Example sentence in Japanese |
| `exampleReading` | string | Example sentence furigana |
| `exampleMeaning` | string | English translation of the example |

### Example

```js
{
  id: 'FC001',
  level: 'N5',
  section: 'Vocabulary',
  type: 'Noun',
  front: '水',
  reading: 'みず',
  romaji: 'mizu',
  meaning: 'Water',
  example: '水を飲みます。',
  exampleReading: 'みずをのみます。',
  exampleMeaning: 'I drink water.'
}
```

---

## 8. Adding Questions

### Step 1 — Choose the right file

| Level | File |
|-------|------|
| N5 | `app/questions-n5.js` |
| N4 | `app/questions-n4.js` |
| N3 | `app/questions-n3.js` |
| N2 | Create `app/questions-n2.js` (see §10) |

### Step 2 — Find the last ID

Open the file and check the last `id` value. Increment by 1.

### Step 3 — Add your question object

Append inside the `window.QUESTIONS_Nx = [ ... ]` array:

```js
{
  id: 171,
  level: "N5",
  section: "Vocabulary",   // Vocabulary | Grammar | Kanji | Reading
  topic: "Colors",
  difficulty: "easy",      // easy | medium | hard | extreme
  question: "What does 「あおい」(青い) mean?",
  options: [
    "A. Red",
    "B. Yellow",
    "C. Blue",
    "D. Green"
  ],
  answer: 2,               // 0-indexed → "C. Blue" is correct
  explanation: "青い (あおい) means 'blue'. Common N5 color adjective."
},
```

### Step 4 — Validate

- `answer` must be `0`, `1`, `2`, or `3`
- `options` must have exactly 4 elements
- No duplicate `id` values within the file
- `level` must match the file (e.g. `"N5"` in `questions-n5.js`)

---

## 9. Adding Flashcards

Open `app/flashcards.js` and append to the `window.FLASHCARDS` array.

```js
{
  id: 'FC131',             // next sequential ID
  level: 'N5',
  section: 'Vocabulary',  // Vocabulary | Grammar | Kanji
  type: 'Adjective (い)', // free text shown on card back
  front: '青い',
  reading: 'あおい',
  romaji: 'aoi',
  meaning: 'Blue',
  example: '空が青いです。',
  exampleReading: 'そらがあおいです。',
  exampleMeaning: 'The sky is blue.'
},
```

**Best practices:**
- Keep `front` short — it's displayed at `2.4rem` on the card face
- Example sentences should use the target word naturally
- `type` should follow the conventions already used (Noun, Verb (う), Verb (る), Adjective (い), Adjective (な), Grammar pattern, Kanji, Keigo, Particle, Expression)

---

## 10. Adding a New JLPT Level (N2 / N1)

### Step 1 — Create the question file

```js
// app/questions-n2.js
window.QUESTIONS_N2 = [
  {
    id: 1,
    level: "N2",
    section: "Vocabulary",
    topic: "...",
    difficulty: "medium",
    question: "...",
    options: ["A. ...", "B. ...", "C. ...", "D. ..."],
    answer: 0,
    explanation: "..."
  },
  // ...
];
```

### Step 2 — Add the `<script>` tag in `index.html`

```html
<script src="questions-n3.js"></script>
<script src="questions-n2.js"></script>   <!-- add this line -->
<script src="flashcards.js"></script>
```

### Step 3 — Register in `getBank()`

Find the `getBank` function in `index.html` and add the new level:

```js
function getBank(level) {
  if (level === 'N5') return window.QUESTIONS_N5 || [];
  if (level === 'N4') return window.QUESTIONS_N4 || [];
  if (level === 'N3') return window.QUESTIONS_N3 || [];
  if (level === 'N2') return window.QUESTIONS_N2 || [];  // add this
  return [];
}
```

That's all — the level switcher buttons (N2) are already in the UI and will automatically start working once the question bank is non-empty.

---

## 11. CSS Design System

### CSS Custom Properties (variables)

```css
:root {
  --primary:       #1b0062;   /* Deep indigo — main brand color */
  --primary-light: #2d0a8f;   /* Lighter indigo */
  --accent:        #c0392b;   /* Red — Japanese flag inspired */
  --success:       #2e7d32;   /* Green — correct answers */
  --danger:        #c62828;   /* Red — wrong answers */
  --warning:       #e65100;   /* Orange — medium difficulty, timer warning */
  --bg:            #f5f0ff;   /* Light purple-tinted page background */
  --card:          #ffffff;   /* Card background */
  --text:          #1a1a2e;   /* Main text */
  --muted:         #546e7a;   /* Secondary text */
  --border:        #d5ccee;   /* Card/input borders */
  --easy:          #1b5e20;   /* Easy difficulty */
  --medium:        #e65100;   /* Medium difficulty */
  --hard:          #b71c1c;   /* Hard difficulty */
  --extreme:       #4a148c;   /* Extreme difficulty */
  --bottom-nav-h:  60px;      /* Mobile bottom nav height */
}
```

### Breakpoints

| Name | Range | Notes |
|------|-------|-------|
| Mobile | ≤ 539px | Single-column, larger touch targets |
| Tablet | 540–767px | Bottom nav enabled, 2-column grids |
| Laptop | 768–1023px | Desktop nav hidden for bottom nav |
| Desktop | 1024–1399px | Two-column layout with sidebar |
| Large monitor | ≥ 1400px | Wider container |

### Key class naming conventions

| Prefix | Used for |
|--------|---------|
| `.card` | White content cards |
| `.btn`, `.btn-*` | All buttons |
| `.fc-*` | Flashcard system |
| `.lb-*` | Leaderboard |
| `.ta-*` | Time Attack |
| `.prep-*` | Preparedness meter |
| `.rec-*` | Practice records |
| `.mob-*` | Mobile-specific |
| `.q-*` | Quiz header badges |

---

## 12. Authentication & Security

### How it works

There is **no traditional authentication** — login is name + PIN only.

```
Login flow:
  Username + PIN entered
       │
       ▼
  getUser(name) from Firestore
       │
       ├─ User exists? → verify: hashEntry(PIN) === user.entryIdHash
       │                         ✓ → loginAs(name)
       │                         ✗ → show error
       │
       └─ New user? → create doc with:
                      entryIdHash: hashEntry(PIN)
                      secqKey / secqHash (security question)
```

### PIN hashing

```js
function hashEntry(s) {
  let h = 0;
  for (let i = 0; i < s.length; i++) {
    h = ((h << 5) - h) + s.charCodeAt(i);
    h |= 0;
  }
  return 'eid_' + Math.abs(h).toString(36);
}
```

> ⚠️ This is a simple djb2-style hash. It is sufficient for a low-stakes study app but is **not cryptographically secure**. Do not use this pattern for sensitive data.

### Session persistence

```js
// On login
sessionStorage.setItem('jp_session_user', name);

// On page load (auto-restore)
const saved = sessionStorage.getItem('jp_session_user');
if (saved) loginAs(saved, true);  // isRestore=true skips streak update

// On sign out
sessionStorage.removeItem('jp_session_user');
```

### Content protection

The app blocks right-click, F12, Ctrl+U, Ctrl+S, Ctrl+P, PrintScreen, and text selection to discourage copying question content:

```js
document.addEventListener('contextmenu', e => e.preventDefault());
document.addEventListener('keydown', e => { /* block DevTools shortcuts */ });
document.addEventListener('copy', e => e.preventDefault());
```

---

## 13. Flashcard Engine

### Daily deck generation

The daily deck is **deterministically shuffled** — every user sees the same 10 cards for a given date + level + section combination, meaning the deck is stable throughout the day but changes at midnight.

```js
// Seed = date string + username + level + section
const seedStr = '2025-01-15' + 'Tanaka' + 'N5' + 'all';
// → seeded Fisher-Yates shuffle → slice first 10
```

### Progress persistence

Progress is stored in `localStorage` per user+date+level+section:

```
Key:   'fc_2025-01-15_Tanaka_N5_all'
Value: { index: 4, known: 3, again: 1, done: false, againIds: ['FC012'] }
```

This means:
- Closing the tab mid-deck resumes from where you left off
- Progress resets automatically the next day (new key)

### Deck lifecycle

```
fcInitDeck()
    │
    ├─ fcGetDayDeck(level, section)   → build shuffled 10-card deck
    ├─ fcLoadProgress()               → check localStorage
    │       ├─ done=true? → fcShowComplete()
    │       └─ partial? → restore index/known/again
    │
    └─ fcRenderCard()
           │
           └─ User taps card → fcFlipCard()
                    │
                    └─ Buttons unlock → fcRate('know' | 'again')
                             │
                             ├─ fcIndex++ → next card → fcRenderCard()
                             └─ last card → fcShowComplete()
```

---

## 14. Leaderboard System

### Entry saved after every completed session

```js
// Entry structure saved to jp_leaderboard
{
  username, level, mode,
  score, total, pct, pass,
  difficulty, section,
  timeTaken, taPoints,
  ts, date, bank
}
```

### Rendering pipeline

```
renderLeaderboard()
  └─ loadLB()                     // fetch up to 2,000 recent entries
       ├─ filter by mode           // practice | exam | time-attack
       ├─ filter by bank           // jp_n5 | jp_n4 | ...
       ├─ filter by period         // daily | weekly | monthly | all-time
       ├─ sort by pct desc, score desc, timeTaken asc
       └─ deduplicate (best entry per username)
            └─ render HTML table
```

### Time periods (UTC)

| Period | Cutoff |
|--------|--------|
| Daily | Now − 86,400,000 ms |
| Weekly | Now − 604,800,000 ms |
| Monthly | Now − 2,592,000,000 ms |
| All Time | No cutoff |

---

## 15. Streak System

A streak increments when the user logs in on consecutive calendar days (UTC).

```js
async function updateStreak(name) {
  const user = await getUser(name);
  const today = todayStr();             // 'YYYY-MM-DD'
  if (user.lastActiveDate === today) return;  // already counted today

  const yesterday = /* yesterday string */;
  let streak = user.streak || 0;
  if (user.lastActiveDate === yesterday) streak++;  // consecutive
  else streak = 1;                                   // broken or new

  await upsertUser(name, { streak, lastActiveDate: today });
}
```

> Streak is updated on login, not on quiz completion. Logging in counts as studying for the day.

---

## 16. Practice Records System

Two global records are tracked per JLPT level:

| Record key | Tracks |
|------------|--------|
| `jp_n5_highest` | Highest practice score (%) |
| `jp_n5_fastest` | Fastest passing practice session (seconds) |

These are stored in the `jp_records` Firestore collection and compared after each Practice mode session. If broken, an animated banner slides in from the top.

---

## 17. Time Attack Mode

### Constants

```js
const TA_TOTAL_TIME = 90;   // starting seconds
const TA_PENALTY    = 8;    // seconds deducted per wrong answer
const TA_Q_COUNT    = 20;   // questions per session
```

### Point system

| Streak | Points awarded |
|--------|---------------|
| 1st correct in a row | +3 pts |
| 2nd correct in a row | +4 pts |
| 3rd+ correct in a row | +5 pts |
| Wrong answer | 0 pts + −8 sec |

### Timer implementation

```js
taTimerInterval = setInterval(() => {
  taTimeLeft--;
  // Update clock display and countdown bar
  if (taTimeLeft <= 20) bomb icon starts ticking animation
  if (taTimeLeft <= 0)  triggerBombExplosion() → endTimeAttack()
}, 1000);
```

### `taPoints` is saved in the leaderboard entry and displayed in the Time Attack leaderboard tab.

---

## 18. Keyboard Shortcuts

### During Quiz (Practice / Exam)

| Key | Action |
|-----|--------|
| `←` / `→` arrows | Navigate Back / Next |

### During Flashcards

| Key | Action |
|-----|--------|
| `Space` or `Enter` | Flip card |
| `→` (after flip) | Rate: Know It ✓ |
| `←` (after flip) | Rate: Review Again ↩ |

Keyboard events are scoped — flashcard shortcuts only fire when `#section-flashcards` is visible.

---

## 19. Local Storage Keys

| Key pattern | Value | Expires |
|-------------|-------|---------|
| `fc_YYYY-MM-DD_<username>_<level>_<section>` | `{ index, known, again, done, againIds }` | Never (stale keys accumulate — harmless) |

Session restoration uses `sessionStorage`:

| Key | Value |
|-----|-------|
| `jp_session_user` | username string |

---

## 20. Deploying to GitHub Pages

1. Go to your repo → **Settings** → **Pages**
2. Source: **Deploy from a branch** → branch `master` → folder `/ (root)`
3. Save — GitHub will publish at `https://weruds.github.io/Japanese-Reviewer/`

> Since the app files live in `app/`, the URL to share will be:  
> `https://weruds.github.io/Japanese-Reviewer/app/index.html`

Alternatively, move all files to the repo root and update script `src` paths accordingly for a cleaner URL.

---

*Last updated: 2025 · My Japanese Companion v1.0*
