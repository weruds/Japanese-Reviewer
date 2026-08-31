# 🎌 My Japanese Companion — JLPT Reviewer

A full-featured, browser-based Japanese study app covering all **JLPT levels N5–N1**.  
Practice with quizzes, simulate real exams, race against the clock in Time Attack, and build daily habits with flip-card flashcards — all backed by a live Firebase leaderboard.

> **Live app:** open `app/index.html` in any modern browser — no install, no build step required.

---

## ✨ Features at a Glance

| Feature | Description |
|---|---|
| 🎯 **5 JLPT Levels** | N5 → N1 switchable at any time |
| 📝 **Practice Mode** | 20 questions, no timer, choose section & difficulty |
| 🎯 **Exam Mode** | 40 questions, 60-minute timer, JLPT simulation |
| 💣 **Time Attack** | 20 questions, 90-second bomb, streak bonus points |
| 🃏 **Daily Flashcards** | 10 cards/day with 3D flip, self-rating, progress saved |
| 🏆 **Live Leaderboard** | Firebase-powered, all-time / daily / weekly / monthly |
| ⚡ **Practice Records** | Best score & fastest passing session per level |
| 🔥 **Daily Streak** | Tracks consecutive study days |
| 👤 **User Profiles** | Name + PIN login, avatar upload, security question |
| 📱 **Fully Responsive** | Mobile bottom nav, desktop sidebar, all screen sizes |
| 📖 **Hiragana Tutorial** | Interactive 3-Week Challenge with audio guidance, stroke order & dynamic drills |

---

## 📚 Question Bank Summary

| Level | Vocabulary | Grammar | Kanji | Reading | **Total** |
|-------|-----------|---------|-------|---------|-----------|
| **N5** | 50 | 50 | 50 | 20 | **170** |
| **N4** | 50 | 50 | 20 | 10 | **130** |
| **N3** | 40 | 30 | — | 10 | **80** |
| N2 | *(coming soon)* | | | | |
| N1 | *(coming soon)* | | | | |
| **Total** | | | | | **380** |

### 🃏 Flashcard Bank Summary

| Level | Vocabulary | Grammar | Kanji | **Total** |
|-------|-----------|---------|-------|-----------|
| **N5** | 30 | 20 | 15 | **65** |
| **N4** | 16 | 19 | — | **35** |
| **N3** | 12 | 18 | — | **30** |
| **Total** | | | | **130** |

---

## 🚀 Quick Start

1. **Clone the repo**
   ```bash
   git clone https://github.com/weruds/Japanese-Reviewer.git
   cd Japanese-Reviewer
   ```

2. **Open the app**
   ```
   app/index.html  →  open in Chrome, Edge, Firefox, or Safari
   ```
   No server, no npm, no build step required.

3. **Create a profile**
   - Enter your name and a 4–6 digit PIN
   - Choose a security question (for PIN recovery)
   - Select your JLPT level (N5 is recommended for beginners)

4. **Start studying!**
   - Pick a mode on the Home screen
   - Use **🃏 Flashcards** daily to build vocabulary
   - Check **🏆 Leaderboard** to compete with others

---

## 🗂 Project Structure

```
Japanese-Reviewer/
│
├── app/
│   ├── index.html          # Main application (HTML + CSS + JS — all-in-one)
│   ├── questions-n5.js     # N5 question bank (170 questions)
│   ├── questions-n4.js     # N4 question bank (130 questions)
│   ├── questions-n3.js     # N3 question bank (80 questions)
│   └── flashcards.js       # Flashcard data for all levels (130 cards)
│
├── README.md               # This file
└── .gitignore
```

See [`app/DOCS.md`](app/DOCS.md) for full technical documentation.

---

## 🎮 How to Use Each Mode

### 📝 Practice Mode
- Select **Section** (All / Vocabulary / Grammar / Kanji / Reading)
- Select **Difficulty** (All / Easy / Medium / Hard / Extreme)
- Answer 20 random questions — explanations appear after each answer
- Navigate freely with ← Back / Next → buttons

