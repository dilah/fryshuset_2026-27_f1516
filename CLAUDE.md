# Fryshuset Practice Plan App

A single-page, static, bilingual (EN/SV) web app that displays the weekly practice
plan for a youth girls' basketball team. Hosted on GitHub Pages. No backend, no
accounts, no database — the whole app is one `index.html` file.

## Who this is for

Dimitris coaches a youth girls' basketball team (born 2015–2016, ages 9–11,
mixed skill levels, 10–16 players per session) at Fryshuset, a community club
in Sweden. He is primarily a solo coach with two assistant coaches whose
attendance is inconsistent. The team trains twice a week:

- **Wednesday — Teach/Train day**: structured, better use of assistant coaches
  when present.
- **Thursday — Play/Apply day**: game-like, more manageable solo.

These two days are *not* interchangeable — Wednesday introduces things,
Thursday applies them. Keep that distinction in mind if asked to restructure
anything.

The season is built around team culture (trust, fun, mutual respect),
age-appropriate skill development, and SBBF (Swedish Basketball Federation)
methodology, with a trimmed BasketSmart warm-up and a Constraints-Led Approach
(CLA) integrated in tension with a more instruction-first retention approach.
The team had no structured summer training, so the season opens with a
multi-week reconditioning phase (movement re-entry) before full intensity.

## Purpose of this app

This app is read *live, on a phone, mid-practice* — by the coach and by
assistant coaches who may not be technical and should never need to log in to
anything. Every design decision should protect that:

- **No accounts, ever.** No login walls, no auth, for anyone.
- **No persistent storage / no backend.** This was tried (via Claude
  artifacts with `window.storage`) and abandoned — it required viewers to be
  signed into a Claude account just to load the page, which broke sharing
  with assistants. Do not reintroduce any storage, database, or
  authentication layer. All content lives directly in the HTML/JS as a plain
  JS object.
- **One stable URL.** The whole point of moving to GitHub Pages was a
  permanent link (`https://dilah.github.io/fryshuset_2026-27_f1516/`) that
  never changes, added to home screens on iOS/Android. Don't do anything that
  would change the URL (e.g. renaming `index.html`, moving it out of the repo
  root, changing the repo name).
- **Fast and legible mid-session.** Large tap targets, high contrast,
  readable at a glance in a gym.

## File structure

- `index.html` — the entire app. Self-contained: HTML + CSS + JS in one file,
  Google Fonts loaded via `<link>`, no build step, no dependencies to install.
  This is intentional — it needs to stay a single file you can drag-and-drop
  upload to GitHub's web UI if needed.

## Content model

All content lives in a single `CONTENT` object near the top of the `<script>`
block, keyed by language:

```js
const CONTENT = {
  en: { tipLabel, eyebrow, title, focus, sections: [...], tip },
  sv: { /* same shape, Swedish */ }
};
```

Each item in `sections` is:

```js
{ id: "s1", title: "...", duration: 10, body: "..." }
```

`body` is plain text with a lightweight convention (parsed by `renderBody()`):

- Blank line (`\n\n`) = new paragraph/block.
- A block where every line starts with `- ` renders as a bullet list.
- A line starting with `Say:` (English) or `Säg:` (Swedish) is pulled out and
  rendered as a highlighted spoken-cue callout, not a regular paragraph.

### Bilingual convention

- The two language objects should have **matching `id` values** per section,
  in the same order, so switching languages mid-view (list or swipe) doesn't
  jump around.
- Direct-address coaching phrases (things the coach literally says to the
  players, e.g. `"Håll koll på bollen!"`) are **already in Swedish** in the
  English version too, and stay identical in both languages — that's what the
  coach actually says out loud regardless of which language the surrounding
  text is in. Only the descriptive/explanatory text around them gets
  translated.
- Swedish basketball terms that don't have a natural English equivalent
  (Bolltjuven, Plankan, Anfall, Försvar, etc.) are used as-is in both
  languages.

## Views

The app has exactly two views, both purely for *reading* — there is
deliberately no checklist, no "mark done," no progress tracking of any kind.
That was tried and explicitly removed at the coach's request. Don't
reintroduce it unless asked.

1. **List view** (default): all sections shown as an accordion — tap a card
   to expand/collapse it (one open at a time). Vertical "sideline" dotted
   line down the left edge is a deliberate basketball-play-diagram visual
   motif; keep it if restyling.
2. **Swipe view** (▶ icon, top right): full-screen, one section per screen,
   swipe or tap arrows to move between them, dot indicator at the bottom.
   Opened/closed via the ▶ / ✕ icons.

A language toggle (EN / SV) sits in the top bar and re-renders whichever view
is currently active without resetting swipe position.

## Visual design

- Palette: warm hardwood-floor tan background (`--wood`), cream card surfaces
  (`--card`), deep navy ink text (`--ink`), basketball orange accent
  (`--orange`). Not the default Claude terracotta — keep it distinct.
- Type: `Archivo Black` for headings/titles, `Space Mono` for
  duration/scoreboard-style numerals, `Inter` for body text.
- Mobile-first, max content width ~480px, generous tap targets, safe-area
  insets respected for notched phones.

## Updating for a new week's session

This is the main ongoing task. When given a new session plan (usually as a
markdown file like `session-2-*.md`):

1. Translate its structure into the `sections` array for **both** `en` and
   `sv` in `CONTENT`, following the body-text conventions above.
2. Update `eyebrow`, `title`, `focus`, and `tip` for both languages.
3. Keep section `id`s stable where a drill repeats week to week if it helps,
   but it's fine to regenerate them per session — nothing else in the app
   depends on specific id values persisting across sessions.
4. Double-check total `duration` across sections lines up with the intended
   session length (usually 60 min).
5. Commit and push directly to `main` — GitHub Pages serves straight from the
   repo root on that branch, so a push is the entire deploy step. No build,
   no PR required for this solo-maintained repo unless asked otherwise.

## Explicitly out of scope unless asked

- Persistent storage, accounts, or login of any kind.
- A "mark done" / checklist / progress-tracking feature.
- Multi-session history/archive inside the app itself (the coach keeps that
  in Claude chat history and project files, deliberately, to keep this app
  simple).
- A build step, framework, or dependency — this stays a single static HTML
  file on purpose.
