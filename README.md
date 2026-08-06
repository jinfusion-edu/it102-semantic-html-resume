# Semantic HTML Resume — John Bruner

IT102 · Introduction to Programming · Seattle Colleges

A personal resume website built from strict semantic HTML and modern CSS, then
deployed to production with Open Graph, SEO and crawler configuration.

This repository covers **two** assignments:

- **Build a Semantic HTML Resume with AI** — the page itself
- **Deploy Your Semantic Resume to the Wild** — production tags and hosting

## What it does

Renders a single-page resume. There is no JavaScript and no build step. Layout
is CSS Grid (two columns on wide screens, one column on narrow), with Flexbox
for the header and navigation. It is designed to meet WCAG 2.1 AA: a skip link,
visible focus rings, landmark elements, sequential headings, and text/background
pairs above a 4.5:1 contrast ratio.

## Files

| File | Purpose |
|---|---|
| `index.html` | The resume. All structure and content. |
| `style.css` | All presentation. No inline styles anywhere. |
| `og-preview.png` | 1200×630 social preview card referenced by `og:image`. |
| `favicon.svg` | Browser tab icon. |
| `robots.txt` | Allows all crawlers; points at the sitemap. |
| `sitemap.xml` | Single-URL sitemap. |
| `DEPLOYMENT.md` | Netlify + Cloudflare steps, and the video shot list. |
| `EXPLANATION.md` | How and why the code works, written to teach. |
| `AUDIT-NOTES.md` | Self-audit against the assignment's checklist. |
| `TEST-CASES.md` | The six test cases and their recorded results. |

## How to run it

No dependencies, no server required.

```bash
git clone https://github.com/jinfusion-edu/it102-semantic-html-resume.git
cd it102-semantic-html-resume
```

Then either open `index.html` directly in a browser, or serve it locally so
that `robots.txt` and absolute paths behave as they do in production:

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

## Expected output

A resume page with the name as an `<h1>`, contact details in a `<header>`,
in-page section navigation, Experience and Education entries as `<article>`
elements inside `<section>` elements, a Skills sidebar in an `<aside>`, and a
`<footer>` carrying the copyright and data-privacy notice. On a screen wider
than 52rem the Skills sidebar sits to the right of the main column; below that
it stacks underneath.

## Live URLs

| Platform | URL |
|---|---|
| GitHub Pages | https://jinfusion-edu.github.io/it102-semantic-html-resume/ |
| Netlify | _pending — see DEPLOYMENT.md_ |
| Cloudflare Pages | _pending — see DEPLOYMENT.md_ |

## AI collaboration — tools and prompts

**Tool used:** Claude (Anthropic), via Claude Code.

The assignment's method is to prompt an AI, then audit what comes back. The
prompt below is the assignment's own framework with the bracketed placeholder
filled in with real resume data.

### Prompt 1 — generate the page

> Act as an expert frontend developer specializing in web accessibility
> (WCAG 2.1 AA) and modern semantic HTML.
>
> I am going to provide my resume details. Your task is to generate the code
> for a personal resume webpage using ONLY a single HTML file and a single CSS
> file.
>
> Strict Requirements:
> 1. Structural Semantics: Do NOT use generic `<div>` or `<span>` tags where
>    semantic elements fit. Use `<header>`, `<nav>`, `<main>`, `<section>`,
>    `<article>`, `<aside>`, and `<footer>` appropriately.
> 2. Typography & Hierarchy: Use heading tags (`<h1>` through `<h6>`) in strict
>    sequential order. Never skip a heading level for styling purposes.
> 3. Accessible Lists & Data: Wrap skills in appropriate lists, and format
>    dates/locations using semantic elements like `<time>` or definition lists
>    (`<dl>`) if applicable.
> 4. Clean CSS: Use CSS Flexbox or Grid for layout. Do not use inline styles or
>    old float-based layouts. Ensure a clean, professional typography scale and
>    a responsive design that looks great on mobile and desktop.
>
> Here is my resume information:
> *(real resume content — name, objective, two roles with dates, education,
> skills; street address and phone deliberately withheld, see AUDIT-NOTES.md)*

### Prompt 2 — the production audit (second assignment)

> I have deployed my raw HTML/CSS resume to GitHub Pages, Netlify, and
> Cloudflare Pages. Act as a senior DevOps engineer and frontend performance
> expert. Review my HTML and CSS code and provide the code/instructions for
> three critical production optimizations:
>
> Open Graph (OG) Meta Tags: Generate the `<meta>` tags I need to add to my
> `<head>` so that when I share my resume link on LinkedIn, Slack, or Discord,
> it displays a professional preview card (including title, description, and a
> preview image).
>
> SEO & Favicon: Give me the exact HTML code to point to a professional favicon
> and add a search-engine-friendly description.
>
> Performance Diagnostics: Explain what a robots.txt file is, and generate a
> basic, valid robots.txt file for my site that allows search engines to index it.

### What I changed after reviewing the output

Auditing rather than pasting is the graded skill, so the corrections are
recorded in `AUDIT-NOTES.md`. In summary: the `og:image` had to be made an
absolute URL, a real preview image had to be produced because the generated
markup left a placeholder, a `sitemap.xml` had to be created because
`robots.txt` referenced one, and a canonical host had to be chosen because the
same code is served from three domains.

## Verification

Structural checks are executable:

```bash
AUDIT_PROFILE=resume python ../../tools/audit_html.py index.html style.css
```

15 checks, all passing as of the last commit. See `TEST-CASES.md` for the full
list and `AUDIT-NOTES.md` for what is *not* covered.
