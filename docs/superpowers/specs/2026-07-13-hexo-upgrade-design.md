# Upgrade to latest Hexo

Date: 2026-07-13

## Context

This repo is a Hexo 3.2.2 static blog. `hexo` and every `hexo-*` plugin are five-plus major versions behind latest. The theme (`landscape`) is vendored (copied into `themes/landscape/`) rather than installed as a package, with exactly one hand-made patch: `url_for(theme.rss)` in `header.ejs`, fixing an RSS link that broke once the site started deploying under the GitHub Pages subpath `/blog` (see commit `c2c424b`). That same fix already exists upstream in `hexo-theme-landscape` v1.1.0, so nothing is lost by dropping the vendored copy.

Local environment: Node v22.23.1, which satisfies Hexo 8's `>=20.19.0` engine requirement. There is no CI, no test suite, and no npm scripts defined — all commands go through the `hexo` CLI directly.

## Goal

Bring `hexo` and all plugins to their latest majors, replace the vendored theme with the official `hexo-theme-landscape` npm package, and verify the site still generates and renders correctly — without deploying.

## Decisions

- **Theme**: fully replace vendored `themes/landscape/` with the `hexo-theme-landscape` npm package (not kept vendored).
- **Deploy scope**: stop after local verification; `hexo deploy` is left as a manual step for the repo owner.
- **New theme features** (banner image, icon-based social links, Valine comments): left at shipped defaults, not enabled. Minimal config migration only.
- **Approach**: single combined pass (bump everything + swap theme together), not staged phase-by-phase, given the small blast radius (no CI, one theme, ~7 posts, no custom plugins).
- **Git workflow**: done on a feature branch (`upgrade/hexo-8`), not directly on `master`.

## Design

### 1. Dependency bumps

`package.json` dependencies updated in place (caret ranges, matching existing convention):

| Package | Current | New |
|---|---|---|
| hexo | ^3.2.0 | ^8.1.2 |
| hexo-deployer-git | ^0.2.0 | ^4.0.0 |
| hexo-generator-archive | ^0.1.4 | ^2.0.0 |
| hexo-generator-category | ^0.1.3 | ^2.0.0 |
| hexo-generator-feed | ^1.2.0 | ^4.0.0 |
| hexo-generator-index | ^0.2.0 | ^4.0.0 |
| hexo-generator-sitemap | ^1.1.2 | ^3.0.1 |
| hexo-generator-tag | ^0.2.0 | ^2.0.0 |
| hexo-renderer-ejs | ^0.2.0 | ^2.0.0 |
| hexo-renderer-marked | ^0.2.10 | ^7.0.1 |
| hexo-renderer-stylus | ^0.3.1 | ^3.0.1 |
| hexo-server | ^0.2.0 | ^3.0.0 |
| hexo-theme-landscape | (none, vendored) | ^1.1.0 (new) |

`hexo-generator-feed`'s config schema has changed since v1.x. The `feed:` block in `_config.yml` (`type: atom`, `path: atom.xml`, `limit: 20`, blank `hub`/`content`) will be checked against v4's current option names and adjusted only as needed to preserve today's behavior — no new feed features added.

### 2. Theme migration

- Delete `themes/landscape/` entirely.
- Add `hexo-theme-landscape` as a `package.json` dependency; `theme: landscape` in `_config.yml` stays as-is (Hexo resolves it from `node_modules`).
- Port the existing theme config block in `_config.yml` key-for-key onto the new theme's schema:
  - Kept: `menu`, `rss`, `excerpt_link`, `fancybox`, `sidebar`, `widgets`, `archive_type`, `show_count`, `favicon`, `google_analytics`.
  - Dropped (no upstream equivalent, both were blank/unused in current config): `index_widgets`, `google_plus`.
  - Left at shipped defaults, not configured (per the "minimal migration" decision): `banner`, `links`, `recent_posts_limits`, `copyright`, `valine`.

### 3. Verification plan

No automated tests exist, so verification is manual:

1. On current `master`, run `hexo generate` and set the resulting `public/` aside as a baseline.
2. On the feature branch, apply all dependency and theme changes, run `npm install`, then `hexo generate` again.
3. Diff the two `public/` outputs, looking for structural regressions: broken internal links, missing/misplaced image assets, malformed feed or sitemap XML.
4. Run `hexo server` and manually check in a browser:
   - Homepage
   - One post containing embedded images and Chinese-language content (image path handling under the `/blog` root subpath is the known fragile spot — see `CLAUDE.md`)
   - An archive or tag listing page
   - `/atom.xml`
   - `/sitemap.xml`
5. No deploy step. `hexo deploy` remains a manual action for the repo owner once they're satisfied with the verification.

### Out of scope

- Adopting new theme features (banner image, social link icons, Valine comments).
- Adding CI, a test suite, or linting.
- Any change to the deploy target or subpath configuration (`root: /blog` stays as-is).
