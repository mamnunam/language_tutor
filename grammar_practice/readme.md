# AI Language Tutor — Open-Source Template

A personal language tutor powered by Claude, using **spaced repetition** (the SM-2 algorithm) to track what you've learned and schedule exactly when to review it. Works for any language.

---

## What this is

This is not an app. It's a set of instructions and a data file that turn Claude into a structured, persistent language tutor. Every session, Claude reads your knowledge base, runs you through exercises on due topics, and writes the results back — so the next session picks up precisely where you left off.

**Key features:**
- **Spaced repetition**: Topics are scheduled using the SM-2 algorithm (the same algorithm behind Anki). Reviews are timed to happen right before you're likely to forget.
- **Granular error tracking**: Claude records not just which topics you struggle with, but the exact sub-issue (e.g. "uses masculine auxiliary with reflexive verbs", not just "past tense is weak"). This makes subsequent exercises targeted rather than generic.
- **Cross-session memory**: A JSON knowledge base persists between sessions. Claude reads it at the start of every session and updates it at the end.
- **Two modes**: Revision (working through due topics) and New Material (verifying and logging exercises from your textbook or class).
- **Language-agnostic**: The system works for any language — change the config and go.

---

## How it works

### The knowledge base
Your progress lives in `knowledge_base/learner.json`. Each topic you've studied is an entry with:

- **Scheduling fields**: `next_review`, `interval`, `repetitions`, `ease_factor` — managed by the SM-2 algorithm
- **`weak_spots`**: an array of precise issue descriptions observed during sessions (e.g. `"omits elision before vowel-initial words"`)
- **`persistent_patterns`**: root-level array of cross-topic errors that recur regardless of which topic is being reviewed

Claude reads this at session start and writes it back at session end. You never need to touch it manually.

### The SM-2 algorithm
After each topic, you rate your recall from 1–4:

| Rating | Meaning |
|--------|---------|
| 1 | Forgot / mostly wrong |
| 2 | Hard / significant errors |
| 3 | Good, with some effort |
| 4 | Easy, confident recall |

The algorithm uses this to calculate your next review date. Topics you find easy get pushed further out (weeks, then months). Topics you struggle with come back the next day. The ease factor — a per-topic multiplier — adjusts based on your history, so the system calibrates to you over time.

### Session flow
1. Claude loads your KB and identifies due topics
2. You choose: revision, new material, or both
3. **Revision**: Claude generates exercises (recognition → production → translation → free writing) for each due topic, one at a time. You answer, Claude corrects, you rate, Claude schedules the next review.
4. **New material**: You bring exercises or questions from class. Claude checks your answers, explains errors, then logs the new topics with your confidence rating.
5. At the end, Claude summarises the session, highlights remaining gaps, and writes the updated KB.

---

## Setup

### 1. Install Claude Desktop
Download from [claude.ai/download](https://claude.ai/download). You need the desktop app (not the web version) to use Cowork mode, which gives Claude persistent file access.

### 2. Enable Cowork mode
Open Claude Desktop and enable **Cowork** from the sidebar. Select this folder (`language-tutor-template/`) as your workspace when prompted.

### 3. Configure your learner profile
Edit `CLAUDE.md` and fill in the configuration block at the top:

```
Learner name:       Your name
Target language:    The language you're learning
Native language:    Your native language
Knowledge base:     knowledge_base/learner.json
```

### 4. Set up your knowledge base
Rename `knowledge_base/learner_template.json` to `knowledge_base/learner.json` and fill in the top-level fields:

```json
{
  "user": "Your name",
  "language": "Target language",
  "native_language": "Native language",
  ...
}
```

Leave `topics` as an empty array — Claude will populate it as you study.

### 5. Start a session
Open Cowork and type something like:
- *"Let's start a session"*
- *"I want to practice"*
- *"I have exercises from chapter 3"*

Claude will read `CLAUDE.md` automatically (it's in the workspace) and follow the protocol from there.

---

## Adding topics for the first time

If you already have a list of topics you've studied (from a class, textbook, or previous self-study), tell Claude:

> *"I've already covered these topics: [list]. Can you add them to my KB?"*

Claude will ask for a confidence rating (1–4) for each one and use those to calibrate the starting ease factor and initial review date. No prior session history needed.

---

## Multiple learners on one device

The system assumes a single learner by default. To support multiple learners:

1. Create a separate KB file per learner: `knowledge_base/alice.json`, `knowledge_base/bob.json`, etc.
2. Update `CLAUDE.md` to ask for the learner's name at session start and load the matching file.

---

## Customising exercise types

The default exercise progression (recognition → controlled production → guided translation → free production) works well for most languages, but some languages benefit from additional types. You can add a note to `CLAUDE.md` under a new section, for example:

```
## 9. LANGUAGE-SPECIFIC NOTES

For Japanese:
- Add a script reading exercise before controlled production (e.g. "Read this kanji aloud")
- For vocabulary topics, always include the pitch accent pattern if known
- Track hiragana/katakana/kanji reading as separate weak_spots
```

---

## File structure

```
language-tutor-template/
├── CLAUDE.md                        ← Claude's operating manual (edit the config at the top)
├── README.md                        ← This file
└── knowledge_base/
    ├── learner_template.json        ← Blank template (copy and rename to learner.json)
    └── learner.json                 ← Your actual knowledge base (created from template)
```

---

## The knowledge base schema

```json
{
  "user": "string",
  "language": "string",
  "native_language": "string",
  "created": "YYYY-MM-DD",
  "last_updated": "YYYY-MM-DD",
  "total_sessions": 0,
  "persistent_patterns": [
    { "issue": "exact description of cross-topic error", "since": "YYYY-MM-DD" }
  ],
  "topics": [
    {
      "id": "snake_case_unique_id",
      "name": "Human readable topic name",
      "category": "grammar | vocabulary | phonetics | other",
      "subcategory": "verbs | nouns | adjectives | pronouns | adverbs | prepositions | conjunctions | articles | tenses | particles | classifiers | scripts | sentence_structure | other",
      "first_added": "YYYY-MM-DD",
      "last_reviewed": "YYYY-MM-DD or null",
      "next_review": "YYYY-MM-DD",
      "interval": 1,
      "repetitions": 0,
      "ease_factor": 2.5,
      "weak_spots": null,
      "status": "new | active | mature"
    }
  ]
}
```

**`status` values:**
- `new` — added but never reviewed
- `active` — in rotation (interval < 21 days)
- `mature` — well-learned (interval ≥ 21 days)

---

## Tips for good sessions

**Be specific when describing errors.** The more specific the weak spot, the more targeted future exercises will be. "Uses the wrong case" is not useful; "uses accusative instead of dative after *mit*" is.

**Rate honestly.** It's tempting to rate yourself a 3 when you barely made it, but accurate ratings are what make the algorithm work. A rating of 2 just means "review again tomorrow" — not a failure.

**Bring your own material.** Mode B is most valuable when you bring real exercises from a class or textbook. Claude will check them, explain mistakes, and log everything — turning passive homework into active review.

**Short sessions are fine.** Even 15 minutes of focused revision is enough to move topics forward. The algorithm handles the rest.

---

## Credits and methodology

The spaced repetition algorithm is a direct implementation of **SM-2**, originally designed by Piotr Woźniak and used in SuperMemo and Anki. The error-tracking and session protocol were designed to complement SM-2 with the kind of nuanced, conversational feedback that a human tutor would provide — capturing not just *what* you got wrong, but *why*, at a level of specificity that a scheduling algorithm alone cannot produce.
