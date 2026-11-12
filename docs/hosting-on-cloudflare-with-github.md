# Hosting a Website on Your Cloudflare Domain with GitHub

## Overview

Use Cloudflare's GitHub integration (Workers & Pages): every `git push` to `main` rebuilds and publishes your site on your domain, with free HTTPS and no server to manage. The whole setup takes about 15 minutes.

| Approach | How it works | Pick it when |
| --- | --- | --- |
| **Cloudflare Workers & Pages** (recommended) | Cloudflare pulls from your GitHub repo and serves the site from its own network | Your domain already lives on Cloudflare — fewest moving parts, preview URLs per branch |
| **GitHub Pages + Cloudflare DNS** | GitHub hosts the files; Cloudflare DNS points your domain at GitHub | You already use GitHub Pages or want everything visible on GitHub |

This guide uses the domain `stevejpeters.org` and the GitHub account `stephenjonpeters`.

**Reference docs:** [Cloudflare Pages docs](https://developers.cloudflare.com/pages/) · [GitHub Pages docs](https://docs.github.com/en/pages)

## Prerequisites

- [ ] A Cloudflare account with your domain added and its status showing **Active** (Cloudflare dashboard → your domain → Overview)
  - Cloudflare: [Dashboard](https://dash.cloudflare.com) · [Add a domain to Cloudflare](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/)
- [ ] A GitHub account
  - GitHub: [Sign up](https://github.com/signup)
- [ ] Git installed locally — check with `git --version` in Terminal
  - [Download Git](https://git-scm.com/downloads) · GitHub: [Set up Git](https://docs.github.com/en/get-started/getting-started-with-git/set-up-git)
- [ ] Optional: the GitHub CLI (`brew install gh`, then `gh auth login`) to create repos from the terminal
  - GitHub: [GitHub CLI](https://cli.github.com/)

## Step 1: Put your site in a GitHub repository

> **Links** — GitHub: [Create a new repository](https://github.com/new) · [Creating a repository (docs)](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) · [Adding a local project to GitHub](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github) · [`gh repo create`](https://cli.github.com/manual/gh_repo_create)
> Cloudflare: [Static HTML on Pages](https://developers.cloudflare.com/pages/framework-guides/deploy-anything/)

A static site only needs an `index.html` at the root of the repo (or in a folder like `public/` or `dist/`).

1. Create a project folder with a starter page:

```bash
mkdir my-site && cd my-site
git init -b main
cat > index.html <<'EOF'
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>My Site</title>
  </head>
  <body>
    <h1>Hello from my domain!</h1>
  </body>
</html>
EOF
git add . && git commit -m "Initial site"
```

2. Create the GitHub repo and push. With the GitHub CLI:

```bash
gh repo create my-site --public --source=. --push
```

Or without it: on [github.com/new](https://github.com/new) create a repository, leave it empty (no README), then run:

```bash
git remote add origin https://github.com/stephenjonpeters/my-site.git
git push -u origin main
```

The repo can be private for the Cloudflare approach; GitHub Pages needs it public on a free plan.

## Step 2: Connect the repo to Cloudflare

> **Links** — Cloudflare: [Workers & Pages dashboard](https://dash.cloudflare.com/?to=/:account/workers-and-pages) · [Pages Git integration](https://developers.cloudflare.com/pages/get-started/git-integration/) · [Workers Builds (Git integration)](https://developers.cloudflare.com/workers/ci-cd/builds/) · [Build configuration & framework presets](https://developers.cloudflare.com/pages/configuration/build-configuration/)
> GitHub: [Installed GitHub Apps (manage Cloudflare's repo access)](https://github.com/settings/installations) · [About GitHub Apps](https://docs.github.com/en/apps/using-github-apps/about-using-github-apps)

Cloudflare builds and deploys the site on every push once the repo is linked.

1. In the Cloudflare dashboard, open **Workers & Pages** → **Create**.
2. Choose **Import a repository** (on the Pages tab this is **Connect to Git**).
3. Click **Connect GitHub**, authorize the Cloudflare app, and grant it access to just the `my-site` repo.
4. Select the repo and fill in the build settings:

| Setting | Plain HTML site | Framework site (e.g. Astro, Vite, Hugo) |
| --- | --- | --- |
| Production branch | `main` | `main` |
| Build command | leave empty | e.g. `npm run build` |
| Output / build directory | `/` (or the folder holding `index.html`) | e.g. `dist` or `public` |

5. Click **Save and Deploy**. After a minute you get a live URL such as `my-site.pages.dev` or `my-site.stephenjonpeters.workers.dev` — open it to confirm the page loads.

If the dashboard offers a framework preset, pick it; it fills in the command and folder for you.

## Step 3: Attach your domain

> **Links** — Cloudflare: [Pages custom domains](https://developers.cloudflare.com/pages/configuration/custom-domains/) · [Workers custom domains](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/) · [Manage DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/) · [Redirect rules](https://developers.cloudflare.com/rules/url-forwarding/single-redirects/)

Because the domain is already on Cloudflare, it creates the DNS records and the HTTPS certificate for you.

1. Open your new project → **Settings** → **Domains & Routes** (Pages: the **Custom domains** tab).
2. Click **Add** → **Custom domain**, enter `stevejpeters.org`, and confirm.
3. Repeat for `www.stevejpeters.org`.
4. Wait for each domain to show **Active** — usually 1–5 minutes.
5. Visit `https://stevejpeters.org` and `https://www.stevejpeters.org`.

To send `www` to the bare domain (or the reverse), add a redirect: your domain → **Rules** → **Redirect Rules** → template **Redirect from WWW to root**.

If an existing A, AAAA or CNAME record already uses `stevejpeters.org` or `www` (for example a parking page), delete it under **DNS → Records** first, or the custom domain step will fail.

## Alternative: GitHub Pages with Cloudflare DNS

> **Links** — GitHub: [Configure a publishing source](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) · [Manage a custom domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) · [Secure your site with HTTPS](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https) · [Verify your domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)
> Cloudflare: [Manage DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/) · [Proxy status (orange vs grey cloud)](https://developers.cloudflare.com/dns/proxy-status/) · [SSL/TLS encryption modes](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/)

Here GitHub serves the files and Cloudflare only points the domain at GitHub. Skip Steps 2–3 if you use this.

1. In the GitHub repo: **Settings** → **Pages** → Source **Deploy from a branch** → branch `main`, folder `/ (root)` → **Save**.
2. On the same page, set **Custom domain** to `stevejpeters.org` and save. GitHub commits a `CNAME` file to the repo — keep it.
3. In Cloudflare → **DNS** → **Records**, add:

| Type | Name | Content | Proxy |
| --- | --- | --- | --- |
| A | `@` | `185.199.108.153` | DNS only (grey cloud) |
| A | `@` | `185.199.109.153` | DNS only |
| A | `@` | `185.199.110.153` | DNS only |
| A | `@` | `185.199.111.153` | DNS only |
| CNAME | `www` | `stephenjonpeters.github.io` | DNS only |

   Confirm the current IP addresses in GitHub's [apex domain instructions](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain).

4. Back in GitHub Pages settings, wait for the DNS check to pass, then tick **Enforce HTTPS**.
5. Optional: once HTTPS works, switch the records to **Proxied** (orange cloud) and set Cloudflare **SSL/TLS** mode to **Full**. Never use **Flexible** with GitHub Pages — it causes an endless redirect loop.

Keep the records grey-clouded until GitHub issues its certificate; the proxy blocks GitHub's domain check.

## Updating the site and troubleshooting

> **Links** — Cloudflare: [Preview deployments](https://developers.cloudflare.com/pages/configuration/preview-deployments/) · [Rollbacks](https://developers.cloudflare.com/pages/configuration/rollbacks/) · [Purge cache](https://developers.cloudflare.com/cache/how-to/purge-cache/) · [Pages troubleshooting](https://developers.cloudflare.com/pages/platform/known-issues/)
> GitHub: [Pushing commits](https://docs.github.com/en/get-started/using-git/pushing-commits-to-a-remote-repository) · [Troubleshooting custom domains](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages)

Edit files, then commit and push — the live site updates within about a minute:

```bash
git add . && git commit -m "Update homepage" && git push
```

With Cloudflare, pushing any other branch creates a separate preview URL, so you can check changes before merging to `main`. Each deploy is listed in the project's **Deployments** tab, where you can roll back to an earlier one.

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Custom domain stuck on **Pending** / fails | Old DNS record on the same name | Delete the conflicting A/AAAA/CNAME in DNS → Records, then retry |
| 404 on the live URL | Wrong output directory | Set it to the folder that contains `index.html` and redeploy |
| Build fails | Wrong build command or missing dependencies | Read the build log in the deployment; run the same command locally |
| "Too many redirects" (GitHub Pages) | SSL/TLS mode set to Flexible | Set SSL/TLS to **Full** |
| Old content still showing | Browser or Cloudflare cache | Hard refresh; or **Caching** → **Purge Everything** |
| GitHub Pages DNS check fails | Records are proxied (orange cloud) | Switch to DNS only until the certificate is issued |
