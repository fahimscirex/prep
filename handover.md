# Handover — MKT4101 study guide (HTML artifact)

**Scope of this document:** the single-file HTML/CSS deliverable only — its architecture,
design system, component inventory, constraints and gotchas. The study *content* (services
marketing theory) is settled and out of scope; do not rewrite it.

**Your job (next agent):** put `MKT4101-midterm-prep.html` into a git repo and publish it on
GitHub Pages.

---

## 1. What this is

A self-contained exam revision guide for a university services-marketing course (MKT4101).
One HTML file, ~104 KB, **zero JavaScript** apart from the print button's one-line `onclick="print()"`, one external request (Google Fonts). It is a
personal study document for one student — not a product, not multi-user, no analytics, no
backend. Nothing is live yet.

It was built by extracting text from the course books/slides and OCR'ing photocopied pages
that carry the teacher's handwritten priority marks. Those source files live beside it
(`books/`, `slides/`, `suggestions/`) and are **inputs only** — see §10 before committing them.

---

## 2. Files

```
/home/fahim/playground/mid/MKT4101/
├── MKT4101-midterm-prep.html   ← THE DELIVERABLE. Everything is inline.
├── handover.md                 ← this file
├── books/         CH0102_Zeithaml.pdf, CH02_Lovelock.pdf        (4.5 MB, source)
├── slides/        3 × .pptx                                      (8.9 MB, source)
├── suggestions/   lovelock_1.pdf, lovelock_2.pdf                (13 MB, source, scanned)
└── .claude/settings.local.json
```

`.html` is standalone: `<!doctype html>`, `<head>` with charset + viewport, all CSS in one
`<style>` block. No build step, no bundler, no dependencies to install.

---

## 3. Git state

**There is no git repo yet.** `git rev-parse --is-inside-work-tree` → *fatal: not a git
repository*. Nothing is committed, staged, or pushed. You are starting clean.

---

## 4. Deploying to GitHub Pages — the immediate task

GitHub Pages serves `index.html` at the directory root. The file is currently named
`MKT4101-midterm-prep.html`. Two options:

```bash
# preferred: keep the descriptive name as the source of truth, publish a copy
cp MKT4101-midterm-prep.html index.html
```
or rename outright. Either is fine — **the filename is not referenced anywhere inside the
file**, and the `<title>` (`MKT4101 Services Marketing`) carries the identity. If you copy
rather than rename, note that you now have two files to keep in sync; renaming is cleaner
unless the user wants the descriptive name kept.

Things that are already correct for Pages and need no change:
- Relative/no asset paths — nothing to break under a `/repo-name/` subpath.
- No `<base>` tag, no absolute local paths.
- Fonts load over `https://` from Google Fonts, so no mixed-content warning.
- No JS except the inline `onclick="print()"` on the print button; a strict CSP would block that one handler and the button would silently do nothing.

Add a `.nojekyll` file at the repo root. Not strictly required here (no underscore-prefixed
paths), but it removes an entire class of surprise from Jekyll preprocessing and costs nothing.

**Do not** add a build action. There is nothing to build. Point Pages at the branch root.

---

## 5. Architecture of the file

```
<head>  charset · viewport · title · meta description · color-scheme
        Google Fonts <link>  (Newsreader + Geist Mono)
        <style>  ~1100 lines, sectioned with ═══ banner comments
<body>
  a.skip                              skip-to-content link
  div.app                             CSS grid: 252px rail | 1fr pane  (≥1040px)
    aside.rail#menu                   wordmark + nav; becomes a drawer <1040px
    div.pane
      header.topbar                   breadcrumb + theme button; sticky on mobile
      div.scrollarea                  the scroll container on desktop
        div.inner                     centred, max-width = measure + 11rem
          header.mast                 title block
          main.doc#doc                5 <section>s + <footer>
```

On desktop the **pane scrolls, not the body** (`.scrollarea { overflow-y: auto }`, `.app`
is `height:100vh; overflow:hidden`). Below 1040px that inverts and the body scrolls normally.
This matters: `window.scrollTo` does nothing on desktop; you must scroll `.scrollarea`.

CSS is organised in labelled blocks — search for `═══════ tokens`, `═══════ rail`,
`═══════ accordion`, etc. **Order is load-bearing** in two places, see §8.

---

## 6. Design system

**Concept:** a warm-paper reading document with a highlighter as the single accent. The
visual references the user converged on were literal.club (paper, serif, airy, highlighter)
plus spec-sheet furniture (black table header bars, hairline frames, a stamped footer).

**Tokens** (`:root`, all themed):

| | light | dark |
|---|---|---|
| `--paper` | `#fdfcfa` | `#161513` |
| `--band` | `#f8f3ef` | `#1c1b18` |
| `--ink` | `#1b1917` | `#eeeae3` |
| `--ink-2` | `#56514b` | `#b2aca3` |
| `--ink-3` | `#8b857d` | `#87817a` |
| `--rule` / `--rule-strong` | `#e7e1da` / `#cec6bc` | `#2c2a26` / `#403c37` |
| `--hi` (highlighter) | `#fbefa2` | `rgba(245,214,106,.26)` |

