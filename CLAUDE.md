# CLAUDE.md

## Purpose
Personal webpage and book recommendation list ("Books That Shaped My Thinking"). It shows a short bio, social links, a rotating quote, "Currently Reading" cards, and three tabs:
- **Timeless Recommendations**: rated book cards that can be filtered by tag.
- **Pipeline**: books queued to read.
- **Horus Heresy**: a reading-path tracker with read/unread/optional books split into parts.

## Stack
- One static HTML file. CSS and vanilla JavaScript are inline. There is no framework, no build step, no package manager, and no dependencies.
- Fonts load from Google Fonts (Fraunces, Inter, Space Grotesk).
- Horus Heresy and Siege of Terra covers, plus Ruinstorm and Reframe Your Brain, are local files in `covers/` (JPEG, max 480px tall, under 150KB). Other covers are still hotlinked from Goodreads, Open Library, and Amazon; the avatar is on imgur. Card links point to Amazon.
- The page reads and writes only one `localStorage` key, `theme` (`light`/`dark`).

## Structure
```
README.md    # one-line description
index.html   # the whole site
covers/      # local cover images (heresy-NN-*.jpg, siege-NN-*.jpg, others by slug)
```
Where things are in `index.html`:
- `<head><style>`: design tokens in `:root` (dark is the default) and `body.light` overrides, then the component styles (header, cards, tabs, filters, Heresy tracker, responsive breakpoints).
- `<header>`: avatar, title, tagline.
- `<main>`: link buttons, bio, quote rotator, Currently Reading (`.reading-grid`), tab nav, then the panels `#panel-timeless` (with `#filterBar` and `.recs-grid`), `#panel-pipeline`, and `#panel-heresy`.
- `<footer>`: a "Last updated" date you change by hand.
- `<script>` at the end of the body: `switchTab`, the tag filter (uses the `data-tags` attribute, space-separated, on each card), the `quotes` array and rotator, `toggleTheme`, the scroll-to-top button, and the `coverFb` image fallback.

To add a book, copy an existing `<a class="card" data-tags="...">` block. Each tag must match a `data-tag` on a `.filter-btn`.

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
- The layout holds at the 900px and 600px breakpoints.

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
- Section counts are hardcoded (e.g. "2 books", "30 books") and must be updated by hand when books change.
- The footer "Last updated" date is set by hand.
- Cover-image fallbacks are inconsistent. Some covers use `coverFb(this, title)`, some use `this.style.display='none'`, and some set `innerHTML` inline. Consider using `coverFb` everywhere.
- Hotlinked covers from external hosts can break or be rate-limited. Heresy covers are local now; the rest could be moved into `covers/` too.
- Bold Conjectures has no cover (no reliable source found), so it shows the text fallback.
- `coverFb` is defined in a small `<head>` script so it exists before any image `onerror` fires.
- The saved light theme is applied only after the page renders, because the script is at the end of the body. Light-mode users can see a brief flash of the dark theme.
- Typo: the README misspells "reccomendations". (The Heresy "Imperium Secundus" label is fixed.)
- Most of the Heresy tracker markup repeats per book (with odd leftover indentation), which suggests it was generated elsewhere. Edit it carefully, or generate it from a JS array.
- Recent commit messages ("Update index.html", "Hello → Goodbye") don't describe the changes. Use descriptive messages.
