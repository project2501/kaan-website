# Kaan Bardak — Personal Website & Blog

My personal website and blog, built as a **static site with [Hugo](https://gohugo.io/)**
(the [Toha](https://toha-guides.netlify.app/) theme) and deployed to **AWS S3**
through **GitHub Actions** on every push to `main`.

<!-- The two values below are updated automatically by the CI/CD pipeline after each deploy. -->
![Build and Deploy](https://github.com/project2501/kaan-website/actions/workflows/deploy.yml/badge.svg)

- **Last published:** <!-- last-published:start -->2026-06-02 14:16 UTC<!-- last-published:end -->
- **Latest post:** <!-- latest-post:start -->Pattern Languages<!-- latest-post:end -->

> ℹ️ Replace `project2501` in the badge URL above with your GitHub username/org if it differs.

---

## About this site

- **What it is:** a personal portfolio landing page + a blog.
- **How it's built:** Markdown content → static HTML via Hugo. No server, no database.
- **Theme:** Toha, pulled in as a **Hugo Module** (no git submodule to manage).
- **Hosting:** the generated `public/` folder is synced to an AWS S3 bucket.
- **CI/CD:** `.github/workflows/deploy.yml` builds and deploys automatically.

### Tech stack & prerequisites

| Tool | Why it's needed | Version used |
|------|-----------------|--------------|
| **Hugo (extended)** | Static site generation + SCSS | `0.156.0` |
| **Go** | Toha is a Hugo Module, fetched via Go | `1.25` |
| **Node.js + npm** | Toha's assets (fonts, icons, KaTeX, Bootstrap) | `20.x` |

---

## Running locally

```bash
# 1. Install the npm assets (fonts/icons/KaTeX/etc.)
npm install

# 2. Start the dev server (-D includes draft posts), then open http://localhost:1313
hugo server -D
```

The first run downloads the Toha module via Go automatically. Edit any Markdown or
data file and the browser live-reloads.

### Writing a post

```bash
hugo new content posts/my-new-post.md
```

Then edit the file in `content/posts/`. Set `draft = false` in the front matter to
publish it (drafts are excluded from the deployed site).

### Repo layout (the parts you'll touch)

```
hugo.yaml                 # site config (title, baseURL, features)
content/posts/            # your blog posts (Markdown)
data/en/author.yaml       # hero: name, photo, taglines, contact
data/en/site.yaml         # copyright, meta description, OpenGraph
data/en/sections/*.yaml   # homepage sections (about, skills, certifications, …)
assets/images/            # site logo/background + section/author images
.github/workflows/        # CI/CD (build + deploy to S3)
```

---

## 🍴 Fork & deploy your own

This repo doubles as a template: **fork it, swap in your own details, point it at your
own host, and you have a similar site.** Steps:

### 1. Get it running
1. **Fork** the repo on GitHub and clone your fork.
2. Install prerequisites (Hugo extended, Go, Node — see table above).
3. `npm install` then `hugo server -D` to preview locally.

### 2. Make it yours — files to **edit**
| File | Change |
|------|--------|
| `hugo.yaml` | `baseURL`, `title`, `languageCode`; toggle features in the `features:` block |
| `data/en/author.yaml` | name, nickname, greeting, photo path, contact info, rotating taglines |
| `data/en/site.yaml` | copyright, meta `description`, OpenGraph title/URL |
| `data/en/sections/about.yaml` | designation, summary, social links, skill bars (`badges`) |
| `data/en/sections/skills.yaml` | your skills (+ logos in `assets/images/sections/skills/`) |
| `data/en/sections/certifications.yaml` | your certifications |
| `assets/images/site/` | `background.jpg`, `main-logo.png`, `inverted-logo.png`, `favicon.png` |
| `assets/images/author/` | add your photo, then update the `image:` path in `author.yaml` |

### 3. Files to **remove / replace**
- `content/posts/my-first-post.md` — the sample post; replace with your own.
- `assets/images/author/john.png`, `jessica.png` — demo author photos.
- Any section you don't want: delete its `data/en/sections/<name>.yaml`
  (or set `enable: false` inside it). Sections must keep their original `id:`
  because the theme picks the rendering template by `id`.

### 4. Wire up your own deployment
This repo deploys to **AWS S3**. To reuse it as-is, add these **GitHub Actions secrets**
(Settings → Secrets and variables → Actions):

| Secret | Purpose |
|--------|---------|
| `AWS_ACCESS_KEY_ID` | AWS credentials for the deploy |
| `AWS_SECRET_ACCESS_KEY` | … |
| `AWS_REGION` | bucket region |
| `S3_BUCKET_NAME` | target bucket |

Prefer a different host? Edit or replace `.github/workflows/deploy.yml` — e.g. swap the
S3 step for GitHub Pages, Netlify, or Cloudflare Pages. If you don't need CI deploys at
all, delete the workflow and run `hugo --minify` yourself.

---

## Notes

#### Hugo Related links
- [Quick Start](https://gohugo.io/getting-started/quick-start/)
- [host and deploy](https://gohugo.io/host-and-deploy/)

- [This is last Theme that I review, continue from here](https://themes.gohugo.io/themes/theme-search/)

#### Interesting Themes

- [SaaSify](https://saasify-demo.chaoming.li/pricing/)
- [Up Business](https://writeonlycode.github.io/hugo-up-business/)
- [Agency Web](https://writeonlycode.github.io/hugo-agency-web/)
- [Nomad Tech](https://m03315.github.io/nomad-tech/posts/)
- [Kawaii Theme](https://hugo-kawaii.pages.dev/posts/)
- [LoveIt](https://hugoloveit.com/)
- ⭐ [Toha](https://toha-docs.hugo-themes.com/#)
