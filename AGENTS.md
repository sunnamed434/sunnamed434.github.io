# AGENTS.md

Guidance for AI agents (Claude Code, Cursor, etc.) working in this repo. Read this before adding or editing content.

## What this is

A personal blog built with **Jekyll + the Chirpy theme**, deployed to **GitHub Pages** from the `main` branch (see `.github/workflows/pages-deploy.yml`). Everything is written in **English**. Core topics: reverse engineering, .NET architecture, obfuscation/deobfuscation, Mono/Unity internals — plus local AI / mini-PC hardware.

## Writing a new post

### File & front matter

- Create `_posts/YYYY-MM-DD-slug.md`. The public URL becomes `/posts/<slug>/` (permalink is `/posts/:title/`, where `:title` is the filename slug after the date — **not** the `title:` field).
- Match the front matter the existing posts use — nothing more, nothing less:

  ```yaml
  ---
  layout: post
  title: "Your Title"
  description: One-sentence summary. Used by the SEO/OpenGraph meta tags AND copied into the llms.txt entry.
  date: YYYY-MM-DD HH:MM:SS +0300
  categories: space separated lowercase tags
  ---
  ```

- Images are optional. If you add any, put them under `assets/images/<slug>/` and reference `/assets/images/<slug>/...`. **Never invent image paths** — a missing file is a broken image. Chirpy supports `.dark`/`.light` variants and `{: width="700" height="400" }` sizing.
- Use Chirpy callouts where they help: a blockquote followed by `{: .prompt-tip }` (also `.prompt-info`, `.prompt-warning`, `.prompt-danger`).

### Voice & style (this matters a lot)

- First-person, casual, opinionated, slightly non-native English. Run-on sentences, asides, the occasional `:D` / `:)`. Calibrate against existing `_posts/` (e.g. the dnSpy and "No one pays me for the Unit Tests" posts).
- **Do not let it read as AI-generated.** No generic filler, no over-symmetric "on one hand / on the other hand", no em-dash-heavy polished copy. Keep concrete numbers, real first-hand anecdotes, and the author's actual opinions/jokes.
- **Do not fabricate.** Verify technical facts with real web research before stating them. If a detail is missing, **ask the author** — don't make it up. Hedge anything you haven't personally measured ("reportedly", "the paper claims", and date-stamp benchmark numbers — "as of <month/year>… these will have moved").
- Link named tools / products / GitHub issues the first time they appear, and keep org names consistent across links.

### SEO checklist — do this for EVERY new post

Most SEO is automatic at build time. Exactly one file is hand-maintained, and it is easy to forget:

- ⚠️ **`assets/llms.txt` — UPDATE IT MANUALLY.** Add the new post to the `## Posts` list, **at the very top** (the list is newest-first):

  ```
  - [Exact Title](https://sunnamed434.github.io/posts/<slug>/): one-line description.
  ```

- Auto-generated, **no action needed**:
  - `assets/feed.xml` — Atom feed, loops over `site.posts`.
  - sitemap (`/sitemap.xml`) — via `jekyll-sitemap` (bundled with Chirpy); referenced by `robots.txt`.
  - `assets/robots.txt` — static; already allows AI crawlers (GPTBot, ClaudeBot, PerplexityBot, etc.).
  - per-page SEO / OpenGraph tags — `jekyll-seo-tag`, built from `title` + `description` + the site config.
  - `last_modified_at` — set from git history by `_plugins/posts-lastmod-hook.rb` once a post has more than one commit.

## Git / publishing

- **Do not commit or push unless the author explicitly asks.** Writing and editing files is fine; committing/pushing is the author's call.
- Pushing to `main` publishes the site live via GitHub Pages (takes ~1-2 min to deploy).
- When you do commit, keep the message style consistent with the repo and append the project's `Co-Authored-By:` trailer.

## Build / preview locally

- `bundle exec jekyll serve` (Ruby version is pinned in `.ruby-version`; `webrick`, Windows gems, and `jekyll-compose` are already in the `Gemfile`).

## Notes for the next agent (lessons learned)

- The post URL slug comes from the **filename**, so name the file carefully — that's the permanent URL.
- The single most-forgotten step is updating **`assets/llms.txt`**. If you add a post and skip it, the live `llms.txt` silently drifts out of sync with the blog. Always do it in the same change as the post.
- `categories` are space-separated and each one generates an archive page (via `jekyll-archives`); existing posts use 2-6 of them, so that range is fine.
- Don't add fields the existing posts don't use (no `author`, `image`, `toc`, `pin` unless there's a reason) — keep front matter minimal and consistent.
