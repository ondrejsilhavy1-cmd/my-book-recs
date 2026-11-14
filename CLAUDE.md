# CLAUDE.md

## Purpose
Personal webpage and book recommendation list ("Books That Shaped My Thinking"). It shows a short bio, social links, a rotating quote, "Currently Reading" cards, and three tabs:
- **Timeless Recommendations**: rated book cards that can be filtered by tag.
- **Pipeline**: books queued to read.
- **Horus Heresy**: a reading-path tracker with read/reading/unread/optional books split into parts, generated from a JS array.

## Design direction: "Editorial Light"
A calm, literary-magazine look. Keep new work in this language:
- Light is the default theme: warm off-white paper (`--bg`), near-black ink, thin rules, and one accent colour (`--accent`, oxblood). Dark is available through the toggle.
- Fraunces (serif) for headings, titles, quotes and bio; Inter for body text and the small-caps labels (uppercase, letter-spaced, `.7rem`).
- No boxes where a hairline rule will do. Covers are shown like books on a shelf: 2:3 aspect ratio, soft shadow, a faint spine sheen.
- Status is shown with understated dots: filled = read, half = reading, hollow = unread. Optional books get a dashed outline and an italic "Optional" label.

## Stack
- One static HTML file. CSS and vanilla JavaScript are inline. There is no framework, no build step, no package manager, and no dependencies.
- Fonts load from Google Fonts (Fraunces, Inter).
- Horus Heresy and Siege of Terra covers, plus Ruinstorm and Reframe Your Brain, are local files in `covers/` (JPEG, max 480px tall, under 150KB). Other covers are still hotlinked from Goodreads, Open Library, and Amazon; the avatar is on imgur. Card links point to Amazon.
- The page reads and writes only one `localStorage` key, `theme` (`light`/`dark`). Light is the default; `dark` adds the class `dark` to `<html>`.

## Structure
```
README.md    # one-line description
index.html   # the whole site
covers/      # local cover images (heresy-NN-*.jpg, siege-NN-*.jpg, others by slug)
```
Where things are in `index.html`:
- `<head><style>`: design tokens in `:root` (light is the default) and `html.dark` overrides, then the component styles (masthead, quote, covers, cards, tabs, filters, Heresy tracker, fixed buttons, responsive breakpoints at 900px and 600px).
- `<head>` scripts: `coverFb`, then a small script that applies the saved theme before first paint and registers a catch-all image `error` listener (any failed cover inside `.cover-wrap`, `.cover` or `.heresy-cover` is replaced by a `.cover-fb` text block using the image's `alt`).
- `<header>`: the masthead. Left: title and tagline. Right: avatar, link buttons (`.bio-row`), bio (`.bio-block`).
- `<main>`: quote rotator, Currently Reading (`.reading-grid`), tab nav, then the panels `#panel-timeless` (with `#filterBar` and `.recs-grid`), `#panel-pipeline`, and `#panel-heresy`.
- `#panel-heresy` holds only the static heading, subtitle, legend, and two empty containers (`#heresyProgress`, `#heresyTree`) that the script fills.
- `<footer>`: a "Last updated" date you change by hand.
- `<script>` at the end of the body, in this order: the `HERESY` array, `renderHeresy`, the section counts, `switchTab`, the tag filter (uses the `data-tags` attribute, space-separated, on each card), the `quotes` array and rotator, `toggleTheme`, and the scroll-to-top button.

### Adding a Timeless / Pipeline / Currently Reading book
Copy an existing `<a class="card" data-tags="...">` (or `.card-mini`, `.card-featured`) block. Each tag must match a `data-tag` on a `.filter-btn`. The "N books" counts update on their own.

### Horus Heresy tracker
All Heresy data lives in the `HERESY` array at the top of the body `<script>`. One object per book, in reading order:
```js
{ arc: 'main' | 'siege', part: 'Part I — The Fall of Horus', title, author,
  cover: 'covers/heresy-01-horus-rising.jpg',
  status: 'read' | 'reading' | 'unread', optional: false, rating: '9/10' | 'Read' | null }
```
- **Mark a book as read**: set `status: 'read'` and a `rating` (`'8/10'`, or `'Read'` if unrated). Then set the next book to `status: 'reading'`.
- **Add a book**: add an object at the right position and put its cover in `covers/`. Consecutive books with the same `part` form one group.
- **Layout rule**: the first main-arc part and the Siege render as full-width "shelves"; the other main-arc parts render side by side as compact lists (4 columns, 2 at 900px, 1 at 600px).
- **Computed automatically**: book numbers within each arc, the "X of N read" counters, the segmented progress lines (one segment per book), the per-part "read/total" counts, and the "Reading now: Title (#n of N)" line.
- **Not computed**: the subtitle sentence above the tracker ("18 main-arc books … 8-book Siege of Terra") and the Ruinstorm card text in Currently Reading ("Book 11 of 18 …") are static HTML. Update them by hand if the totals or the current book change.

## How to run locally
Open the file directly (`xdg-open index.html`), or serve the folder:
```
python3 -m http.server 8000   # then visit http://localhost:8000
```

## How to test
There are no automated tests, linters, or CI. Check changes by hand in a browser:
- Each tab switches correctly, and the filter bar shows only on Timeless.
- Each tag filter shows and hides the right cards.
- The theme toggle works, persists after a reload, and does not flash the other theme on load.
- The quote rotator and its dots work.
- Broken cover images fall back to a text block.
- The Heresy tab shows 18 main-arc and 8 Siege books, and its counters match the `HERESY` array.
- The layout holds at 1280px, at the 900px and 600px breakpoints, and at 390px, with no horizontal scrolling.
- The browser console shows no errors.

## Deployment
- The default branch is `main`.
- The repo is a static site: `README.md`, `index.html`, and the `covers/` folder.
- There are no GitHub Actions workflows and no `vercel.json`. There is also no wrangler, railway, or Dockerfile config.
- The GitHub repo homepage is https://my-book-recs.vercel.app
- GitHub Deployments show `vercel[bot]` creating a Production deployment for every commit on `main`. Deploys happen automatically through the Vercel GitHub integration, and Vercel serves `index.html` as a static file with zero configuration.
- To deploy, merge or push to `main`. Pushes to other branches may get Vercel preview deployments; this has not been verified.

## Environment variables
None. There is no `.env.example`, and the code has no `process.env`/`import.meta.env` references or server-side code.

## Known issues / TODOs
- The footer "Last updated" date is set by hand.
- The Heresy subtitle and the Ruinstorm "Book 11 of 18" card text are static (see above).
- The card blocks still carry three different inline `onerror` handlers (`coverFb(this, title)`, `this.style.display='none'`, inline `innerHTML`). The catch-all listener in `<head>` makes them all end in the same text fallback, so this is cosmetic; the inline handlers could be removed.
- Hotlinked covers from external hosts can break or be rate-limited. Heresy covers are local; the rest could be moved into `covers/` too.
- Bold Conjectures has no cover (no reliable source found), so it shows the text fallback.
- Typo: the README misspells "reccomendations".
- Older commit messages ("Update index.html", "Hello → Goodbye") don't describe the changes. Use descriptive messages.