`--measure: 72ch` is the reading width; almost every block is capped to it.
`--ease-out: cubic-bezier(.23,1,.32,1)`.

**Aliases exist for the SVG only:** `--bg`, `--surface`, `--line`, `--line-2`, `--accent`,
`--mark` map onto the real tokens so the inline gaps-model SVG needs no edits. Don't delete them.

**Type:** Newsreader (serif) for everything readable; Geist Mono *only* for the course code,
stage numerals, priority chips, and `<code>`. Mono was deliberately cut back — the user
explicitly rejected a heavier monospace treatment as "killing the look". Don't reintroduce it
for labels, nav, captions or table headers.

**Colour rule:** monochrome plus the yellow highlighter. The user rejected an earlier
teal/rust palette as "typical Claude colors". Do not add a hue.

---

## 7. Component inventory (all CSS-only, no JS)

| Component | Selector | Count | Mechanism |
|---|---|---|---|
| Theme toggle | `.themebtn` + `#flip` | 1 | checkbox + `:has()`; **inverts** the system theme rather than setting an absolute one |
| Mobile nav drawer | `.rail#menu` | 1 | `:target`; opened by `a[href="#menu"]`, closes automatically because tapping any section changes the hash |
| Collapsible nav groups | `details.navgrp` | 3 | native `<details>`, `open` by default |
| Self-test accordions | `.sheet details` | 11 | native `<details>`; `@media print` force-expands all |
| Term popovers | `.term` / `.tip` | 35 | `:hover` on pointer, `:focus-within` on touch/keyboard |
| Highlighted terms | `<mark>` | 35 | gradient background, not a flat fill |
| Priority chips | `.mark` / `.mark.key` | 8 | 3 tiers: must know / important / background |
| Definition rows | `.terms` | 13 | 2-col grid, collapses <700px |
| Tables | `.scroller` + `table` | 7 | black header bar; scroll container **only** <760px |
| Callouts | `.note` | 9 | tinted band |
| Numbered cards | `.card` | 3 | the three-stage model |
| Gaps diagram | inline `<svg>` | 1 | uses the alias tokens |

Total `<details>` elements: 14. `<script>` tags: 0.

**Accessibility already handled:** skip link, `:focus-visible` rings, `aria-hidden` on
decorative glyphs, `.sr-only` label on the theme checkbox, `role="tooltip"`, 44px+ touch
targets, `prefers-reduced-motion` (keeps fades, drops movement and blur).

---

## 8. Gotchas — read this before editing the CSS

These cost real time this session. Each is a live landmine.

1. **`overflow-x: auto` silently forces `overflow-y: auto`.** Tables were scroll containers
   on desktop as a side effect. Worse, the hidden `.tip` popovers inside `<td>` are
   `position:absolute; visibility:hidden`, and **hidden elements still contribute to a scroll
   container's scrollable area** — inflating table scroll height by ~121px and producing a
   vertical scrollbar on a table that fits. Fix in place: `.scroller` only gets `overflow-x`
   below 760px, and below 760px the tips switch to `position:fixed`. Don't re-add blanket
   overflow.

2. **Media queries add no specificity.** A `@media (max-width:1039px){ h3{scroll-margin-top:74px} }`
   rule was being beaten by a later plain `h3{scroll-margin-top:20px}`. Source order decides.
   The mobile `scroll-margin` block is deliberately parked at the **end** of the stylesheet,
   just above `@media print`. Keep it there.

3. **`scroll-behavior: smooth` was removed on purpose.** The document is ~16,000px tall; smooth
   scrolling an anchor jump took several seconds and read as "the nav is broken". Do not re-add.

4. **Geist Mono has no `✱` (U+2731).** It silently falls back to another font with different
   metrics, which misaligns the stars in the priority chips. They are therefore wrapped in
   `<i class="st">` set in Newsreader, inside an `inline-flex` chip with `align-items:center`.
   Verify by canvas-measuring the glyph in Geist Mono vs generic monospace — identical widths
   means the glyph is missing.

5. **Tooltip edge overflow.** Tips default to `left:0` (hang rightward, where there is room).
   Only `td:last-child .tip` flips to `right:0`. An earlier blanket `td .tip{right:0}` pushed
   middle-column tips off the left edge of the pane.

6. **Chrome caches `file://` aggressively** — even `reload(ignoreCache)` served stale CSS
   repeatedly. Append a throwaway query string (`?v=2`) when testing locally. Irrelevant once
   served over HTTP from Pages.

7. **Generated selector lists.** Two bugs shipped from Python-generated CSS: doubled commas,
   and a `{ {`. Both silently killed **every rule after them**, which surfaced as mobile-only
   elements leaking onto desktop. If layout breaks globally and inexplicably, grep for `,,`
   and `{ {` first.

