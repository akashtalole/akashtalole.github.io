# CLAUDE.md — AI Assistant Guide for akashtalole.github.io

## Repository Overview

Personal blog/portfolio for Akash Talole, built with **Jekyll** and the **Chirpy theme** (~7.5), hosted on **GitHub Pages** at `https://akashtalole.github.io`.

- **Author**: Akash Talole — Lead AI Engineer, 11 years of industry experience
- **Content focus**: Practical AI engineering — Claude Code, GitHub Copilot, Microsoft Copilot Studio, Agentic AI, Coding Agents, Agent Skills, Claude CoWork, AI in SDLC
- **Deployment**: GitHub Actions builds the site and deploys to GitHub Pages on push to `main`/`master`

---

## Directory Structure

```
.
├── _config.yml          # Main Jekyll + Chirpy configuration
├── Gemfile              # Ruby gem dependencies
├── index.html           # Homepage (minimal; Jekyll processes it)
├── _posts/              # Blog posts: YYYY-MM-DD-slug.md
├── _tabs/               # Navigation pages (about, archives, categories, tags)
├── _data/               # YAML data files (contact.yml, share.yml)
├── _plugins/            # Ruby plugins (posts-lastmod-hook.rb)
├── assets/
│   ├── img/             # Images (avatar.jpg, post images)
│   └── lib/             # Git submodule: chirpy-static-assets (do not edit)
├── tools/
│   ├── run.sh           # Local dev server
│   └── test.sh          # Build + HTML validation
├── .github/workflows/   # GitHub Actions CI/CD (pages-deploy.yml)
└── .devcontainer/       # Dev container config (Jekyll + zsh)
```

---

## Development Workflows

### Setup

```bash
bundle install   # Install Ruby dependencies (first time or after Gemfile changes)
```

### Local Development

```bash
bash tools/run.sh             # Serve at http://127.0.0.1:4000 with live reload
bash tools/run.sh -H 0.0.0.0  # Bind to all interfaces (Docker/devcontainer)
bash tools/run.sh -p          # Production mode
```

The VS Code task **"Run Jekyll Server"** (Ctrl+Shift+B) calls `tools/run.sh`.

### Build & Test

```bash
bash tools/test.sh   # Build to _site/ + run htmlproofer (HTML validation)
```

Run this before pushing to catch broken internal links. The VS Code task **"Build Jekyll Site"** calls `tools/test.sh`.

### CI/CD

Pushing to `main` or `master` triggers `.github/workflows/pages-deploy.yml`:
1. Builds with `bundle exec jekyll b`
2. Validates HTML with `htmlproofer` (external links excluded)
3. Deploys artifact to GitHub Pages

---

## Content Strategy & Voice

### Author Background
Akash is a Lead AI Engineer with 11 years of experience. Posts are written from the perspective of someone actively using these tools on real enterprise projects — not reviewing them from the outside.

### Tone & Style
- **Humanized, direct, no-fluff** — write like a senior engineer explaining something to a peer, not a tutorial site
- **Practical over theoretical** — lead with real use cases, real friction, real outcomes
- **Honest** — acknowledge limitations and failure modes; don't oversell tools
- **Audience**: engineers who already code for a living and are past "should I try AI?" — they're in the "how do I use this well?" phase
- Avoid: marketing language, excessive disclaimers, generic intros, bullet-point padding

### Active Content Series: 30-Day AI Engineering Plan

A pinned series (`_posts/2026-04-09-30-day-ai-engineering-blog-plan.md`) maps out 30 daily posts across five arcs. **Before creating any new post, check this plan to avoid duplicates and maintain arc continuity.**

| Arc | Days | Theme |
|-----|------|-------|
| 1 | 1–6 | Foundations & the AI-Augmented Engineer |
| 2 | 7–13 | Claude Code for Enterprise |
| 3 | 14–19 | GitHub Copilot Mastery |
| 4 | 20–25 | Coding Agents & Agent Skills |
| 5 | 26–30 | Microsoft Copilot Studio & Multi-Agent Solutions |

### Approved Tags & Categories

Use these consistently — do not invent new tags without a clear reason:

| Topic | Category | Tags |
|-------|----------|------|
| Claude Code | `ai`, `claude-code` | `claude-code`, `enterprise`, `coding-agents` |
| Claude CoWork | `ai`, `claude-code` | `claude-cowork`, `pair-programming` |
| GitHub Copilot | `ai`, `github-copilot` | `github-copilot`, `copilot-workspace` |
| Copilot Studio | `ai`, `copilot-studio` | `copilot-studio`, `multi-agent`, `microsoft` |
| Agentic AI | `ai`, `agentic-ai` | `agentic-ai`, `agents` |
| Agent Skills | `ai`, `agent-skills` | `agent-skills`, `tool-use` |
| AI in SDLC | `ai`, `sdlc` | `sdlc`, `ai-in-sdlc` |
| Coding Agents | `ai`, `coding-agents` | `coding-agents`, `automation` |
| Meta / Series | `ai`, `meta` | `meta` |

