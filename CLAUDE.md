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
- `archive/` — plain markdown, one file per past session (e.g.
  `week-01-wed.md`), kept outside the app itself. See "Archiving past
  sessions" below. Not read by `index.html` at runtime — purely a durable,
  git-tracked record for the coach (and for Claude, in future sessions) to
  look back at.

## Content model

All content lives in a single `CONTENT` object near the top of the `<script>`
block, keyed by language, then by day:

```js
const CONTENT = {
  en: {
    wed: { tipLabel, eyebrow, title, focus, sections: [...], tip },
    thu: { /* same shape */ }
  },
  sv: { /* same shape as en, Swedish */ }
};
```

Only the **current week's** Wednesday and Thursday sessions live in
`CONTENT` — this stays small and fast to load. Everything before the current
week belongs in `archive/`, not in `CONTENT`.

A day with nothing planned yet uses `sections: []` and a `focus` note saying
so (see the `thu` placeholder) — the UI shows an empty state and disables the
swipe-view icon automatically when `sections` is empty.

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

A day toggle (WED / THU) sits next to it. On load it defaults to today's
session — Thursday shows Thursday's plan, every other day of the week shows
Wednesday's (either today's if it's Wednesday, or the next upcoming one
otherwise) — via `defaultDay()` in the script. The coach can still tap the
other day to preview it ahead of time; switching days re-renders instantly
like the language toggle does, no reload.

## Visual design

- Palette: matches Fryshuset Basket's actual club colors — black and white,
  with red as the sole accent (`--red`/`--red-deep`/`--red-tint`). Light
  neutral background (`--bg`), white card surfaces (`--card`), near-black ink
  text (`--ink`). Deliberately switched from an earlier orange/wood-floor
  placeholder theme to this once the real club branding was confirmed
  (Wikipedia: Fryshuset Basket team colors are red/white; fryshusetbasket.se
  uses a minimalist black-and-white site with a bold black display font).
  Don't reintroduce orange or the wood-plank background.
- Type: `Anton` (bold uppercase display face) for headings/titles — chosen to
  echo the bold black condensed/extended headline font Fryshuset Basket uses
  on their own site — `Space Mono` for duration/scoreboard-style numerals,
  `Inter` for body text.
- Mobile-first, max content width ~480px, generous tap targets, safe-area
  insets respected for notched phones.

## Updating for a new day's or week's session

This is the main ongoing task. When given a new session plan (usually as a
markdown file like `session-2-*.md`), first check whether it's replacing a
day that's already in `CONTENT` (i.e. moving to a new week) or filling in a
day that's currently a placeholder (e.g. this week's `thu` still says "TBD").

**If it's replacing content already in `CONTENT`** (rolling into a new
week): first archive the outgoing session — see "Archiving sessions" below
— then overwrite it.

**Either way, to write the new session:**

1. Translate its structure into the `sections` array for **both** `en` and
   `sv`, under the correct day (`wed` or `thu`) in `CONTENT`, following the
   body-text conventions above.
2. Update `eyebrow`, `title`, `focus`, and `tip` for both languages.
3. Keep section `id`s stable where a drill repeats week to week if it helps,
   but it's fine to regenerate them per session — nothing else in the app
   depends on specific id values persisting across sessions.
4. Double-check total `duration` across sections lines up with the intended
   session length (usually 60 min).
5. **Archive this same session** to `archive/week-NN-<day>.md` — see
   "Archiving sessions" below. Do this immediately, every time content is
   added or changed in `CONTENT`, not only when a week is about to be
   overwritten — a day filled in from a placeholder (e.g. `thu` going from
   "TBD" to real content) needs an archive entry created now just as much as
   a day being replaced needs one saved before it's lost. There's no
   in-session state to rely on to catch this later — the archive step lives
   in this checklist so it isn't missed.
6. Commit and push directly to `main` — GitHub Pages serves straight from the
   repo root on that branch, so a push is the entire deploy step. No build,
   no PR required for this solo-maintained repo unless asked otherwise.

## Archiving sessions

Every session that goes into `CONTENT` — new or replacing an old one — gets
saved to `archive/week-NN-<day>.md` (e.g. `archive/week-01-wed.md`) as plain
markdown, both languages, readable on its own without the app. Use the
existing archive files as the template. This keeps `CONTENT` limited to the
current week (fast, small `index.html`) while still preserving every session
in the repo, versioned by git — do it as part of writing the session, not as
an afterthought only triggered by an overwrite.

## Explicitly out of scope unless asked

- Persistent storage, accounts, or login of any kind.
- A "mark done" / checklist / progress-tracking feature.
- Multi-session history/archive inside the *app* itself — past sessions live
  as markdown in `archive/`, not in `CONTENT` or in any in-app browsing UI,
  deliberately, to keep the live app simple and fast.
- A build step, framework, or dependency — this stays a single static HTML
  file on purpose.
