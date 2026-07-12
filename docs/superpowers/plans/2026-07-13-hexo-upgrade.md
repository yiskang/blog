# Hexo Upgrade to Latest Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bring this Hexo 3.2.2 blog and all its plugins to latest, replace the vendored `landscape` theme with the official `hexo-theme-landscape` npm package, and verify the site still generates and serves correctly — without deploying.

**Architecture:** All work happens on branch `upgrade/hexo-8`. `package.json` gets every `hexo-*` dependency bumped to its current major, plus a new `hexo-theme-landscape` dependency. The vendored `themes/landscape/` directory is deleted; theme customization moves to a project-root `_config.landscape.yml` override file (Hexo's built-in mechanism for configuring an npm-installed theme without touching its files inside `node_modules`). Verification is manual/scripted (no test suite exists): a structural diff of `hexo generate` output against a pre-upgrade baseline, plus `curl` checks against a running `hexo server` to confirm pages, images, feed, and sitemap all resolve correctly under the site's `/blog` root subpath.

**Tech Stack:** Hexo 8.1.2, Node.js (>=20.19, v22.23.1 available locally), npm, EJS/Stylus/Marked renderers, `hexo-theme-landscape` v1.1.0.

## Global Constraints

- Node.js engine floor: `>=20.19.0` (required by `hexo@8.1.2`). Local Node v22.23.1 satisfies this.
- No new npm scripts, no test framework, no linter — none exist today and none are being added.
- No deploy (`hexo deploy`) as part of this plan — verification stops at local `hexo generate`/`hexo server` checks.
- Theme feature adoption is out of scope: new `hexo-theme-landscape` options (`banner`, `links`, `recent_posts_limits`, `copyright`, `valine`) are left at their shipped defaults, not configured.
- `root: /blog` and the deploy target in `_config.yml` are unchanged.

---

### Task 1: Create feature branch and capture a pre-upgrade generation baseline

**Files:** none created or modified — this task only creates a git branch and local build artifacts (`public-baseline/`) that later tasks compare against and task 5 deletes.

**Interfaces:**
- Produces: a directory `public-baseline/` at the repo root containing the current (pre-upgrade) `hexo generate` output, used by Task 5's diff step.

- [ ] **Step 1: Create and switch to the feature branch**

```bash
git checkout -b upgrade/hexo-8
```

Expected: `Switched to a new branch 'upgrade/hexo-8'`

- [ ] **Step 2: Install current (pre-upgrade) dependencies**

```bash
npm install
```

Expected: exits 0, `node_modules/` created with `hexo@3.2.x` and the current plugin set.

- [ ] **Step 3: Generate the baseline site**

```bash
npx hexo generate
```

Expected: exits 0, prints a line ending in "generated in ...ms", and creates a `public/` directory containing `index.html`.

- [ ] **Step 4: Preserve the baseline output under a different name**

```bash
mv public public-baseline
```

Expected: `public-baseline/` now exists, `public/` no longer does.

- [ ] **Step 5: Clean Hexo's cache**

```bash
npx hexo clean
```

Expected: exits 0. This clears Hexo's internal `db.json` cache so the upgraded Hexo version in later tasks doesn't try to read a cache file written by the old version.

No commit for this task — it produced no changes to tracked files, only the branch and a local, uncommitted `public-baseline/` directory.

---

### Task 2: Bump hexo core and all non-theme plugins to latest

**Files:**
- Modify: `package.json`

**Interfaces:**
- Consumes: nothing from Task 1 except being on the `upgrade/hexo-8` branch.
- Produces: `node_modules` containing `hexo@8.1.2` and the bumped plugins, which Task 3 and Task 5 both depend on.

- [ ] **Step 1: Edit `package.json` dependencies**

Change the `dependencies` block from:

```json
  "dependencies": {
    "hexo": "^3.2.0",
    "hexo-deployer-git": "^0.2.0",
    "hexo-generator-archive": "^0.1.4",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-feed": "^1.2.0",
    "hexo-generator-index": "^0.2.0",
    "hexo-generator-sitemap": "^1.1.2",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.2.0",
    "hexo-renderer-marked": "^0.2.10",
    "hexo-renderer-stylus": "^0.3.1",
    "hexo-server": "^0.2.0"
  }
```

to:

```json
  "dependencies": {
    "hexo": "^8.1.2",
    "hexo-deployer-git": "^4.0.0",
    "hexo-generator-archive": "^2.0.0",
    "hexo-generator-category": "^2.0.0",
    "hexo-generator-feed": "^4.0.0",
    "hexo-generator-index": "^4.0.0",
    "hexo-generator-sitemap": "^3.0.1",
    "hexo-generator-tag": "^2.0.0",
    "hexo-renderer-ejs": "^2.0.0",
    "hexo-renderer-marked": "^7.0.1",
    "hexo-renderer-stylus": "^3.0.1",
    "hexo-server": "^3.0.0"
  }
```

Also update the informational `hexo.version` field at the top of the same file, from:

```json
  "hexo": {
    "version": "3.2.2"
  },
```

to:

```json
  "hexo": {
    "version": "8.1.2"
  },
```

- [ ] **Step 2: Clean-install the new dependency versions**

```bash
rm -rf node_modules
npm install
```

Expected: exits 0. Deprecation warnings from transitive dependencies are fine; a non-zero exit code or `ERESOLVE` peer-dependency error is not — if that happens, stop and inspect before continuing.

- [ ] **Step 3: Verify the new Hexo core version is active**

```bash
npx hexo version
```

Expected: the `hexo:` line reads `8.1.2` (not `3.2.2`).

- [ ] **Step 4: Commit**

```bash
git add package.json
git commit -m "Bump hexo core and plugins to latest majors"
```

---

### Task 3: Replace the vendored theme with the hexo-theme-landscape package

**Files:**
- Delete: `themes/landscape/` (entire directory)
- Modify: `package.json`

**Interfaces:**
- Consumes: `hexo@8.1.2` installed by Task 2 (theme resolution behavior being relied on here is Hexo 8's, not Hexo 3's).
- Produces: `node_modules/hexo-theme-landscape`, which Hexo resolves via the existing `theme: landscape` setting in `_config.yml` (unchanged) now that `themes/landscape/` no longer shadows it.

- [ ] **Step 1: Remove the vendored theme directory**

```bash
git rm -r themes/landscape
```

Expected: reports the theme's files as deleted (staged for removal).

- [ ] **Step 2: Add the theme package as a dependency**

Add this line to the `dependencies` block in `package.json` (alphabetical position, after `hexo-server`):

```json
    "hexo-server": "^3.0.0",
    "hexo-theme-landscape": "^1.1.0"
```

- [ ] **Step 3: Install the new dependency**

```bash
npm install
```

Expected: exits 0, `node_modules/hexo-theme-landscape/` now exists.

- [ ] **Step 4: Verify the package is resolvable**

```bash
npm ls hexo-theme-landscape
```

Expected: prints `hexo-theme-landscape@1.1.0` with no `(empty)` or `UNMET DEPENDENCY` marker.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "Replace vendored landscape theme with hexo-theme-landscape package"
```

---

### Task 4: Migrate theme configuration to a project-root override file

**Files:**
- Create: `_config.landscape.yml` (repo root)

**Interfaces:**
- Consumes: Hexo's theme-config-merge behavior — Hexo looks for a file named `_config.<theme>.yml` at the project root and merges it over the theme's own internal `_config.yml`, with a log line `Second Theme Config loaded: <path>` at debug level when found (source: `hexo`'s `lib/hexo/load_theme_config.js`).
- Produces: the site's actual rendered theme configuration going forward — the settings this file defines are what appears on the generated pages (menu, RSS link, sidebar widgets, etc.).

- [ ] **Step 1: Create `_config.landscape.yml`**

This carries forward every setting from the old `themes/landscape/_config.yml`, using only keys that still exist in `hexo-theme-landscape@1.1.0`. `index_widgets` and `google_plus` are dropped (both were blank/unused in the old config and have no equivalent in the new theme). New optional keys (`banner`, `subtitle`, `links`, `recent_posts_limits`, `copyright`, `valine`) are intentionally omitted so they fall back to the theme's shipped defaults.

```yaml
# Header
menu:
  Home: /
  Archives: /archives
rss: /atom.xml

# Content
excerpt_link: Read More
fancybox: true

# Sidebar
sidebar: right
widgets:
- category
- tag
- tagcloud
- archive
- recent_posts

# Widget behavior
archive_type: 'monthly'
show_count: false

# Miscellaneous
google_analytics:
favicon: /favicon.png
```

- [ ] **Step 2: Verify Hexo loads the override file**

```bash
npx hexo generate --debug 2>&1 | grep -i "Second Theme Config loaded"
```

Expected: one line of output containing `_config.landscape.yml`, confirming Hexo found and merged the override file. If this line is absent, the file is misnamed or misplaced — it must be named exactly `_config.landscape.yml` and live at the repo root (same level as `_config.yml`), not inside `themes/`.

- [ ] **Step 3: Commit**

```bash
git add _config.landscape.yml
git commit -m "Migrate theme config to root-level _config.landscape.yml override"
```

---

### Task 5: Full verification — structural diff and live server checks

**Files:** none persisted as a deliverable. This task only produces local build artifacts (`public/`) that get deleted at the end, plus `public-baseline/` from Task 1 (also deleted at the end).

**Interfaces:**
- Consumes: `public-baseline/` from Task 1; the fully upgraded, generate-able site from Tasks 2–4.
- Produces: nothing persisted — pass/fail confidence that the upgrade didn't regress site output.

- [ ] **Step 1: Regenerate the upgraded site**

```bash
npx hexo clean && npx hexo generate
```

Expected: exits 0, `public/` recreated.

- [ ] **Step 2: Structural diff against the baseline**

```bash
diff -rq public-baseline public
```

Expected: differences are limited to expected churn — regenerated HTML markup (new theme version renders different markup than the old vendored copy), and timestamps inside `atom.xml`/`sitemap.xml`. There should be **no** entries indicating a file present in `public-baseline` is now **missing** from `public` — check specifically for that pattern:

```bash
diff -rq public-baseline public | grep "^Only in public-baseline"
```

Expected: no output. Any output here means something the old build produced (a post page, an image, a tag/category page) no longer exists in the new build — investigate before proceeding.

- [ ] **Step 3: Confirm post and image counts are preserved**

```bash
find public-baseline -name "*.html" | wc -l
find public -name "*.html" | wc -l
find public-baseline \( -name "*.jpg" -o -name "*.png" \) | wc -l
find public \( -name "*.jpg" -o -name "*.png" \) | wc -l
```

Expected: the `public` counts are greater than or equal to the `public-baseline` counts in both cases (equal is expected; greater would mean the new theme adds pages, which is not inherently wrong but should be noted).

- [ ] **Step 4: Start the server in the background**

```bash
npx hexo server &
sleep 2
```

Expected: log line `Hexo is running at http://localhost:4000/blog/ . Press Ctrl+C to stop.`

- [ ] **Step 5: Check the homepage and archives resolve under the `/blog` root**

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:4000/blog/
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:4000/blog/archives/
```

Expected: both print `200`.

- [ ] **Step 6: Check a post with embedded images renders and its images actually resolve**

This is the known fragile spot noted in `CLAUDE.md`: image references in post Markdown are relative (`images/foo.jpg`), which only resolve correctly if the theme and generator handle the `/blog` root subpath correctly.

```bash
curl -s -o /dev/null -w "%{http_code}\n" "http://localhost:4000/blog/2015/05/06/Revit-debugging-on-Visual-Studio-2013/"
curl -s "http://localhost:4000/blog/2015/05/06/Revit-debugging-on-Visual-Studio-2013/" | grep -o 'src="[^"]*RevitDebuggingError\.jpg"'
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:4000/blog/images/RevitDebuggingError.jpg
```

Expected: first command prints `200`; second command prints a `src="..."` line containing `RevitDebuggingError.jpg` (confirming the image tag rendered); third command prints `200` (confirming the image itself, not just the `<img>` tag, resolves at that URL).

- [ ] **Step 7: Check the feed and sitemap resolve**

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:4000/blog/atom.xml
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:4000/blog/sitemap.xml
```

Expected: both print `200`.

- [ ] **Step 8: Stop the server**

```bash
kill %1
```

Expected: background `hexo server` process terminated.

- [ ] **Step 9: Clean up local build artifacts**

```bash
rm -rf public public-baseline
```

- [ ] **Step 10: Commit, only if any fix was needed**

If every check above passed with no code changes, there is nothing to commit — skip this step. If a check failed and required a fix (e.g., a config key needed adjusting), stage and commit that fix now:

```bash
git add -A
git commit -m "Fix <specific issue found during upgrade verification>"
```

---

### Task 6: Update CLAUDE.md to reflect the new theme architecture

**Files:**
- Modify: `CLAUDE.md`

**Interfaces:**
- Consumes: nothing code-level; this is a documentation-accuracy follow-up to Tasks 3–4 changing where the theme lives and how it's configured.
- Produces: nothing consumed by later tasks — this is the last task in the plan.

- [ ] **Step 1: Update the Architecture section**

In `CLAUDE.md`, replace the current `themes/landscape/` bullet (which describes it as vendored in-repo with a hand-patched `url_for()` fix) with a description matching the new reality:

```markdown
- `themes/` — no vendored theme; the active theme (`theme: landscape` in `_config.yml`) is the `hexo-theme-landscape` npm package. Its settings are overridden via the project-root `_config.landscape.yml` file (Hexo's built-in mechanism for configuring an installed theme without editing files inside `node_modules`) rather than by editing the theme package directly.
```

Remove the now-inaccurate sentence about the hand-patched `header.ejs`/`url_for()` fix — that fix is present upstream in the current theme package, so there is no local patch to describe anymore.

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "Update CLAUDE.md to describe theme as an npm dependency"
```

---

## Self-Review Notes

- **Spec coverage:** dependency bumps (Task 2), theme swap (Task 3), theme config migration (Task 4), verification without deploy (Task 5), feature branch workflow (Task 1). The spec's `hexo-generator-feed` config concern was resolved during planning — its current README shows `type`, `path`, `limit`, `hub`, `content` are all still valid keys, so no `_config.yml` change is needed there; Task 5's generate/diff/curl checks cover confirming the feed still works.
- **Out-of-scope items respected:** no new theme features configured (Task 4 deliberately omits `banner`/`links`/`recent_posts_limits`/`copyright`/`valine`), no deploy step, no CI/tests added.
- **Type/name consistency:** `public-baseline/` (Task 1) is the exact name diffed against in Task 5; `_config.landscape.yml` (Task 4) is the exact filename whose load is verified in Task 4 Step 2 and whose settings are exercised in Task 5.
