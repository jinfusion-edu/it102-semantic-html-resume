# How this works, and why

Written for someone learning the material, not for someone grading it.

---

## 1. The concept being tested

The assignment is not really about resumes. It is about the difference between
**markup that describes content** and **markup that describes appearance**.

Both of these look identical in a browser:

```html
<div class="big-bold">John Bruner</div>

<h1>John Bruner</h1>
```

They are not equivalent. The first is a generic box that happens to be styled
large and bold. The second tells every machine reading the page — screen
readers, search engines, the browser's own accessibility tree, Reader Mode,
LinkedIn's preview scraper — *this is the top-level heading of this document*.

Semantic HTML is metadata. A sighted user cannot see the difference; everyone
and everything else can. That is why the assignment forbids `<div>` "where
semantic elements fit" and audits the heading hierarchy separately from the
styling.

---

## 2. Walking the document

### The landmarks

The page is divided into regions that assistive technology can jump between:

```html
<header>   name, role, contact
<nav>      in-page section links
<main>     the resume itself
  <section id="objective">
  <section id="experience">
      <article>  one job
      <article>  another job
  <section id="education">
  <aside id="skills">
  <section id="references">
<footer>   copyright + privacy notice
```

A screen-reader user can press a key to list landmarks and jump straight to
`main`, skipping the header and nav entirely. That is not something CSS can
provide; it comes from choosing the right element name.

**Why `<section>` vs `<article>`?** The rule of thumb: an `<article>` should
still make sense if you cut it out and pasted it somewhere else. A single job —
title, employer, dates, accomplishments — is self-contained, so it is an
`<article>`. "Experience" is only meaningful as part of this resume, so it is a
`<section>`. The assignment's audit checklist asks for exactly this nesting:
each job in an `<article>` inside an Experience `<section>`.

**Why `<aside>` for skills?** `<aside>` means "related to the main content but
not part of its main flow". The chronological narrative is Objective →
Experience → Education. The skills list supports that narrative without being
a step in it. It is the one place on a resume where `<aside>` is genuinely
correct rather than decorative.

### The heading hierarchy

```
h1  John Bruner
  h2  Career Objective
  h2  Experience
    h3  End User Support Technician
    h3  Samsung Expert
  h2  Education
    h3  St. Mary's University
  h2  Skills
    h3  Certification
  h2  References
```

Headings form a document outline, like the contents page of a book. Skipping
from `<h1>` to `<h4>` because `<h4>` "looked the right size" leaves a hole in
that outline — a screen reader announces "heading level 4" and the listener has
no idea what happened to levels 2 and 3.

The fix is to never choose a heading for its size. Choose it for its depth, and
set the size in CSS. That is why `style.css` sets an explicit `font-size` on
`h1`, `h2` and `h3`: the visual scale is decoupled from the structural level.

There is exactly one `<h1>` because the document has exactly one subject.

### Dates and `<time>`

```html
<time datetime="2020-08">August 2020</time> – Present
```

The text a human reads is "August 2020". The `datetime` attribute carries the
same instant in a machine-readable format (`YYYY-MM`). A parser does not have
to guess whether "08/2020" means August or the 8th of some month, or handle
"Aug", "August", "8/20" as separate cases.

Note `Present` is **not** wrapped in `<time>`. There is no machine value for
"now, ongoing" — writing `datetime="2026-08-06"` would be a lie, because it
would claim the job ended today.

### Definition lists for skills

```html
<dl>
  <dt>Programming languages</dt>
  <dd>C, C++, PowerShell, Java — 8+ years</dd>
</dl>
```

`<dl>` is a list of name/value pairs: `<dt>` is the term, `<dd>` the
description. It is the right element whenever data is genuinely paired, and it
is stronger than a `<ul>` of `"Programming languages: C, C++..."` strings
because the pairing is structural rather than living inside a colon.

---

## 3. The CSS

### Layout: Grid for the page, Flexbox for the rows

The two are complementary, not competing:

- **Grid** is two-dimensional — you place things into rows *and* columns at
  once. It handles the page skeleton.
- **Flexbox** is one-dimensional — a row or a column of items that share space.
  It handles the header stack and the nav strip.

```css
main {
  display: grid;
  gap: 2.5rem;
}

@media (min-width: 52rem) {
  main {
    grid-template-columns: minmax(0, 2fr) minmax(0, 1fr);
  }
  #skills {
    grid-column: 2;
    grid-row: 1 / span 4;
  }
}
```

Read that in order. By default `main` is a single-column grid — every child
stacks. That is the **mobile layout, and it is the default**, so no media query
is needed for small screens. Only when the viewport is at least 52rem do we
introduce a second column and move the skills sidebar into it.

This is mobile-first: the base stylesheet describes the simplest case, and
media queries *add* complexity for larger screens. The alternative — writing
the desktop layout first and then undoing it with `max-width` queries — means
every rule needs a counter-rule.

