# CLAUDE.md

## Purpose
Personal webpage and book recommendation list ("Books That Shaped My Thinking"). It shows a short bio, social links, a rotating quote, "Currently Reading" cards, and three tabs:
- **Timeless Recommendations**: rated book cards that can be filtered by tag.
- **Pipeline**: books queued to read.
- **Horus Heresy**: a reading-path tracker with read/reading/unread/optional books split into parts, generated from a JS array.

## Design direction: "Grimdark Gothic"
- Dark, atmospheric, Warhammer 40k / Horus Heresy flavour. Dark is the default theme; `body.light` is a parchment/vellum variant.
- Near-black and charcoal surfaces, aged gold/brass (`--gold*`) and blood-red (`--red*`) accents, square corners, gilded cover frames, diamond ornaments. Textures are CSS gradients only (no image files).
- Fonts: Cinzel (headings, labels, buttons), EB Garamond (body), UnifrakturCook (only the quote mark).
- Cinzel has no lowercase, so labels look like capitals without `text-transform`. Only use `text-transform:uppercase` where it is already used.
- The Heresy tab is the showpiece: Imperial dividers per part, gilded frames with a position plate, lit read books, dimmed unread books, a glowing current book, a parchment "Optional" seal, a notched gilded gauge, and the Siege of Terra as a framed red finale with its own gauge.

## Stack
- One static HTML file. CSS and vanilla JavaScript are inline. There is no framework, no build step, no package manager, and no dependencies.
- Fonts load from Google Fonts (Cinzel, EB Garamond, UnifrakturCook).
- Horus Heresy and Siege of Terra covers, plus Ruinstorm and Reframe Your Brain, are local files in `covers/` (JPEG, max 480px tall, under 150KB). Other covers are still hotlinked from Goodreads, Open Library, and Amazon; the avatar is on imgur. Card links point to Amazon.
- The page reads and writes only one `localStorage` key, `theme` (`light`/`dark`).

## Structure
```
README.md    # one-line description
index.html   # the whole site
covers/      # local cover images (heresy-NN-*.jpg, siege-NN-*.jpg, others by slug)
```
Where things are in `index.html`:
- `<head><style>`: design tokens in `:root` (dark is the default) and `body.light` overrides, then the component styles (header, cards, tabs, filters, Heresy tracker, responsive breakpoints at 900/700/600px).
- `<head><script>`: `coverFb` plus one document-level `error` listener that swaps any broken cover inside `.cover-wrap` or `.heresy-cover` for a text plate (it uses the image `alt`).
- Top of `<body>`: a tiny script that applies the saved light theme before first paint.
- `<header>`: avatar, title, tagline.
- `<main>`: link buttons, bio, quote rotator, Currently Reading (`.reading-grid`), tab nav, then the panels `#panel-timeless` (with `#filterBar` and `.recs-grid`), `#panel-pipeline`, and `#panel-heresy`.
- `#panel-heresy` holds only a static shell: title, subtitle, empty progress elements (`#heresyNum`, `#heresyBar`, `#heresySub`), the legend, and an empty `#heresyTree`.
- `<footer>`: a "Last updated" date you change by hand.
- `<script>` at the end of the body, in this order: the `heresyBooks` array (first thing in the script), `renderHeresy`, the section counts, `switchTab`, the tag filter (uses the `data-tags` attribute, space-separated, on each card), the `quotes` array and rotator, `toggleTheme`, and the scroll-to-top button.

### Timeless / Pipeline / Currently Reading
To add a book, copy an existing `<a class="card" data-tags="...">` block. Each tag must match a `data-tag` on a `.filter-btn`.

### Horus Heresy tracker
The tracker is generated from the `heresyBooks` array at the top of the last `<script>`. One object per book, in reading order:
```js
{ part: 'Part III — Imperium Secundus', title: 'Ruinstorm', author: 'David Annandale',
  cover: 'covers/heresy-11-ruinstorm.jpg', status: 'reading', optional: false, rating: null }
```
- `status`: `'read'`, `'reading'`, or `'unread'`. `optional: true` adds the "Optional" seal. `rating`: `'9/10'` or `'Read'` for finished books, otherwise `null`.
- To mark a book as finished, set `status: 'read'` and a `rating`, then set the next book to `status: 'reading'`.
- To add a book, insert an object at the right position and put its cover in `covers/`. A new `part` string starts a new section.
- Books whose `part` equals `SIEGE_PART` (`'⚔ The Siege of Terra'`) count towards the Siege total. All other books count towards the main arc.

Computed automatically (do not hardcode):
- "Main arc N of M read", the gauge fill and notches, and the pulsing notch for the current book.
- The "Reading now: Title (#position of total) · Siege of Terra: N of M" line.
- The position plate on each cover, the read tally on each part divider, and the Siege gauge.
- The "2 books" and "30 books" counts next to Currently Reading and Timeless Recommendations (counted from the cards on the page).

Layout: parts with more than 4 books and the Siege span the full width. Other parts sit in a two-column grid, and in one column at 900px and below.

## How to run locally
Open the file directly (`xdg-open index.html`), or serve the folder:
```
python3 -m http.server 8000   # then visit http://localhost:8000
```

## How to test
There are no automated tests, linters, or CI. Check changes by hand in a browser:
- Each tab switches correctly, and the filter bar shows only on Timeless.
- Each tag filter shows and hides the right cards.
- The Heresy tab shows 26 covers (18 main arc, 8 Siege), and the counters match the `heresyBooks` array.
- The theme toggle works and persists after a reload, with no flash of the dark theme.
- The quote rotator and its dots work.
- Broken cover images fall back properly.
- The layout holds at 1280px, at the 900px, 700px (tabs become a grid) and 600px breakpoints, and at 390px, with no horizontal scrolling.

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
- The Ruinstorm card in Currently Reading still says "Book 11 of 18" in its description text. Update it by hand when the current Heresy book changes.
- The footer "Last updated" date is set by hand.
- The card markup still has mixed inline `onerror` handlers (`coverFb(...)`, `this.style.display='none'`, inline `innerHTML`). They are harmless: the `<head>` listener handles every cover in `.cover-wrap` and `.heresy-cover`. The two Currently Reading cards (`.cover`) use their own inline fallback.
- Hotlinked covers from external hosts can break or be rate-limited. Heresy covers are local now; the rest could be moved into `covers/` too.
- Bold Conjectures has no cover (no reliable source found), so it shows the text fallback.
- The Heresy tracker needs JavaScript; without it the tab shows only the title and legend.
- Typo: the README misspells "reccomendations".
- Recent commit messages ("Update index.html", "Hello → Goodbye") don't describe the changes. Use descriptive messages.
