# Kasus Apparatus

Interactive single-file web app to learn and drill German grammatical cases
(Nominative, Accusative, Dative, Genitive). Built for a Spanish-speaking
learner who already knows enough English to work in that language, currently
targeting beginner/intermediate level.

## Status

Working prototype, all core features implemented. Not yet deployed anywhere —
currently a single static HTML file, no backend, no persistence between
sessions.

## Files

- `index.html` — the entire app (HTML + CSS + JS in one file). No build step,
  no dependencies except two Google Fonts loaded via CDN link tag
  (`Special Elite`, `IBM Plex Sans`, `IBM Plex Mono`). Works fully offline
  except for font loading (falls back to system fonts without internet).

## How it's structured (for whoever picks this up in Claude Code)

Single HTML file, roughly in this order:
1. `<style>` block — all CSS, uses CSS custom properties (`:root` vars) for
   the color palette. Search for `/* Game card */`, `/* Theory accordion */`,
   `/* Difficulty toggle */` etc. to jump to a section.
2. `<body>` — three top-level "modes" toggled by JS (`.mode-panel`,
   `.mode-btn`), not routes, just show/hide divs:
   - `mode-theorie` — grammar explanation, one `<details>` accordion card per
     case, plus a "3 questions to ask yourself" intro block.
   - `mode-tabelle` — static reference table (articles by case/gender) +
     verb lists + preposition lists.
   - `mode-spiel` — the actual game/quiz.
3. `<script>` — all game logic, vanilla JS, no framework, no build step.

## Design concept

Themed around German bureaucratic paperwork (`Durchschlagformular` —
carbon-copy triplicate forms). Each grammatical case is styled like a
different colored copy in a form pad:
- Nominative → cream/white
- Accusative → dusty pink
- Dative → muted mustard/yellow
- Genitive → powder blue

Feedback uses a rubber-stamp visual (rotated box, ink-like border) that reads
`RICHTIG` / `FALSCH` / `BEIDE RICHTIG` / `KASUS RICHTIG` / `ARTIKEL RICHTIG`
depending on outcome (see Game logic below).

Fonts: `Special Elite` (typewriter, for headings/stamps) + `IBM Plex Sans`
(body) + `IBM Plex Mono` (data/labels/buttons), reinforcing the "office
document" feel.

## Game logic (the core of index.html's <script>)

### Data
- `nouns` — object of nouns with their gender and declined article per case
  (`{gender, word, forms:{nom,akk,dat,gen}}`). Currently 12 nouns covering
  masculine, feminine, neuter, and one plural.
- `bank` — array of ~24 practice items:
  `{cue, tpl, noun, case, clue, explain}`
  - `cue`: English translation of the sentence (learner reads this first)
  - `tpl`: German sentence template, `___` marks where the article blank goes
  - `noun`: key into `nouns`
  - `case`: the correct case (`nom`/`akk`/`dat`/`gen`)
  - `clue`: the literal substring in `tpl` that determines the case (a verb
    or preposition), or `null` if there's no fixed clue (plain subject/
    possession cases) — used to highlight the word after step 1
  - `explain`: HTML string shown after answering, explains the reasoning

### Two-step flow (this was a deliberate fix — see history below)
Each question is answered in two steps so the learner practices *identifying*
the case, not just declining an already-known case:
1. **Step 1 — Identify the case**: shown the English cue + German sentence
   (no color-coding, no case revealed). Learner picks from 4 case-name
   buttons. On answer: buttons get disabled + colored (correct/wrong), the
   `clue` word gets highlighted inline in the German sentence (or a note
   explains why there's no clue word), and step 1's buttons **stay visible**
   (important: earlier version hid them, which made it impossible to tell
   *why* the final verdict was wrong — see bug notes below).
2. **Step 2 — Pick the article**: 4 article-form buttons (der/die/das/den/
   dem/des subset) appear. On answer: card background switches to the
   case's theme color, stamp animates in, explanation shows, "Next card"
   button appears.

### Scoring
- `score` increments by 1 for each correct sub-answer (case correct, article
  correct) — so up to 2 points per card.
- `streak` only increments if **both** steps were correct; resets to 0
  otherwise.
- Wrong cards (either step wrong) get pushed to `wrongQueue` and re-surface
  once the main queue empties (lightweight Leitner-style repetition).

### Stamp outcomes (4 states, not just right/wrong)
This was specifically requested after a bug where "article correct, case
wrong" showed a plain FALSCH stamp with no way to tell which part failed:
- Both correct → `BEIDE RICHTIG` (navy)
- Case correct, article wrong → `KASUS RICHTIG` (mustard/amber, "partial")
- Case wrong, article correct → `ARTIKEL RICHTIG` (mustard/amber, "partial")
- Both wrong → `FALSCH` (red)

### Difficulty toggle (inside mode-spiel)
- **Easy** (default): a compact reference table + preposition lists sit
  permanently above the game card (`#easyTable`, class `.show` toggled).
- **Hard**: table hidden, learner plays from memory only.
This is a simple CSS class toggle, no separate game logic.

## Content notes / theory accuracy

Theory content (case definitions, verb lists, preposition-case groupings)
was cross-checked against the user's own reference PDF ("Basic German",
project knowledge). Confirmed accurate. Two prepositions were added after
that check to match the source more completely: `bis` (accusative), `außer`
and `gegenüber` (dative). If more content is added later, cross-check against
that same source for consistency.

