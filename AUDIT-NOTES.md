# Audit notes

Written to be attacked. Structured to mirror the assignment's own **Part 2: The
Audit** checklist first, then everything that checklist does not cover.

---

## Part 2 — the assignment's audit checklist

### ▸ Header & Footer
> *"Is your name and contact info wrapped in a `<header>`? Is the copyright or
> data-privacy notice in a `<footer>`?"*

**Yes, both.** Name (`<h1>`), role, and contact are inside `<header>`. Contact
uses `<address>`, which is the correct element for contact details *of the
document's author* — worth stating because `<address>` is widely misused for
postal addresses generally.

The `<footer>` carries both a copyright line and a data-privacy notice. The
assignment says "copyright **or**"; I included both because the privacy notice
does double duty as the disclosure for the next point.

**Unflattering detail:** the footer hardcodes `2026`. It will be wrong on
1 January 2027 and there is no JavaScript to update it. A one-line `<script>`
would fix it but the assignment specifies HTML and CSS only.

### ▸ The Heading Hierarchy
> *"Did it start with an `<h1>` for your name, and then use `<h2>` for major
> sections? Ensure it didn't jump from an `<h1>` straight to an `<h4>`."*

**Yes.** Verified by execution, not by eye —
`python tools/audit_html.py` parses every heading tag in order and fails on any
skip. Actual output:

```
heading order: [1, 2, 2, 3, 3, 2, 3, 2, 3, 2]
PASS  starts with a single <h1>
PASS  exactly one <h1>
PASS  no skipped heading levels
```

### ▸ Experience Blocks
> *"Each job should ideally be contained within an `<article>` tag inside an
> 'Experience' `<section>`, treating each role as a self-contained composition."*

**Yes.** Two `<article>` elements inside `<section id="experience">`. Checked
programmatically: the auditor isolates the experience section and counts
`<article>` tags within it (result: 2).

**Worth flagging:** Education also uses `<article>` inside its `<section>`. The
checklist only demands this for jobs. I judged a degree entry to be equally
self-contained, but a strict grader could argue a single education entry does
not need the wrapper. It is additive, not a violation.

### ▸ Dates
> *"Look closely at how dates are written. Did the AI just use plain text, or
> did it wrap them in a `<time>` tag?"*

**Wrapped in `<time>`,** with machine-readable `datetime` attributes in
`YYYY-MM` form (`datetime="2020-08"`).

**Deliberate exception:** the word `Present` is *not* wrapped. There is no
honest machine value for an ongoing role — any date I supplied would assert an
end date that does not exist. If the grader expects every date-like token inside
a `<time>`, this reads as a miss. I think the omission is correct and am
flagging it rather than hiding it.

---

## Beyond the checklist

### Assumptions about environment and grader behaviour

1. The grader opens `index.html` in a current browser. CSS Grid, `clamp()`,
   `:focus-visible` and custom properties are all used. `clamp()` and
   `:focus-visible` are the newest; both are well past baseline support, but on
   a browser older than ~2021 the type would be fixed-size and focus rings would
   fall back to the UA default. Nothing breaks; it degrades.
2. The repo is public and served from the repository root.
3. Files are served over HTTP for the live URL. Opening `index.html` via
   `file://` works for the page, but `robots.txt` and `sitemap.xml` are only
   meaningful over HTTP.
4. `og:image` assumes the canonical GitHub Pages URL is reachable. If the repo
   is ever renamed or moved out of the `jinfusion-edu` org, every absolute URL
   in `<head>`, `robots.txt` and `sitemap.xml` silently breaks. They are
   hardcoded in four files.

### Requirements interpreted rather than read literally

- **"ONLY a single HTML file and a single CSS file."** Taken as constraining the
  *webpage*, since the same assignment separately requires a `README.md`, and
  the deployment assignment requires `robots.txt`, a favicon and an OG image.
  Read absolutely literally, those requirements contradict each other. The page
  itself loads exactly two files.
- **"Do NOT use generic `<div>` or `<span>` where semantic elements fit."**
  There are **zero** `<div>` and `<span>` elements in the document. This was
  achievable only because the layout is Grid — a float layout would have needed
  wrapper elements.
- **"Wrap skills in appropriate lists."** Used `<dl>` for skill-category pairs
  and `<ul>` for the certification. A grader searching only for `<ul>` will find
  one, so this should not misfire.
- **Contact information.** The source resume contains a street address
  (4000 Hulen St, APT 362, Fort Worth TX) and a phone number. **Both are
  deliberately omitted.** The repository is public and the deployment assignment
  adds a `robots.txt` that invites search engines to index the page, so
  publishing them would put a home address into search results permanently.
  Email and city/state are published. This was an explicit decision by the
  repository owner, not an oversight, and the footer states it on the page.

### Finding: the deployments are NOT byte-identical