8. **`re.sub` eats CSS escapes.** `\263E` in a replacement string is parsed as a backreference
   and produced `³E` in the output. Use `str.replace`, or avoid escapes entirely (the theme
   icon is now CSS-drawn, no glyph).

---

## 9. Verification — how this was checked, and how to re-check

Driven through chrome-devtools MCP. Re-run after any layout change:

- **Viewports:** 1440×900 (desktop) and 390×844 @2x (`mobile,touch`), in both
  `colorScheme: light` and `dark`.
- **No horizontal overflow:** `document.documentElement.scrollWidth === clientWidth` at 390px.
- **Touch targets:** every `.nav a`, `.navgrp summary`, `.sheet summary`, `.menubtn` ≥44px tall.
- **Mobile nav, end to end** — this is the one I initially checked by computed style only and
  got wrong; **click through it**: tap `.menubtn` → rail `visibility:visible` → tap a section
  link → hash changes, rail goes `hidden`, and the heading lands *below* the sticky topbar
  (compare `heading.getBoundingClientRect().top` against `.topbar` `.bottom`).
- **Anchors:** all 18 `.nav a[href]` targets must resolve (`document.querySelector(href)`).
- **Theme toggle:** flip `#flip.checked` under both emulated system schemes; `--paper` must
  invert in both directions.
- **Tooltips:** focus a `.term` in a middle table column and one in the last column; the
  `.tip` rect must stay inside the `.pane` rect.
- **Print:** `@media print` drops rail/topbar/footer/tooltips, expands every `<details>`,
  and strips the theme.

---

## 10. Decisions already made — don't relitigate

- **No CSS framework.** Pico/Blades/Water were evaluated (Pico confirmed via Context7 as
  having CSS-only accordion/dropdown/card/modal). Rejected: ~80 KB plus a CDN dependency to
  style generic semantic HTML, when ~70% of this page is custom structure a classless
  framework would style wrong and then need overriding. The only thing worth taking —
  `<details>`/`<summary>` — is native HTML, not a library.
- **No JavaScript**, with one exception: the print button's `onclick="print()"`. Every other interaction is `:target`, `:has()`, `:focus-within` or
  `<details>`. Keep it that way; it is why the file needs no build and no CSP thought.
- **Serif body, not mono.** Considered and rejected — unreadable over a 35-minute document.
- **Em dashes are banned** in the prose (user preference). 95 were removed and replaced with
  context-appropriate punctuation. Don't reintroduce them. En dashes in ranges are fine.
- **British-ish plain wording** in UI copy; the user repeatedly asked for *shorter*. When in
  doubt, cut.

---

## 11. Backups / stray files

Two earlier design iterations are saved outside the project, in the session scratchpad:

```
/tmp/claude-1000/-home-fahim-playground-mid-MKT4101/dfbed4a1-.../scratchpad/
├── prep-backup.html          grey minimal iteration
└── prep-retro-terminal.html  monospace terminal iteration
```

These are **temp-dir files and will not survive indefinitely**. If the user may want them,
copy them somewhere durable before they vanish. They should **not** go in the repo.

---

## 12. Repo hygiene — decide with the user

`books/`, `slides/` and `suggestions/` total **~26 MB** of copyrighted textbook PDFs and
lecture slides. They are build inputs, not deliverables.

**Recommendation:** `.gitignore` them and commit only the HTML. Reasons: repo weight, and
publishing scanned textbook pages to a public GitHub Pages site is a copyright problem the
user probably has not considered. **Raise this with the user before pushing** — do not
silently commit or silently drop them.

Suggested `.gitignore`:
```
books/
slides/
suggestions/
.claude/
```

If the repo must be private, note that **GitHub Pages from a private repo requires a paid
plan**; a public repo is the free path, which makes the copyright question above sharper.

---

## 13. Next steps, in order

1. Ask the user: public or private repo, and confirm the PDFs/slides stay out of it (§12).
2. `git init`, add `.gitignore`, add `.nojekyll`.
3. `cp MKT4101-midterm-prep.html index.html` (or rename — §4).
4. Commit, create the GitHub repo, push.
5. Enable Pages: branch `main`, folder `/ (root)`.
6. Once live, re-run the §9 checks **against the deployed URL**, not the local file — this
   also clears the `file://` caching noise from §8.6.
7. Confirm the Google Fonts request succeeds over HTTPS and the page is legible if it fails
   (fallback stacks are declared: `Charter/Georgia/Times` and `ui-monospace/Menlo`).

---

## 14. Tools that were useful

- **chrome-devtools MCP** — the only reliable way to verify this. `emulate` for
  viewport/colour-scheme, `evaluate_script` for measuring rects and computed styles,
  `take_screenshot`. Browser binary: `/usr/bin/helium-browser`.
- **Context7 MCP** — used to check Pico CSS's actual component list rather than trusting memory.
- Toolchain rules from the user's CLAUDE.md: **`bun`/`bunx`, never `npm`/`npx`**;
  **`uv` for Python**; prefer shell (`cat`/`rg`/`sed`) over file-read tools.
