# CLAUDE.md

## Purpose
Personal webpage and book recommendation list ("Books That Shaped My Thinking"). It shows a short bio, social links, a rotating quote, "Currently Reading" cards, and three tabs:
- **Timeless Recommendations**: rated book cards that can be filtered by tag.
- **Pipeline**: books queued to read.
- **Horus Heresy**: a reading-path timeline with read/reading/unread/optional books split into parts, generated from a JS array.

## Design direction
"Modern Bento + Timeline" (Proposal C). A product-style dashboard look: rounded tiles (`.tile`) with soft borders and shadows, Space Grotesk for display text, Inter for body text, and one vivid violet accent (`--accent`). Green means "read" and amber means "optional"; both are used only in the Heresy timeline. Dark is the default theme; `body.light` overrides the tokens.

The Horus Heresy tab is the one exception: it uses a scoped grimdark treatment inspired by design B (Cinzel headings, gold on dark parchment, lozenge-ended `.heresy-divider` lines, a red-and-gold framed Siege of Terra segment). It lives in the "HERESY GOTHIC" CSS block, where `#panel-heresy` re-points `--accent`, `--surface`, `--border` and friends to `--heresy-*` values, with light-theme overrides under `body.light #panel-heresy`. Nothing outside `#panel-heresy` changes; the Heresy progress bar (`.heresy-progress-bar`) is split from the bento `.stat-bar` so each styles separately.

## Stack
- One static HTML file. CSS and vanilla JavaScript are inline. There is no framework, no build step, no package manager, and no dependencies.
- Fonts load from Google Fonts (Inter, Space Grotesk, plus Cinzel for the Heresy tab).
- Horus Heresy and Siege of Terra covers, plus Ruinstorm and Reframe Your Brain, are local files in `covers/` (JPEG, max 480px tall, under 150KB). Other covers are still hotlinked from Goodreads, Open Library, and Amazon; the avatar is on imgur. Card links point to Amazon.
- The page reads and writes only one `localStorage` key, `theme` (`light`/`dark`).

## Structure
```
README.md    # one-line description
index.html   # the whole site
covers/      # local cover images (heresy-NN-*.jpg, siege-NN-*.jpg, others by slug)
```
Where things are in `index.html`:
- `<head><style>`: design tokens in `:root` and `body.light`, then the component styles in page order (bento tiles, section headers, Currently Reading, stat tile, tabs, filter chips, book grid, Pipeline, Heresy header/legend, Heresy timeline, chrome), then the `900px` and `600px` media queries.
- `<head><script>`: `coverFb`, the image fallback, so it exists before any `onerror` fires.
- Top of `<body>`: a one-line script that applies the saved light theme before first paint.
- `<main>` starts with `<section class="bento">`, a 12-column grid of tiles:
  - `header.tile-profile`: theme toggle, avatar, title, tagline, bio.
  - `.tile-links`: the four link buttons.
  - `.tile-quote`: the quote rotator.
  - `.tile-reading`: the "Currently Reading" heading and `.reading-grid`.
  - `button#heresyStat.tile-stat`: Heresy progress, filled in by JS; clicking it calls `openHeresy()`, which switches to the Heresy tab.
- Then the tab nav (`#tabNav`, buttons with `data-tab="panel-..."`) and the panels `#panel-timeless` (with `#filterBar` and `.recs-grid`), `#panel-pipeline`, and `#panel-heresy`.
- `#panel-heresy` holds only static shell markup: the title block, the progress block (`#hpNum`, `#hpBar`, `#hpFill`, `#hpSub`), the legend (`.tl-legend`), and an empty `#heresyTimeline`.
- `<footer>`: a "Last updated" date you change by hand.
- `<script>` at the end of the body, in order: the `HERESY` array, `switchTab`, the tag filter (uses the `data-tags` attribute, space-separated, on each card), the `quotes` array and rotator, `toggleTheme`, the scroll-to-top button, the computed counts, `renderHeresy()`, and `openHeresy()`.

### Adding a Timeless, Pipeline or Currently Reading book
Copy an existing `<a class="card" data-tags="...">` block (or `.card-mini` / `.card-featured`). Each tag must match a `data-tag` on a `.filter-btn`. The "N books" pills update themselves.

### The Heresy tracker
All of it comes from the `HERESY` array at the top of the main script. Each entry is one book:
```js
{ arc: 'main', part: 'Part III — Imperium Secundus', title: 'Ruinstorm', author: 'David Annandale',
  cover: 'covers/heresy-11-ruinstorm.jpg', status: 'reading', optional: false, rating: null }
```
- `arc`: `'main'` (counted in the main-arc total) or `'siege'` (Siege of Terra, drawn as the final full-width segment).
- `part`: the segment label. Consecutive books with the same `part` form one segment, so array order is reading order.
- `status`: `'read'`, `'reading'` or `'unread'`.
- `optional: true` draws the book on a dashed branch off the main track.
- `rating`: shown for read books (`'9/10'`, or `'Read'` when unrated).

To mark a book finished, set `status: 'read'` and a `rating`, then set the next book to `status: 'reading'`. To add a book, add an entry and put its cover in `covers/`.

`renderHeresy()` computes the rest:
- "Main arc X of Y read", the progress bar, and "Reading now: … (#N of Y) · Siege of Terra: X of Y".
- The per-segment read counts (e.g. `3/3`) and how far the track is filled (up to and halfway into the current book).
- The numbers in the bento "Heresy progress" tile.

The timeline is horizontal above 900px (segments wrap onto rows) and vertical at 900px and below.

The intro text of the Ruinstorm card in Currently Reading ("Book 11 of 18…") and the Heresy subtitle ("18 main-arc books… 8-book Siege of Terra") are plain prose and are not computed.

## How to run locally
Open the file directly (`xdg-open index.html`), or serve the folder:
```
python3 -m http.server 8000   # then visit http://localhost:8000
```

## How to test
There are no automated tests, linters, or CI. Check changes by hand in a browser:
- Each tab switches correctly, and the filter bar shows only on Timeless.
- Each tag filter shows and hides the right cards.
- The theme toggle works and persists after a reload.
- The quote rotator and its dots work.
- Broken cover images fall back properly.
- The layout holds at 1280px, at the 900px and 600px breakpoints, and at 390px, with no horizontal scroll.
- The bento "Heresy progress" tile opens the Heresy tab, and its numbers match the Heresy tab's progress block.
- The Heresy timeline shows all 26 covers, is horizontal above 900px and vertical below, and the track is filled up to the current book.
- There are no errors in the browser console.

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
- Cover-image fallbacks are inconsistent in the hand-written cards. Timeless cards and the Heresy timeline use `coverFb`, Pipeline cards use `this.style.display='none'`, and Currently Reading cards set `innerHTML` inline. Consider using `coverFb` everywhere.
- Hotlinked covers from external hosts can break or be rate-limited. Heresy covers are local now; the rest could be moved into `covers/` too.
- Bold Conjectures has no cover (no reliable source found), so it shows the text fallback.
- Typo: the README misspells "reccomendations".
- The Pipeline grid has one stray `</a>` just before it closes. Browsers ignore it.
- The Heresy timeline needs JavaScript; with scripts disabled the tab shows only its heading and legend.
- Recent commit messages ("Update index.html", "Hello → Goodbye") don't describe the changes. Use descriptive messages.
