# Hosting and deploying the City Research Solutions site
Corrected October 16, 2026. Supersedes the June 16, 2026 setup guide.

> **This file is named `GITHUB_HOSTING.md` and the site is NOT on GitHub Pages.**
> The name is kept deliberately: the old contents of this file described a GitHub Pages
> setup that was planned but never used, and anyone who goes looking for deploy
> instructions tends to find this filename first. Renaming it would leave that trap
> open. The correct instructions are below.

## The one thing to know

**Pushing to GitHub does not publish this site.** The GitHub repo
(`Popedome/city-research-solutions-site`) is a record of the source only. The live site is a
**Cloudflare Pages project deployed by direct upload**. If you push and walk away, nothing
reaches cityresearchsolutions.com.

This is not hypothetical. On 2026-10-15 a change was committed and pushed to `main`
exactly as the old version of this file described. The push succeeded, the commit is in
history, and **nothing was published**. It was caught only because the verification step
compared bytes instead of status codes (see "Verify by bytes" below).

## How it is actually hosted

| | |
|---|---|
| Platform | Cloudflare Pages |
| Project name | `city-research-solutions` |
| Cloudflare account | `6d3aed9710c4aca44a4a57f5419e9659` (Popedome@gmail.com) |
| **Git provider** | **None.** The project is not connected to GitHub. |
| Deploy method | `wrangler pages deploy` (direct upload) |
| Domains | cityresearchsolutions.com, www.cityresearchsolutions.com, city-research-solutions.pages.dev |
| DNS | Cloudflare proxied records on the apex and www. No GitHub Pages A/AAAA records exist. |
| Credentials | `CLOUDFLARE_API_TOKEN` is already set in the environment; wrangler picks it up. |

Verify the git-provider claim yourself at any time:

```bash
npx wrangler pages project list     # "Git Provider" column reads No
```

## Deploying

The site is plain static HTML with no build step. Deploy a **clean staging copy**, not the
repo folder itself, so repo-only files are never published.

```bash
# 1. Build a staging copy that excludes repo-only files
STAGE=../_crs_stage_$(date +%Y-%m-%d)
rm -rf "$STAGE" && mkdir -p "$STAGE"
tar -cf - --exclude='./.git' --exclude='./CNAME' \
          --exclude='./GITHUB_HOSTING.md' --exclude='./.gitattributes' . \
  | (cd "$STAGE" && tar -xf -)

# 2. Upload
npx wrangler pages deploy "$STAGE" \
  --project-name=city-research-solutions \
  --branch main --commit-dirty=true
```

Excluded on purpose:

- `.git` - never publish repository internals.
- `CNAME` - a GitHub Pages artifact. Cloudflare does not use it; the custom domain is
  configured on the Pages project.
- `GITHUB_HOSTING.md`, `.gitattributes` - repo documentation and config, not site content.

**Before uploading, reconcile the staging copy against the previously deployed tree** and
confirm the only differences are the ones you intended, that no file was dropped, and that
nothing unexpected changed. A direct upload replaces the whole site, so a file missing from
staging is a file deleted from the live site.

## Verify by bytes, not status codes

**Any unknown path on this domain returns HTTP 200 with the homepage body.** A status-code
check will report a clean deploy for a site where nothing deployed. Measured 2026-10-16:

```
GET /this-does-not-exist-xyz.png  ->  200, content-type: text/html, 41097 bytes
```

Note it returns **HTML for a `.png` request**. That is the tell.

So verification compares the live response against the file you deployed:

```bash
# an asset: must be byte-identical
curl -sL -o /tmp/live.png "https://cityresearchsolutions.com/images/pricing/pricing-van-westendorp.png"
cmp /tmp/live.png images/pricing/pricing-van-westendorp.png && echo IDENTICAL

# a page: assert a marker unique to this deploy, and that it is not the homepage
curl -sL "https://cityresearchsolutions.com/pricing-research-methods" | grep -c "Bracket, pick, then stress test"
```

Two further traps when checking HTML pages:

1. **`.html` paths 308-redirect to clean URLs** (`/about.html` -> `/about`). Without
   `curl -L` the body is 0 bytes, which looks like a different failure than the real one.
2. **Cloudflare Email Obfuscation inflates any HTML page containing a `mailto:` by about
   206 bytes**, rewriting the link to `/cdn-cgi/l/email-protection#<hex>` and injecting a
   decoder script. The hex token changes per response, so two fetches of the same correct
   page are not byte-identical. Compare markers on HTML; reserve byte-equality for assets.

## What is in this repo

- `index.html` and the other pages, plus `images/` - the site itself.
- `llms.txt`, `robots.txt`, `sitemap.xml` - add new pages to `sitemap.xml` and `llms.txt`.
- `CNAME` - a leftover from the GitHub Pages plan. Not used, not deployed.
- `GITHUB_HOSTING.md` - this file.

There is no `.nojekyll` and no `README_FOR_DEPLOY.md`; the June version of this file listed
both as included and neither has ever existed here.

## Normal workflow

1. Make the change and confirm `git status` shows only what you intended.
2. Commit and push - this records the source. **It does not publish.**
3. Build the staging copy, reconcile it, and run `wrangler pages deploy`.
4. Verify at the live domain by bytes/markers. Only then is the change shipped.
