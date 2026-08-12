# Deployment — the three platforms

The assignment requires the same codebase live on three hosts. One is done;
two need an account authorisation that only you can give.

| Platform | Status | URL |
|---|---|---|
| GitHub Pages | **done** — enabled from this repo | `https://edu.jinfusion.dev/it102-semantic-html-resume/` |
| Netlify | **done** — git-connected | `https://resumejinfusion.netlify.app/` |
| Cloudflare | **done** — `wrangler deploy` | `https://it102-semantic-html-resume.jeskeyejin.workers.dev/` |

All three are live. Each was verified serving `200` for the page, `style.css`,
`robots.txt`, `og-preview.png`, `favicon.svg` and `sitemap.xml`.

> **Two things about the Cloudflare one that affect the video.**
>
> 1. It is **not git-connected** — it was deployed from the CLI with
>    `wrangler deploy`. A `git push` rebuilds Pages and Netlify but does
>    **nothing** to Cloudflare. Use **Netlify** for the automated-build shot in
>    step 3 below, and run `npx wrangler deploy` on camera if you want to show
>    Cloudflare updating.
> 2. It is a **Worker with static assets**, not Cloudflare Pages. Cloudflare has
>    consolidated Pages into Workers for new static projects, so the URL is
>    `*.workers.dev` rather than `*.pages.dev`. Same product family, same
>    git-backed hosting story the assignment is teaching — worth saying out loud
>    in the video, since the assignment says "Cloudflare Pages" by name.

Nothing about the code changes between platforms. All three serve this repo's
`main` branch from the repository root, which is exactly the point the
assignment is making about Git-backed continuous deployment.

---

## Platform 2 — Netlify (Git integration)

1. Go to <https://app.netlify.com/signup> and sign in **with GitHub**.
2. **Add new site → Import an existing project → Deploy with GitHub**.
3. Authorise Netlify for the **`jinfusion-edu`** organisation. This is the step
   I cannot do for you — it grants a third party access to the org.
   If the org does not appear, click *Configure the Netlify app on GitHub* and
   grant it access to `jinfusion-edu` repositories.
4. Pick **`it102-semantic-html-resume`**. Leave the build settings empty:
   - Build command: *(blank)* — there is no build step, this is plain HTML/CSS
   - Publish directory: `.` (the repository root)
5. **Deploy site.** Netlify assigns a random name like
   `resplendent-hamster-1a2b3c.netlify.app`. Rename it under
   **Site configuration → Change site name** to something like
   `john-bruner-resume`, then copy the final URL into the table above.

## Platform 3 — Cloudflare  *(done — recorded for the write-up)*

This was deployed from the CLI rather than the dashboard:

```bash
npx wrangler deploy      # reads wrangler.jsonc in this repo
```

`wrangler.jsonc` sets `"assets": { "directory": "." }`, which serves the repo
root. A `.assetsignore` sits beside it to keep `.git/` and `.wrangler/` out of
the upload — see `AUDIT-NOTES.md` for why that file exists and what was exposed
before it did. **Re-run `npx wrangler deploy` to apply it; it is not live yet.**

<details>
<summary>The original dashboard route, if you ever want git-connected builds</summary>

1. Go to <https://dash.cloudflare.com> → **Workers & Pages → Create → Pages →
   Connect to Git**.
2. Authorise Cloudflare for the **`jinfusion-edu`** organisation — again, yours
   to grant.
3. Select **`it102-semantic-html-resume`**.
4. Build settings:
   - Framework preset: **None**
   - Build command: *(blank)*
   - Build output directory: `/`
5. **Save and Deploy.** The URL is `<project>.pages.dev`. Copy it into the
   table above.

</details>

> You already own the `jinfusion.dev` zone in Cloudflare. Do **not** point a
> custom domain at this project unless you want to — the assignment only asks
> for the three platform URLs, and the existing `edu.jinfusion.dev` record for
> the org Pages site must be left alone.

---

## Recording the video (submission items 2–5)

The assignment wants four specific things on camera. Do them in this order so
the build has time to finish while you are still talking.

1. **The three live sites.** Open all three URLs in tabs and show they are
   functional and visually identical.

2. **The "git push" proof.** Show your terminal or VS Code and make one small,
   safe edit. Use the footer line in `index.html`:

   ```
   <p>&copy; 2026 John Bruner. All rights reserved.</p>
   ```

   Change `2026` to `2026 ·` plus today's date, or edit the privacy sentence.
   Then:

   ```bash
   git add index.html
   git commit -m "A3: demo edit for deployment walkthrough"
   git push origin main
   ```

3. **The automated build.** Switch to the Netlify or Cloudflare dashboard and
   show it catching the commit and building in real time. Netlify's
   **Deploys** tab updates within a few seconds.

4. **Meta tag proof.** On the live site press **F12 → Elements**, expand
   `<head>`, and show the `og:title` / `og:image` / `canonical` tags and the
   favicon link. Then visit `/robots.txt` directly to show it serves.

A useful extra: paste the live URL into <https://www.opengraph.xyz> to show
the preview card rendering. Not required, but it demonstrates the OG tags
actually work rather than merely existing.

---

## Verifying GitHub Pages yourself

```bash
curl -sI https://edu.jinfusion.dev/it102-semantic-html-resume/ | head -1
curl -s  https://edu.jinfusion.dev/it102-semantic-html-resume/robots.txt
```

The first should report `HTTP/2 200`. Pages can take a couple of minutes to
publish after the first push, and returns 404 until it does.
