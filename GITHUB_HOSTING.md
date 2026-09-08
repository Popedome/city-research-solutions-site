# Hosting the City Research Solutions site on GitHub Pages
Setup guide | June 16, 2026

This site is plain static HTML, which is exactly what GitHub Pages serves best. There is no
build step.

## What is in this package
- index.html plus the other pages, and the images/ folder (the site itself)
- CNAME ........... tells GitHub Pages the custom domain is cityresearchsolutions.com (included)
- .nojekyll ....... tells GitHub Pages to serve files as-is, no Jekyll processing (included)
- README_FOR_DEPLOY.md ... general deploy notes
- GITHUB_HOSTING.md ...... this guide

## Step 1 - Put the site in a GitHub repository
1. Create a new GitHub repository. Public is fine (the site is public anyway).
   Suggested name: cityresearchsolutions
2. Upload the entire contents of this folder to the repo root. index.html must sit at the top
   level with images/ next to it. Keep the CNAME and .nojekyll files.

## Step 2 - Turn on GitHub Pages
1. Repo Settings > Pages.
2. Build and deployment: Source = "Deploy from a branch"; Branch = main; Folder = / (root). Save.
3. Custom domain: enter cityresearchsolutions.com and Save. (The CNAME file already sets this.)
4. Recommended: verify the domain (Settings > Pages, or the organization Pages settings) to
   protect against domain takeovers.

## Step 3 - Send Foremost the DNS records (see the note below)
These are GitHub's official GitHub Pages targets, confirmed from GitHub's documentation on
June 16, 2026. Setup is apex domain as primary, with a www redirect.

## Step 4 - Enforce HTTPS
After DNS propagates (can take up to 24 hours), return to Settings > Pages and check
"Enforce HTTPS". GitHub provisions a free certificate automatically.

------------------------------------------------------------------------------------------
NOTE TO SEND FOREMOST (fill in the GitHub account name first)
------------------------------------------------------------------------------------------
Hi team - we are moving cityresearchsolutions.com to GitHub Pages. Please set these DNS
records for cityresearchsolutions.com:

  A     @     185.199.108.153
  A     @     185.199.109.153
  A     @     185.199.110.153
  A     @     185.199.111.153

  AAAA  @     2606:50c0:8000::153
  AAAA  @     2606:50c0:8001::153
  AAAA  @     2606:50c0:8002::153
  AAAA  @     2606:50c0:8003::153

  CNAME www   <GITHUB-USERNAME-OR-ORG>.github.io.

Please also remove any existing A, ALIAS, or ANAME record on @ that points to the old site,
so it does not conflict. DNS changes can take up to 24 hours to propagate. Thanks!
------------------------------------------------------------------------------------------

Notes
- Replace <GITHUB-USERNAME-OR-ORG> with the GitHub account or organization that owns the repo
  (you will know it once the repo exists). The www CNAME must point to ACCOUNT.github.io with
  no repository name on the end.
- This replaces the old WordPress site. The DNS switch is the moment cityresearchsolutions.com
  starts showing the new site, and it is reversible.
- The site loads Tailwind CSS and the Inter font from CDNs at runtime, which works fine on
  GitHub Pages. If you want zero external dependencies later, we can compile a self-hosted CSS.