The assignment's video requirement asks to show the live sites are "functional
and visually identical". All three are now live and serve the same commit, but
the delivered HTML differs — and **the outlier is GitHub Pages, not the others**:

```
repo file   7216 bytes   fd7d1b40…
netlify     7216 bytes   fd7d1b40…   identical to the repo
cloudflare  7216 bytes   fd7d1b40…   identical to the repo
pages       7471 bytes   7766b170…   <-- differs
```

That 2-against-1 split is itself the diagnosis. An earlier version of this note
framed Netlify as the exception, which had the cause backwards: the rewrite is a
**zone-level setting on `jinfusion.dev`**, not anything about Cloudflare as a host.
The `workers.dev` deployment is also Cloudflare and is completely unaffected,
because it is not inside that zone.

The diff is entirely the contact link:

```html
<!-- Netlify and Cloudflare both serve the file as authored -->
<a href="mailto:brunerjohnpeter@gmail.com">brunerjohnpeter@gmail.com</a>

<!-- edu.jinfusion.dev, rewritten in transit by Cloudflare -->
<a href="/cdn-cgi/l/email-protection#5735...">
  <span class="__cf_email__" data-cfemail="690b...">[email&#160;protected]</span></a>
<script data-cfasync="false" src="/cdn-cgi/scripts/5c5dd728/.../email-decode.min.js"></script>
```

`edu.jinfusion.dev` is proxied through Cloudflare with **Email Address
Obfuscation** (Scrape Shield) enabled. Cloudflare rewrites any `mailto:` it
finds and injects a script that restores it client-side.

**Consequences, stated plainly:**

- With JavaScript enabled — nearly all visitors, and certainly the grader — all
  three sites render the identical address. The video claim holds.
- **With JavaScript disabled, the Cloudflare copy shows the literal text
  `[email protected]`** where the email should be. On a resume that is a bad
  failure mode, and it is the one case where the two deployments are visibly
  different.
- The repo content is correct. This is a CDN transform, not a defect in the
  committed file, and it cannot be fixed by editing the HTML.

**Why I did not "fix" it.** The natural fix is the technique already used on
`jinfusion.dev` (`src/components/Email.astro`): keep the address out of the
markup and assemble it in JavaScript. That would require adding a script to this
page — and **R1 of this assignment permits only a single HTML file and a single
CSS file.** Fixing the CDN symptom would break a stated requirement.

The other route is a Cloudflare dashboard setting (Scrape Shield → Email Address
Obfuscation → off) for the zone. That is the owner's call, not mine, and there
is a reasonable argument for leaving it **on**: it is actively protecting a real
email address on a page that `robots.txt` invites search engines to index.

### Finding: the Cloudflare deploy published files that .gitignore excluded

`wrangler.jsonc` sets `"assets": { "directory": "." }`, which uploads the entire
repository root. Verified publicly readable on the live Worker:

```
/.git/config                             200
/.wrangler/cache/wrangler-account.json   200
```

`.wrangler/` is listed in `.gitignore` — and was published anyway. **`.gitignore`
governs git; it does not govern what Wrangler uploads.** Only a `.assetsignore`
does, and there wasn't one.

**Severity: low, but not zero — checked rather than assumed.**

- **No credentials.** The served `.git/config` carries no token in the remote URL
  (grepped for `ghp_`, `gho_`, `github_pat_` and `user:pass@` forms — zero hits).
- The account file holds a Cloudflare account **id** and account **name**. The id
  is an identifier, not a secret; it appears in every dashboard URL.
- **The real issue is the account name: it is an email address**, and a different
  one from the address deliberately published on the resume. So a second personal
  email was disclosed on a page whose own `robots.txt` invites indexing — directly
  against the privacy position taken elsewhere in this repo.
- Serving `.git/` from a web root leaks nothing here (the repo is public) but is a
  habit that leaks history and tokens the moment either of those changes.

**Fixed in commit `585937c`** by adding a `.assetsignore` excluding `.wrangler`,
`.git`, and the deploy config. **The fix is not live yet** — `.assetsignore` is
read by `wrangler deploy`, and this deployment is not git-connected, so a `git
push` does not apply it. It needs `npx wrangler deploy`, which is the repository
owner's action. Until then the URLs above still return `200`.

### What I executed vs. what I only reasoned about

**Executed:**
- `tools/audit_html.py` against `index.html` + `style.css`: 15 structural
  assertions, all passing (heading sequence, single `h1`, no inline `style=`,
  `lang`, viewport, non-empty title, no floats, Grid/Flexbox present, media
  query present, all seven landmarks present, `<time datetime>` present, `<dl>`
  present, experience section found, ≥2 articles inside it).
- Generated `og-preview.png` and inspected the rendered image visually. The
  first version was rejected — the hand-plotted monogram rendered a "J" that
  read as a lowercase "d" — and the design was replaced. The current generator
  carries `assert` statements that fail the build if any block overflows the
  card.