`minmax(0, 2fr)` rather than plain `2fr` prevents a long unbroken string (a URL,
say) from forcing the column wider than its share. `2fr` alone has an implicit
minimum of `auto`, which means "at least as wide as my content".

`grid-row: 1 / span 4` places the sidebar so it runs alongside all four content
blocks rather than sitting in a single row of its own.

### Why no floats

`float` was designed for wrapping text around an image. For most of the 2000s
it was also the only tool available for columns, so an entire generation of
layout hacks was built on it — clearfixes, negative margins, equal-height
workarounds. Grid and Flexbox were designed for layout, so they do it without
side effects. The assignment forbids floats because reaching for one in 2026
signals copied-from-2010 code.

### The typographic scale

```css
font-size: clamp(1rem, 0.95rem + 0.25vw, 1.1875rem);
```

`clamp(minimum, preferred, maximum)` gives fluid type that stops growing at
both ends. Text scales gently with viewport width but never drops below 16px
(which triggers zoom-on-focus on iOS) and never exceeds ~19px. The alternative —
a fixed size plus several breakpoints — produces visible jumps at each one.

### Accessibility details worth naming

```css
.skip-link { position: absolute; left: -9999px; }
.skip-link:focus { left: 0; }
```

The skip link is off-screen until focused. A keyboard user pressing Tab on load
gets "Skip to main content" as the first stop and can bypass the nav. Note it is
moved off-screen rather than `display: none` — a hidden element cannot receive
focus at all, so `display: none` would defeat the purpose.

```css
a:focus-visible { outline: 3px solid var(--accent); }
```

`:focus-visible` shows the ring for keyboard focus but not for mouse clicks, so
the design stays clean without stripping the indicator keyboard users depend on.
Removing focus outlines entirely (`outline: none`) is one of the most common
accessibility failures on the web.

Colours were chosen against contrast targets, not by eye: `--ink` `#1f2933` on
white is roughly 14:1, `--ink-muted` `#52606d` about 7.3:1, and the `--accent`
link colour `#0b5d8a` about 6.5:1. AA requires 4.5:1 for body text.

---

## 4. The production layer (second assignment)

### Open Graph

When you paste a link into Slack, LinkedIn or Discord, that service fetches the
page and looks for `og:` meta tags to build a preview card. Without them it
guesses — usually badly, showing a bare URL or a random image.

```html
<meta property="og:image"
      content="https://jinfusion-edu.github.io/it102-semantic-html-resume/og-preview.png" />
```

**The URL must be absolute.** A scraper is not browsing your site; it has no
base URL to resolve `og-preview.png` against. Relative image paths are the most
common reason a preview card renders blank, and they fail silently — the page
looks perfect in a browser.

Note `og:title` duplicates the `<title>` and `og:description` duplicates the
meta description. That repetition is intentional: they serve different consumers
and a scraper will not fall back to the standard tags reliably.

### Canonical URL

The same files are served from three domains. To a search engine that looks
like three copies of one document, and it has to decide which to rank —
"duplicate content".

```html
<link rel="canonical" href="https://jinfusion-edu.github.io/..." />
```

This says: wherever you found this page, the real address is this one. All
three deployments carry the *same* canonical tag pointing at GitHub Pages. So
the Netlify copy advertises the Pages URL, which looks wrong at a glance and is
exactly right.

### robots.txt

A crawler requests `/robots.txt` before anything else. It is advisory —
well-behaved crawlers obey, hostile ones ignore it. So it manages
discoverability and is **never a security control**. Putting `Disallow: /admin`
in it tells everyone that `/admin` exists.

Here everything is allowed, because a public resume wants to be found.

---

## 5. Alternatives considered

**A CSS framework (Bootstrap, Tailwind).** Faster to write, but the assignment
grades semantic structure and hand-written modern CSS. Framework markup tends
toward `<div class="row"><div class="col-md-8">`, which is the exact pattern
being marked against, and none of it is in the course's scope yet.

**A static site generator.** Overkill for one page, and it would put a build
step between the source and the deployed output — the opposite of what the
deployment assignment is demonstrating.

**`<table>` for the two-column layout.** How this was done in 1999. Tables carry
semantics too: a screen reader announces rows and columns and tries to relate
cells to headers. Using one for visual arrangement makes the page actively
harder to understand aurally.

**Separate `<section>` for skills instead of `<aside>`.** Defensible — but
`<aside>` communicates "supporting, not part of the main flow", which is exactly
what a skills list is, and the assignment explicitly lists `<aside>` among the
elements to use appropriately.

**A JPEG photograph for the OG image.** Better, and the honest answer is that no
photograph was supplied and no screenshot tool was available in this environment,
so a generated placeholder card ships instead. That limitation is recorded in
`AUDIT-NOTES.md` rather than hidden.