### 🎯 Exam Mode
- Simulates a real JLPT exam — 40 questions in 60 minutes
- No explanations shown during the exam
- Go back and change answers before finishing
- Auto-submits when the timer hits zero
- **Passing score: 70%** (28/40)

### 💣 Time Attack Mode
- 20 questions, 90-second countdown
- **First click is final** — no going back
- ❌ Wrong answer: **−8 seconds**
- ✅ Streak bonuses: 1st correct +3 pts, 2nd in a row +4 pts, 3rd+ +5 pts
- Bomb explodes when time runs out

### 🃏 Daily Flashcards
- **10 new cards every day** (seeded by date — same deck all day)
- Tap/click the card to flip and reveal the answer
- Rate yourself: **✓ Know It** or **↩ Review Again**
- Progress auto-saved in `localStorage` — resume anytime
- "Review Again" cards listed at the end of the deck
- **Keyboard shortcuts**: `Space` = flip · `→` = Know It · `←` = Review Again
- Filter by section: All / Vocabulary / Grammar / Kanji

### 📖 Hiragana 3-Week Mastery Challenge
- **Structured Schedule**: Program split into Week 1 (Foundations), Week 2 (Progression), and Week 3 (Mastery)
- **Interactive Syllabus & Cards**: Tap any character to read translation, pronunciation guidelines, writing strokes, and mnemonic memory aids
- **Dynamic Drills**: Generates randomized 10-question quizzes for either the active week or all Hiragana, testing both Kana-to-Romaji and Romaji-to-Kana identification
- **Progress Preservation**: Automatically marks characters as learned and saves progress per user to `localStorage`

---

## 🔥 Difficulty Levels

| Level | Description |
|---|---|
| **Easy** | Basic recall, direct definitions |
| **Medium** | Contextual usage, intermediate grammar |
| **Hard** | Edge cases, nuanced differences, keigo |
| **Extreme** | Advanced reasoning, compound nuances, exam-trap questions |

---

## 🏆 Leaderboard

- Powered by **Firebase Firestore** (real-time, shared across all users)
- Separate boards per JLPT level
- Filter by mode: Practice / Exam / Time Attack
- Filter by period: All Time / Daily / Weekly / Monthly
- **Records tracked**: Global highest score & fastest passing session per level

---

## 👤 User Profiles

- **Name + PIN** authentication (no email required)
- PIN is hashed before storage — never stored in plain text
- **Security question** for PIN recovery
- **Avatar upload** (max 2 MB image, stored as base64 in Firestore)
- **Day streak** counter — increments for each day you study
- Works across devices: log in with the same name + PIN on any device

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML5, CSS3, JavaScript (ES6+) |
| Database | Firebase Firestore (compat SDK v10.12.2) |
| Hosting | Any static file server / GitHub Pages |
| Storage | `localStorage` (flashcard progress) + Firestore (scores, profiles) |
| Dependencies | Firebase CDN only — zero npm packages |

---

## ➕ Adding Questions

See the detailed guide in [`app/DOCS.md → Adding Questions`](app/DOCS.md#adding-questions).

**Quick format:**
```js
{
  id: 171,
  level: "N5",
  section: "Vocabulary",   // Vocabulary | Grammar | Kanji | Reading
  topic: "Colors",
  difficulty: "easy",       // easy | medium | hard | extreme
  question: "What does 「あおい」(青い) mean?",
  options: ["A. Red", "B. Yellow", "C. Blue", "D. Green"],
  answer: 2,                // 0-indexed correct answer
  explanation: "青い (あおい) means 'blue'."
}
```

---

## 📅 Roadmap

- [ ] N2 question bank
- [ ] N1 question bank
- [ ] N2 / N1 flashcards
- [x] Hiragana tutorial & dynamic drills (3-Week Challenge)
- [ ] Katakana tutorial & dynamic drills
- [ ] Audio pronunciation (Web Speech API)
- [ ] Spaced repetition system (SRS) for flashcards
- [ ] Admin panel for question management

---

## 📄 License

MIT — free to use, modify, and distribute.

---

*Built with ❤️ for Japanese learners. 頑張れ！🎌*
