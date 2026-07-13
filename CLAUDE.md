# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A [Hexo](https://hexo.io) 8.x static blog ("13筆記" / 13 Notes), source-controlled on `master` and built/deployed to the `gh-pages` branch of the same repo (`git@github.com:yiskang/blog.git`). The site is served from GitHub Pages under a project subpath (`https://yiskang.github.io/blog`), not the domain root.

## Commands

There are no npm scripts defined in `package.json` — use the `hexo` CLI directly (via `npx` since `hexo-cli` isn't installed globally by default):

```bash
npm install                # install dependencies

npx hexo server            # serve locally with live reload at http://localhost:4000
npx hexo generate          # build static site into public/
npx hexo clean             # clear the generated public/ and cache
npx hexo deploy            # generate + push public/ to the gh-pages branch (requires SSH access to the repo)
```

There is no test suite or linter configured for this repo.

## Architecture

- `_config.yml` — Hexo site config. Notably `root: /blog` because the site is deployed under a GitHub Pages project subpath, not the domain root. **`url` and `root` must stay in sync with the subpath the site is actually served from**, or generated links/assets break (see the "Fix path issue for deploying on project gh-pages" commit).
- `source/_posts/` — blog post Markdown files, one per post. Filenames become part of the post's identity; Chinese-language filenames are common here.
- `source/images/` — post images. **Image references in post Markdown must use relative paths (`images/foo.png`), not root-relative (`/images/foo.png`)** — since `root` is `/blog`, a root-relative path resolves to the wrong location once deployed. `hexo-renderer-marked` only prepends `root` to an image's `src` when it parses `![]()` syntax as an actual markdown image token; if an image line sits directly inside a raw HTML block (e.g. immediately after a `<div>` on its own line with no blank line before/after), CommonMark's HTML-block rules make the parser skip it entirely, and the markdown text is emitted literally instead of an `<img>` tag. Always put a blank line before and after `![]()` syntax that's wrapped in `<div>`/`<span>` HTML for layout.
- `themes/` — no vendored theme; the active theme (`theme: landscape` in `_config.yml`) is the `hexo-theme-landscape` npm package. Its settings are overridden via the project-root `_config.landscape.yml` file (Hexo's built-in mechanism for configuring an installed theme without editing files inside `node_modules`) rather than by editing the theme package directly.
- `scaffolds/` — templates (`draft.md`, `page.md`, `post.md`) used by `hexo new` when creating new content.
- Deployment is handled entirely by `hexo-deployer-git` per the `deploy:` block in `_config.yml` — it pushes the generated `public/` directory to the `gh-pages` branch of the deploy `repo`.

## Preferred tool usage

- Prefer Read/Edit/Write tools over Bash `cat`, `echo >`, or heredocs whenever possible.
- Prefer `npm run ...` over invoking binaries in `node_modules/.bin`.
- Prefer existing worktree over creating a new one if already inside a Git worktree.
