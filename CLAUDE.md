# Documentation guides — project guide for Claude

This repo is a small **hub of self-contained HTML documentation guides**, all sharing one
hand-crafted design system ("warm paper", light + dark, bilingual IT/EN). This file explains
how the guides are built so a **new guide can be produced that matches the existing ones EXACTLY**.

## Files

- `docs/index.html` — the **hub** (landing page) that links to every guide.
- `docs/git.html`, `docs/markdown.html` — the guides. Each is one **fully self-contained** HTML
  file: all CSS and JS are inline; the only external dependency is Google Fonts. No build step,
  no bundler, no framework, no external images (icons are inline SVG).
- `LICENSE` — CC BY-NC-SA 4.0 (the license of the content). `.gitignore` — macOS cruft.
- A new guide is `docs/<topic>.html` and must also be added as a card on the hub.

## THE GOLDEN RULE

**To create a new guide, copy an existing guide and replace ONLY the content.**

```
cp docs/markdown.html docs/<topic>.html   # or git.html
```

Then change only: `<title>` + meta description, the brand (glyph + title + subtitle), the TOC,
the whole `<article>` inner content (hero + sections + references + footer), the diagrams, and
the `EN_DICT`. **Keep the entire `<style>` block and the JS engine byte-for-byte identical.**
Never recreate the CSS/JS from scratch and never "restyle" — fonts, colors, margins, tokens,
layout, components are all inherited by copying. Delete anything left over from the source guide
(e.g. git.html still carries a base64 xkcd comic + its `.comic` CSS — remove dead pieces).

`git.html` and `markdown.html` are the **source of truth** for every component. When unsure how
something should look or be marked up, open one of them and match it.

## Design system (do not reinvent — it comes free with the copy)

- **Fonts:** Fraunces (display / headings), Newsreader (body), JetBrains Mono (code, labels,
  diagram text). Loaded from Google Fonts in `<head>`.
- **Theme:** warm paper. Light + dark via `data-theme` on `<html>`. A small **theme-preload
  script in `<head>`** applies the theme before first paint (reads `localStorage.theme` =
  `auto|light|dark`; `auto` follows `prefers-color-scheme`). Never remove it (avoids flash).
- **Tokens:** all colors/shadows are CSS custom properties defined twice — `:root` (light) and
  `[data-theme="dark"]`. Key ones: `--bg --surface --surface-2 --ink --ink-soft --muted
  --border --border-strong --accent --accent-ink --accent-soft --teal --teal-soft --amber
  --shadow --shadow-sm --sel`; code tokens `--code-bg --code-ink --code-comment --code-string
  --code-prompt --code-border --code-head --code-name --code-scroll --code-ok`; diagram tokens
  `--dgm-*`. **Always use tokens, never hardcode colors** (the only hardcoded color is `#fff`
  for the topbar brand glyph stroke and the mac-window dots).
- **Spacing is intentionally compact.** Match the existing margins/padding; do not inflate
  whitespace. When in doubt, copy the spacing of the analogous component in another guide.

## Page anatomy of a guide

1. `<head>`: charset, viewport, `<title>`, meta description, theme-preload script, fonts link,
   the full inline `<style>`.
2. `<body>`: reading-progress bar; **shared SVG defs** (arrowhead marker `#ah`, flag symbols
   `#flag-it` / `#flag-gb`); **topbar** (brand glyph + title + subtitle; theme button; language
   dropdown); **layout grid** = sidebar + `<main>`; scripts.
3. **Sidebar** `<nav class="toc-wrap">`: kicker "Indice", `<ul class="toc" id="toc">` of numbered
   items, and at the **bottom** a `.hub-link` "← Tutte le guide" → `index.html` (see Cross-links).