---

## Creating Blog Posts

### File Naming

Place files in `_posts/` using the convention: `YYYY-MM-DD-slug-with-hyphens.md`

Example: `_posts/2026-04-09-my-new-post.md`

### Required Front Matter

```yaml
---
layout: post
title: "Post Title Here"
date: 2026-04-09
categories: [category1, category2]
tags: [tag1, tag2]
description: "Brief description for SEO and post previews."
author: akashtalole
---
```

> **Note on dates**: Use `YYYY-MM-DD` only — no time or timezone suffix. Adding a time (e.g. `08:00:00 +0530`) can cause Jekyll to silently skip the post if the CI build runs before that time in UTC. `_config.yml` sets `future: true` as a safety net, but the date-only format is the safest convention.

### Optional Front Matter

```yaml
image:
  path: /assets/img/post-image.jpg
  alt: "Image description"
pin: true          # Pin post to top of home page
math: true         # Enable LaTeX math rendering
mermaid: true      # Enable Mermaid diagram rendering
toc: false         # Disable table of contents (enabled globally by default)
comments: false    # Disable comments for this post
```

### Notes on `last_modified_at`

Do **not** set `last_modified_at` manually. The plugin `_plugins/posts-lastmod-hook.rb` reads Git history and sets it automatically for posts with more than one commit.

---

## Configuration Reference (`_config.yml`)

Key settings:

| Setting | Value | Notes |
|---|---|---|
| `url` | `https://akashtalole.github.io` | No trailing slash |
| `timezone` | `Asia/Kolkata` | Use `+0530` in post dates |
| `paginate` | `10` | Posts per page |
| `toc` | `true` | TOC enabled globally for posts |
| `permalink` | `/posts/:title/` | Post URL structure |
| `pwa.enabled` | `true` | PWA + offline cache enabled |
| `comments.provider` | *(empty)* | Comments disabled; set to `disqus`, `utterances`, or `giscus` to enable |

---

## Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Blog posts | `YYYY-MM-DD-slug.md` | `2026-04-09-agent-skills.md` |
| Categories/tags | lowercase, hyphen-separated | `ai`, `agent-skills` |
| Images | lowercase, hyphen-separated | `avatar.jpg`, `post-header.png` |
| Shell scripts | `.sh` | `run.sh`, `test.sh` |
| YAML config | lowercase, snake_case | `_config.yml` |

---

## Key Constraints

1. **`assets/lib` is a Git submodule** pointing to `chirpy-static-assets`. Never edit files inside it directly; run `git submodule update --remote` to update it.

2. **`.nojekyll` is intentional**. GitHub Pages deploys the pre-built `_site/` artifact produced by CI — it does not re-run Jekyll on the source. Do not remove this file.

3. **Do not add custom theme overrides** (layouts, includes, SCSS) without first consulting the [Chirpy theme documentation](https://chirpy.cotes.page/). Most UI behavior is provided by the `jekyll-theme-chirpy` gem.

4. **Code style** is enforced by `.editorconfig`:
   - UTF-8 charset
   - 2-space indentation (all files)
   - LF line endings
   - Single quotes in JS/CSS; double quotes in YAML

5. **Navigation tabs** (`_tabs/archives.md`, `_tabs/categories.md`, `_tabs/tags.md`) are auto-generated by Jekyll/Chirpy. Only edit `_tabs/about.md` for personal content.

6. **Do not edit `_site/`**. It is the build output directory, git-ignored, and overwritten on every build.

7. **Avoid committing `Gemfile.lock`** — it is in `.gitignore`. Dependencies are resolved fresh in CI.

---

## Data Files

- `_data/contact.yml` — Sidebar social links (GitHub, Twitter, email, RSS). Add/remove entries here to modify the sidebar.
- `_data/share.yml` — Post-footer sharing buttons (Twitter, Facebook, Telegram). Add platforms by uncommenting or adding entries.

---

## Testing Checklist Before Pushing

- [ ] `bash tools/test.sh` completes without errors
- [ ] No broken internal links reported by `htmlproofer`
- [ ] Post front matter includes all required fields
- [ ] Post file name follows `YYYY-MM-DD-slug.md` convention
- [ ] Images placed in `assets/img/` and referenced with absolute paths (e.g., `/assets/img/my-image.jpg`)