## History / iteration notes (context for why things look the way they do)

1. Started as a simple single-step quiz (case was pre-selected via colored
   tab, learner only picked the article). User pointed out this never
   actually taught *case identification*, which was the real pain point.
2. Split into the two-step flow described above.
3. First version of two-step flow hid the step-1 (case) buttons once step 2
   appeared. Bug: user got the case wrong but the article right, saw a
   correct-looking "der" pick, then got confused why the overall stamp said
   FALSCH — because they couldn't see their earlier wrong case pick anymore.
   Fixed by keeping step 1 buttons visible (disabled, colored) through step 2.
4. That led directly to the 4-state stamp system (BEIDE/KASUS/ARTIKEL/FALSCH)
   so partial correctness is communicated clearly instead of a binary stamp.
5. All content was originally in Spanish (the user's working language for
   most of the conversation) but was translated to English on request,
   since the user already learned grammar terminology in English and mixing
   languages was confusing them. German stays as German throughout (that's
   the subject being taught, and also the source of the bureaucratic-form
   aesthetic/flavor text like "Aktenzeichen", "Formular Nr. 4").

## Monetization plan (for when this becomes an Android app)

Decided direction: **freemium with one-time in-app purchase (IAP)**, not
subscription, not ads. Reasoning: this is a scoped, non-recurring-content
product (grammar doesn't change), so a subscription feels forced and ads
would interrupt the exact learning-flow UX this app was designed around
(the stamp feedback, the two-step case/article flow).

### Free vs. paid split
- **Free tier**: Nominative + Accusative only (covers ~80% of common
  mistakes, per the original "start with articles, not the full table"
  advice that kicked off this whole project). Includes the theory section
  for those two cases and the easy-mode reference table for them.
- **Paid unlock (one-time IAP)**: Dative + Genitive, hard mode, and the
  full reference table (all 4 cases + full preposition lists).

This split is a natural difficulty boundary already baked into how the
content was taught throughout this project (Nom/Akk first, Dativ as "the
one that costs the most because Spanish/English don't mark it", Genitiv
last as "barely used in spoken German"), not an arbitrary paywall.

### Platform requirements to actually ship this
- Google Play Developer account: one-time USD 25 fee.
- Google Play Billing must be used for any in-app purchase sold through
  Play Store — can't process payment outside that system if distributing
  there.
- If wrapped as a TWA (see "Path to Android" below), Google Play Billing
  integration is not automatic — needs explicit wiring between the web
  content and the native billing API (typically via the Digital Goods API
  in the TWA, or a small bridge library).
- Google takes a cut of each sale (historically 15–30% depending on annual
  revenue tier) — factor this into pricing.
- Tax/reporting obligations vary by country of the developer account —
  worth checking local requirements before setting a price.

### Path to Android (recap of options discussed, cheapest first)
1. **PWA** — add `manifest.json` + service worker to the existing
   `index.html`. Installable from Chrome, works offline, no Play Store
   needed. Good first step regardless of monetization plans.
2. **TWA (Trusted Web Activity)** — package the PWA via Android Studio or
   Google's Bubblewrap tool into a real `.apk`/`.aab` for Play Store.
   Minimal changes to the existing HTML/CSS/JS. This is the recommended
   path for this project — best effort-to-result ratio, and where the
   Play Billing integration above would happen.
3. **Capacitor (Ionic)** — heavier wrapper, gives native API access
   (notifications, native storage) if the app grows beyond a static quiz.
   Only worth it if features like true cross-device progress sync get
   built.
4. **Full native rewrite** — not worth it unless the app becomes much more
   complex than a card-based quiz.

### Suggested order of operations
1. Add manifest + service worker (PWA) — cheap, reversible, useful even
   without monetization.
2. Gate Dativ/Genitiv/hard-mode/full-table behind a simple client-side
   flag first (to validate the free/paid split feels right before wiring
   real payments).
3. Wrap as TWA, set up Play Developer account, wire Play Billing to that
   flag.
4. Launch.

## Recently implemented (this session)

- **Bug fix — Definite/Indefinite toggle appeared broken**: the toggle's
  underlying logic was actually correct (verified by simulating clicks in
  jsdom — `artMode` and the button's `active` class updated fine, no JS
  errors). The real problem was UX: unlike the Easy/Hard toggle, which
  visibly hides/shows the reference table on click, the Definite/Indefinite
  toggle only changed an internal variable with zero visible feedback until
  the next time step 2 (article picking) was reached — so it looked dead on
  tap. First fix attempt dimmed the non-matching table (`opacity:0.35`);
  user didn't like that and asked for it to fully hide/show instead, to
  match the Easy/Hard pattern exactly. Final fix: `#defTable` and
  `#indefTable` toggle a `.hide{display:none;}` class on each other
  (mutually exclusive, same mechanism as `#easyTable`'s `.show` class),
  and `nextQuestion()` is called immediately on toggle so the practice card
  reflects the change right away too. Verified with a jsdom click-simulation
  script (not just by reasoning about the code) after the user reported the
  first version "didn't work" — that first debugging pass showed the JS
  itself had no bug at all; the fix was entirely about giving the toggle an
  immediate, unambiguous visual result. General lesson for this project: a
  state toggle with no immediate visible effect reads as broken on a touch
  device, even when the underlying code is correct — pair every toggle with
  something that visibly changes the instant it's tapped, and prefer a hard
  show/hide over a subtle opacity change for consistency with the rest of
  the UI's toggle patterns.

- **Indefinite articles (ein/eine/einer/einem/eines)**: added as a toggle
  inside `mode-spiel` (`#artToggle`, reuses `.diff-btn` styling), alongside
  the existing Easy/Hard toggle. Indefinite forms are derived purely from
  noun gender via a lookup table (`indefiniteForms`), since — unlike the
  definite article — they don't vary noun-by-noun. No plural form exists,
  so plural nouns (`Kinder`) always fall back to definite forms even in
  indefinite mode (`correctArticleFor()` handles this). Added a 5th theory
  card ("Bonus — indefinite articles") and a second compact table (in both
  the Falltabelle mode and the always-visible Easy-mode table) showing
  ein-/eine- forms by case. `articleShiftNote()` generates a dynamic
  explanation line appended after each answer, e.g. "Feminine (indefinite):
  eine → einer."

- **Sentence bank expanded via generation, not hand-writing**: added
  `genPool` (verb/preposition templates for accusative and dative, ~22
  entries total) combined programmatically with the 11 singular nouns in
  `buildGeneratedBank()`, yielding ~240 generated sentences on top of the
  original ~24 hand-written ones (`manualBank`, kept separate — it still
  covers the cases generation can't handle safely: Wechselpräpositionen
  movement/location contrast, genitive noun-ending changes, and dative
  plural noun-ending changes). Final `bank = manualBank.concat(
  buildGeneratedBank())`. English cues and explain strings are also
  templated (`{e}`/{n}` placeholders), not hand-translated per sentence.
  Known minor rough edges: a few generated sentences are semantically odd
  but grammatically valid (e.g. "Ich bin bei dem Buch" / "I'm near the
  book") since templates were combined with all 11 nouns indiscriminately;
  no de-duplication against `manualBank` was done, so a handful of
  generated sentences may exactly repeat a manual one (e.g. "Ich sehe den
  Mann"). Neither issue affects correctness, just occasional slight
  repetition/awkwardness — fine to leave, or worth revisiting if it's
  noticeable during play.

- **Sound effects**: implemented with the Web Audio API (synthesized
  oscillator tones, no external audio files, works fully offline) —
  option 1 from the earlier backlog note. `playTone()` builds a short
  pitch-ramped tone; `playSound(kind)` maps `'correct'` (rising triangle
  wave), `'wrong'` (falling sawtooth), `'partial'` (short triangle, used
  when case XOR article was right), and `'click'` (short square blip,
  used for all button/toggle/next-card interactions) to that. A floating
  mute toggle (`#soundToggle`, 🔊/🔇) sits fixed top-right of the page.
  `soundOn` flag gates everything; `audioCtx` is lazily created on first
  use (required by browser autoplay policies — can't create an
  AudioContext before a user gesture).

## Pending feedback (not yet implemented — backlog for next Claude Code session)



- No persistence: score/streak/history reset on page reload. Nothing is
  saved server-side or in browser storage (intentionally avoided — see
  artifact storage restrictions if this becomes a Claude artifact again;
  no such restriction applies if run as a plain static file, so
  `localStorage` would work fine here if wanted).
- If persistence across devices is wanted, would need a small backend
  (e.g. Node + SQLite/JSON file, served via PM2) with endpoints to save/
  load progress — discussed with the user but not yet built.
- Only 12 nouns and ~24 sentences in the bank — could grow content over
  time, especially genitive examples (currently thin, matching the fact
  that genitive is rare in spoken German per the reference book).
- Could add a harder variant of step 1 where the learner has to *click the
  clue word themselves* in the sentence before picking the case, instead of
  it being highlighted for them after the fact (discussed as "idea 5",
  not implemented).
- Fonts depend on Google Fonts CDN; for fully offline use, would need to
  self-host the font files.

## Hosting

Plan is to self-host on the user's own server, accessed via Tailscale.
No backend needed for the current feature set — any static file server
works (`python3 -m http.server`, nginx, Caddy, etc.). Just serve `index.html`
directly.