**Reasoned about but NOT executed — treat these as unverified claims:**
- **Contrast ratios.** The numbers quoted (14:1, 7.3:1, 6.5:1) are computed from
  the WCAG formula by hand, not measured with a tool. No axe, Lighthouse or
  WAVE run was performed — none is available in this environment.
- **Actual rendering.** No browser was opened. The layout is not confirmed
  visually at any viewport. Grid behaviour, the 52rem breakpoint, and the print
  stylesheet are all reasoned, not seen.
- **Screen-reader behaviour.** No NVDA/JAWS/VoiceOver pass. Landmark and heading
  claims rest on correct element usage, not on hearing them announced.
- **W3C validation.** `validator.w3.org` was not called. My checker is
  structural and is not a substitute for a real validator.
- **The OG preview card actually rendering** on LinkedIn/Slack/Discord. Tags are
  present and absolute; no scraper was invoked.
- **GitHub Pages serving.** At the time of writing, publication had not yet been
  confirmed with a live request.

### Edge cases known to be unhandled

- **Long unbroken strings.** `minmax(0, 2fr)` prevents column blowout, but no
  `overflow-wrap: anywhere` is set. A 60-character unspaced token would still
  overflow horizontally.
- **Very narrow viewports.** Tested by reasoning down to 320px, not observed.
  The nav is `flex-wrap: wrap` so it should reflow, but this is unconfirmed.
- **Print pagination.** `break-inside: avoid` is set on `<article>`, but no
  print preview was run. A long entry could still split awkwardly.
- **`prefers-contrast` / forced-colors mode.** Not handled at all.
- **No `lang` variation** for any non-English content — there is none, so this
  is moot, but worth stating.
- **The stale certification line.** The resume says "OCAJP Certification Expected
  Fall 2021", which is years in the past. It is reproduced **verbatim** because
  correcting a factual claim about someone's credentials is not mine to make.
  **This should be updated before the page is shown to an employer.**

### Three places I would look first if this turned out to be wrong

1. **The absolute URLs in `<head>`, `robots.txt` and `sitemap.xml`.** If the OG
   card is blank or the sitemap 404s, a mismatch between the real Pages URL and
   these hardcoded strings is the most likely cause — especially the trailing
   slash and the case of the repo name.

   **This did in fact happen.** The URLs were originally written as
   `https://jinfusion-edu.github.io/it102-semantic-html-resume/`. On enabling
   Pages, GitHub reported the site's address as
   `https://edu.jinfusion.dev/it102-semantic-html-resume/` — the
   `jinfusion-edu` org already owns a verified custom domain (`edu.jinfusion.dev`,
   set by the `CNAME` in `jinfusion-edu.github.io`), and **project** Pages sites
   inherit it. Every absolute URL across 11 files was rewritten to the real host.

   The `github.io` address still works: it returns `301` and redirects to the
   custom domain, so an old link is not broken. But `og:url` and
   `rel="canonical"` should name the final destination rather than a redirect,
   which is why they were changed rather than left.
2. **The `grid-row: 1 / span 4` on `#skills`.** It hardcodes the number of
   blocks in the content column. Adding or removing a `<section>` inside `<main>`
   will silently misalign the sidebar; nothing errors, it just looks wrong.
3. **The 52rem breakpoint.** Between roughly 45rem and 52rem the single-column
   layout gets wide, and line length may exceed comfortable reading measure
   despite `--measure: 68ch`. If someone reports "it looks stretched on a
   tablet", this is why.

### What I would flag reviewing this as someone else's code

- The colour custom properties carry contrast ratios in comments. Comments drift
  from reality. If someone tweaks `--accent` and does not recompute, the comment
  becomes a false claim about accessibility — worse than no comment.
- `og:description` and `twitter:description` differ slightly in wording. Not a
  defect, but they will drift further with editing; one source of truth would be
  better if a build step ever existed.
- The `.skip-link` uses `left: -9999px`. This works and is widely used, but the
  modern idiom is a clip-path/`position: absolute` visually-hidden utility. The
  older technique can cause horizontal-scroll artefacts in RTL contexts.
- `og-preview.png` is a generated geometric mockup, not a real screenshot or
  photograph. It satisfies "do not leave a placeholder" in the letter — a real
  image file, correct dimensions, committed — but a human would immediately see
  it is not a photo of anyone. **This is the weakest artefact in the repo and
  the first thing worth replacing.**

### Nothing found clean

I did not find this repo clean. The items above are real. The two I would most
want a second opinion on are the `Present`-not-in-`<time>` decision and whether
omitting the phone number will read to the grader as an incomplete contact block
rather than a deliberate privacy choice — which is precisely why the footer
states it on the page rather than only here.