4. **Article**: hero (`.eyebrow`, `h1` with exactly one `<em>` accent word, `.lead`, `.meta`
   chips) → numbered `<section class="section reveal" id="...">` each with
   `.sec-head` (`.sec-num` + `h2`) + `.rule`, then content. Components: callouts
   `.note` / `.note.tip` / `.note.warn`; tables `.table-wrap > table`; diagrams `figure.diagram`;
   code blocks `pre.code-src`; `.learn` links (inline SVG + text); the references section
   (`.refs` of `.refcard`); the footer `.foot` (intro paragraph, CC badge, license note).
- **Section ids:** use the heading's GitHub auto-anchor — lowercase, spaces→hyphens, punctuation
  removed (e.g. "Liste, citazioni, righe" → `id="liste-citazioni-righe"`). Keep the TOC `href`
  in sync so internal links resolve and scroll-spy works.
- **reveal-on-scroll:** sections carry `class="reveal"`; an IntersectionObserver adds `.in`.
  (When screenshotting, force `.in` on all `.reveal` via JS, otherwise off-screen sections stay
  at opacity 0.)

## Content language & writing rules

- **Author the DOM in Italian.** English is supplied separately (see Bilingual). Never translate
  existing Italian to English in the DOM.
- Follow the user's global writing rules: **no em dashes** (use commas), and **never italianize
  English verbs** (e.g. "fare il commit", not "committare").

## Code blocks

- Author as `<pre class="code-src" data-name="filename">…</pre>`. JS rebuilds each into the fancy
  block (mac dots + filename + "copia" button + highlighted `<pre>`).
