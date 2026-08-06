# Test cases

The assignment requires at least **3 normal** and **3 edge** cases presented and
walked through in the video demonstration.

> **Status honesty:** the structural checks below were **executed** by
> `tools/audit_html.py`. The browser-based checks were **not executed** — no
> browser is available in the environment where this was authored. They are
> written as a script to perform on camera, with the expected result stated in
> advance so the recording either confirms or refutes it. Fill in the Actual
> column while recording.

## Normal cases

### N1 — Renders correctly at desktop width
- **Steps:** open `index.html` at ≥1280px wide.
- **Expected:** two-column layout; Skills sidebar to the right of the main
  column, spanning from Objective down to References. Header spans full width.
- **Actual:** _record on camera_

### N2 — Heading outline is sequential
- **Steps:** run `AUDIT_PROFILE=resume python ../../tools/audit_html.py index.html style.css`.
  (Or in DevTools, the Accessibility pane's heading list.)
- **Expected:** order `[1, 2, 2, 3, 3, 2, 3, 2, 3, 2]`, no skipped levels,
  exactly one `<h1>`.
- **Actual:** ✅ **EXECUTED — PASS.** Tool output reproduced verbatim:
  `heading order: [1, 2, 2, 3, 3, 2, 3, 2, 3, 2]` / `PASS no skipped heading levels`.

### N3 — Semantic landmarks are all present and correctly nested
- **Steps:** same tool run; or DevTools → Elements.
- **Expected:** `header`, `nav`, `main`, `section`, `article`, `aside`, `footer`
  all present; two `<article>` inside `<section id="experience">`; zero `<div>`
  and zero `<span>` in the document.
- **Actual:** ✅ **EXECUTED — PASS.** 15/15 structural assertions passed.

## Edge cases

### E1 — Narrow viewport (320px)
- **Steps:** DevTools device toolbar → responsive → width 320px.
- **Expected:** single column; sidebar drops below References; nav links wrap
  onto multiple lines; **no horizontal scrollbar**.
- **Actual:** _record on camera_
- **Risk:** this is the case most likely to fail. It was reasoned about, never
  observed. Watch specifically for horizontal overflow.

### E2 — Keyboard-only navigation
- **Steps:** load the page, press <kbd>Tab</kbd> repeatedly without touching the
  mouse.
- **Expected:** first stop is the "Skip to main content" link, which becomes
  *visible* on focus; pressing <kbd>Enter</kbd> jumps focus past the nav. Every
  subsequent link shows a visible blue focus ring. No focus trap; no invisible
  focused element.
- **Actual:** _record on camera_

### E3 — Stylesheet fails to load
- **Steps:** DevTools → Network → block `style.css`, then reload. (Equivalent to
  a CDN failure or a very old browser.)
- **Expected:** the page remains fully readable and in a sensible order —
  name, role, contact, nav, objective, experience, education, skills, references,
  footer. Nothing is hidden or unreachable, because meaning lives in the HTML
  rather than the CSS.
- **Actual:** _record on camera_
- **Why this one matters:** it is the clearest possible demonstration of what
  "semantic HTML" buys you, which is the concept the assignment is testing.

## Additional executed checks (not part of the required six)

| Check | Method | Result |
|---|---|---|
| No inline `style=` attributes | regex over `index.html` | ✅ PASS — 0 found |
| No `float: left/right` in CSS | regex over `style.css` | ✅ PASS — 0 found |
| Uses Grid or Flexbox | regex over `style.css` | ✅ PASS |
| Has a responsive media query | regex over `style.css` | ✅ PASS |
| `<html lang>` present | regex | ✅ PASS |
| Viewport meta present | regex | ✅ PASS |
| Dates use `<time datetime>` | regex | ✅ PASS |
| Definition list used | regex | ✅ PASS |
| OG image is an absolute URL | manual read of `<head>` | ✅ PASS |
| `robots.txt` sitemap target exists | file listing | ✅ PASS — `sitemap.xml` committed |

## Explicitly NOT tested

- Contrast ratios with a real tool (computed by hand only)
- W3C HTML validator
- Any screen reader
- Print output
- The OG card rendering on a real social platform
- Live GitHub Pages serving