- **Theme-adaptive:** light panel in light theme, dark in dark theme (via `--code-*` tokens).
- **Highlighter:** default = render the content **plain** (so Markdown symbols `# * > \``
  stay visible). Add the **`data-sh`** attribute to a block to get shell highlighting (`#`-lines
  as comments, "quoted strings" colored) — use ONLY for terminal/shell command blocks.
  (git.html does NOT have the plain/`data-sh` switch because all its blocks are shell — it keeps
  the original highlighter. markdown.html added the switch. If a new guide mixes prose-syntax and
  shell, copy markdown.html's highlighter.)
- **Escape `<`, `>`, `&` as entities** inside `code-src` (`&gt;` for a blockquote, `&lt;…&gt;`
  for an autolink). The copy button copies the decoded text.

## Bilingual (IT / EN)

Two different mechanisms — do not confuse them:

### Guides (index-aligned arrays)
The DOM is Italian; an inline `EN_DICT` holds parallel **arrays aligned by document order**.
At load the engine captures the Italian BASE from the DOM via these selectors, then swaps to
English when the language toggles:

- `content[]` ← `'.article .eyebrow, .article h1, .article .lead, .article .chip, .article h2,
  .article h3, .article p, .article li, .article figcaption, .article th, .article td,
  .article .note .bd b'` (each element's innerHTML, **document order**).
- `icon[]` ← `'.article .learn, .article .refcard h4'` (text after the inline SVG).
- `svgMap{}` ← `'.article .dgm text'` keyed by the Italian string → English (include only entries
  that differ; identical ones fall back to the Italian).
- `code[]` ← `'pre.code-src'` as `{name, body}`.
- `toc[]` ← `'#toc a > span:last-child'`.
- `meta{}` = `{title, desc, brandTitle, brandSub}`.

**The EN_DICT array lengths MUST equal the DOM counts, index for index.** Adding, removing or
reordering content shifts indices, so EN_DICT must be regenerated or surgically patched.

Workflow to build/verify EN_DICT:
1. Serve locally: `cd docs && python3 -m http.server 8731` (stdlib only, no install).
2. Load in a headless browser and call **`window.__i18nDump()`** → returns the exact Italian
   BASE (a JSON string; it is double-encoded, so `JSON.parse` twice / `jq 'fromjson'`).
3. Translate every entry (a value→value map keyed by the Italian string is robust against
   reordering and de-duplicates repeats like "↳ risultato reso"). Assert every Italian string is
   covered so nothing leaks untranslated.
4. Serialize with `json.dumps(..., ensure_ascii=False, separators=(",", ":"))` and splice it onto
   the single `  var EN_DICT = {...};` line.
5. **Verify:** `len(BASE.content) == len(EN_DICT.content)` for every array; then reload, switch to
   EN, and check a LATE section (references) is correct EN — if indices were off, late content
   would be shifted.
6. For a small addition, surgically insert the matching EN entries at the same positions (locate
   by value, e.g. "insert the new content entries right before the `Quotes` heading; insert the
   new code object right after `numbered-list.md`"). Watch for multi-line/nested entries (a nested
   `<li>` carries its child `<ul>` markup as one entry).

### Hub (`index.html`) — simpler
Translatable elements carry `data-i18n="key"`; a JS `I18N = { key: {it, en} }` + `META = {it,en}`
sets innerHTML/title on toggle. Add a `data-i18n` key + dict entry for any new text.

### Shared bits
- `localStorage.lang` and `localStorage.theme` are **shared across hub + guides** (consistent
  preference between pages).
- An element that must be bilingual **without** touching EN_DICT (e.g. the sidebar "back to hub"
  link, which lives outside `.article`) uses two spans `.t-it` / `.t-en` toggled by CSS on
  `html[lang="en"]` (the engine sets the `lang` attribute on `<html>`).

## Diagrams (inline SVG)

Markup: `<figure class="diagram"><svg class="dgm" viewBox="0 0 W H" role="img" aria-label="…">…
</svg><figcaption>…</figcaption></figure>`.

- **Contrast in BOTH themes is mandatory.** Achieve it by using the shared `.dgm` classes only,
  which are driven by `--dgm-*` tokens that flip with the theme: `.box .work .stage .repo .remote
  .commit .blob .branch .head .tag` (filled shapes), `.edge` / `.edge-d` (dashed) for connectors,
  `.t` (centered text) `.cap .cmd .area-name .hash .lbl` (text styles). **Never hardcode diagram
  colors.** Box title text = body font 15px bold; captions/labels = `.cap` mono 13px; output/box
  labels ≈ 14px. Keep these sizes consistent with the other guides — do **not** shrink fonts to
  fix overflow.
- **Connectors:** only horizontal/vertical segments or 45° diagonals, rounded joins; arrowheads
  via `marker-end="url(#ah)"`. For fan-outs use an orthogonal "bus" (one trunk + stubs), never
  curved or crossing lines.
- **Hard rules — verify the RENDERED result, fix in SVG coordinates (CSS scaling does not fix
  internal overlaps):**
  - No lines intersect each other; no line crosses a label or a box border.
  - **No text overflows its box.** Each label must sit inside its rect with ≈12–16px clearance
    per side. If it doesn't fit: **widen the box** (keep fonts as-is). If an arrow label doesn't
    fit in the gap between two boxes, **widen the gap** or move the label above/below.
  - Keep diagrams compact (~75% of content width — the `figure.diagram` max-width handles it) and
    centered.
- **Check BOTH languages:** English strings differ in length (e.g. "scrivi" → "you write"), so a
  box/gap that fits in Italian may overflow in English. Size for the longer of the two.
- **Measurement method (do this, don't eyeball):** in the browser, for every `.dgm text` compare
  `getBBox()` against the containing `rect` (x..x+width); flag any text with negative margin or
  whose box intersects a different rect; run it with `lang=it` and `lang=en`. Then confirm
  visually with element screenshots in light and dark.

## "Source vs rendered" demo (used in markdown.html)

For syntax examples, show source and result side by side:
```
<figure class="md-demo"><div class="md-demo-grid">
  <pre class="code-src" data-name="x.md">SOURCE</pre>
  <div class="md-render"><p class="rh">↳ risultato reso</p><div class="rb">RENDERED</div></div>
</div></figure>
```
- The generated code block fills the left grid cell (the hidden `pre.code-src` is `display:none`).
- The `.md-render .rb` styles cover rendered elements: headings via `<p class="md-h1|md-h2|md-h3">`
  (NOT real `h1/h2/h3`, to avoid clashing with section styles and the content selectors),
  `ul/ol/li`, `ul.tasks` + checkboxes, `blockquote`, inline `code`, `table`, `hr`, `del`, `sup`
  (footnotes), `pre.md-cb` (rendered code block), `svg.md-img` (placeholder image).
- A rendered code block may be syntax-highlighted with simple spans `.k` (keyword), `.f`
  (function), `.n` (number) → `--code-prompt` / `--teal` / `--code-string`. Keep the **source**
  plain; only the rendered side is colored (that is the teaching point).
- If the rendered must match the source across languages, either use **language-neutral** example
  code (e.g. `def area(r): return 3.14 * r * r`) or translate both the source body and the render.
- Keep long source lines visible: split over two lines (a Markdown soft wrap still renders as one
  paragraph) so important parts (e.g. a long `#anchor`) aren't hidden behind horizontal scroll.

## Cross-links

- **Each guide → hub:** a `.hub-link` "← Tutte le guide" at the **bottom of the sidebar** (inside
  `.toc-wrap`, after the `</ul>`), pointing to `index.html`, bilingual via `.t-it`/`.t-en`.
- **Hub → guides:** each guide is a curated **card**, never a bare link. A `.doccard` is an `<a>`
  containing: an icon tile (`.ic`, gradient `--card-grad`) with an inline SVG glyph; a `.kicker`
  (mono uppercase); an `<h2>` title; a `.desc`; `.tags` chips; and a `.go` CTA ("Apri la guida" +
  arrow SVG that slides on hover). Each card declares its own accent palette
  (`--card-grad/--card-soft/--card-stroke/--card-ink/--card-shadow`) — give every new guide a
  **distinct accent** (git = orange/accent, markdown = teal; pick a new one next).

## Keep counts in sync

- **Hub guide count:** the topbar `.count` chip (`data-i18n="count"`, e.g. `<b>3</b> guide`) and
  its `I18N.count` it/en — bump it whenever you add/remove a guide.
- **Per-card chapter count:** the `.tags` chip like "11 capitoli" / "11 chapters" on each card —
  keep it equal to the guide's number of sections.
- **EN_DICT array lengths:** must always equal the DOM counts (see Bilingual). After any content
  change, re-dump and assert.

## Licensing & third-party assets

- Content is **CC BY-NC-SA 4.0**. Every page footer shows the CC badge + license note; the repo
  root has the canonical `LICENSE` (full legal code, fetched from SPDX, not hand-typed).
- Any third-party asset must be attributed with its own license (e.g. git.html previously embedded
  an xkcd comic credited as CC BY-NC 2.5).

## Verification checklist before declaring done

Serve locally and drive a headless browser; use cache-busting (`?v=N`) after edits.

- [ ] No console errors (a missing `favicon.ico` is harmless).
- [ ] EN_DICT counts equal the dumped IT BASE counts for every array.
- [ ] Toggle **IT and EN** and **light and dark**: hero, diagrams (svgMap), code blocks
      (plain vs `data-sh`), demos, tables, callouts, references, footer, sidebar hub-link.
- [ ] Diagrams: getBBox check (both languages) shows no overflow/intersection; visually OK in
      both themes.
- [ ] Internal anchors resolve; TOC + scroll-spy work; "back to hub" works; hub card links work.
- [ ] Hub guide count and per-card chapter counts are correct.
- [ ] Clean up: remove any temp files/screenshots written to the repo (and `.playwright-mcp/`);
      `git status` should show only the intended files.

## Git

- History lives on `main`. Commit messages in English; end them with the project's
  `Co-Authored-By` trailer. Propose the commit; the **push is typically run by Pietro**
  (`! git push origin main`) — don't assume it's done.
