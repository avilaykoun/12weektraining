# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a personal marathon + strength hybrid training tracker ("AV Fitness" / "Albert 12 Week Program") built as a vanilla HTML/CSS/JavaScript Progressive Web App. There is no framework, bundler, or transpiler. The only external dependency is Dexie.js (an IndexedDB wrapper), loaded via CDN `<script>` tag directly in `index.html`. The app is fully client-side with no backend/API/server — all data lives in the browser (localStorage + IndexedDB).

Repo contents: `index.html` (the entire app — markup, CSS, and JS in one file, ~8,500 lines), `sw.js` (service worker), `manifest.json` (PWA manifest), icon PNGs, and `SETUP-GUIDE.md` (an end-user guide for hosting on GitHub Pages and installing as an iPhone home-screen app — not dev-facing).

## No build/test/lint tooling

There is no package.json, Makefile, CI config, test suite, or linter. Nothing to install or build. Changes are verified by opening `index.html` in a browser and exercising the feature manually — use the `run`/`verify` skills available in this environment for that. Do not invent build/test commands that don't exist.

Deployment is static hosting via GitHub Pages: commit the static files directly to `main`, no build step.

## Critical convention: bump the service worker version

`sw.js` defines:
```js
const VERSION = 'v83';
const CACHE = 'a12-' + VERSION;
```
`sw.js` implements network-first fetch for same-origin requests (so fresh deploys are picked up) with cache fallback for offline use. **Every change to `index.html`, `sw.js`, or `manifest.json` must bump `VERSION`**, otherwise clients can get stuck on stale cached assets. Commit/PR messages in this repo conventionally tag the version, e.g. `Fix session cards cutting off content on tall workout sessions (v83) (#68)`.

## Architecture

Everything lives in one large `<script>` block inside `index.html`. Since there are no files/modules to separate concerns, code is organized by `// ===== SECTION NAME =====` comment banners (e.g. `TIER 2/3 FEATURE LOGIC`, `RENDER TODAY`, `HUB PAGE`). Navigate with grep for these banners or for function/state names rather than scrolling — this is the most effective way to work in this file.

**Navigation / SPA structure**: no router library. The page is a set of `<div class="section" id="section-*">` blocks (`today`, `week`, `calendar`, `program`, `history`, `stats`, `pace`, `progress`, `nutrition`, `hub`) shown/hidden by `showSection(id, btn)`, which also updates the bottom nav bar (`.bottom-nav` / `.bnav-btn`) and calls that section's render function (`renderToday()`, `renderWeek()`, `renderCalendar()`, `renderHistory()`, `renderStats()`, `renderProgress()`, `renderNutritionMacros()`, `renderHub()`).

**Rendering pattern**: a global `STATE` object is mutated directly, then the relevant `render*()` function is called explicitly to re-render `innerHTML` for that section. There's no vdom/component framework. Dirty flags (`_historyDirty`, `_statsDirty`) gate expensive recomputation — set them when the underlying data changes, and rely on them rather than re-rendering unconditionally.

Storage/JSON operations are defensively wrapped in `try{...}catch(e){}` throughout — this is a personal, phone-based app where data loss is a real risk, so preserve that defensiveness when touching persistence code.

## Persistence layer

Dual-write pattern:
- **localStorage** key `mt_state` for synchronous instant boot, with an automatic backup key `mt_state_bak` (refreshed at most once per 24h) as a fallback if the primary is corrupted.
- **IndexedDB via Dexie** (`albert12_v1` database, single `state` table) as the async primary store for large payloads (GPX/TCX run imports) that exceed localStorage's ~5MB quota.
- `_idbMigrate` performs a one-time copy of localStorage state into IndexedDB on first load.
- `saveState(s)` / `loadState()` are the central read/write functions — use these rather than touching localStorage/IndexedDB directly.

State carries a `SCHEMA_VERSION` (currently 2) with an ordered `MIGRATIONS` array of idempotent, versioned migration functions run on load. **When changing the state shape, add a new migration function and bump `SCHEMA_VERSION` — never mutate the meaning of an existing field in place**, since real user data on real devices depends on migrations running in order.

## Domain data

The entire 12-week training program is hardcoded as a `PROGRAM` array literal, e.g.:
```js
{phase:1,week:1,day:'Wed',name:'Lower body A + core',type:'strength',exercises:[
  {id:'p1w1mon1',name:'Back squat',detail:'4 × 8 @ 65–70% 1RM · 3-sec descent',cat:'strength'}, ...
],note:'...'}
```
Exercise IDs follow a `p{phase}w{week}{day}{n}` convention (e.g. `p1w1mon1`). `RACE_DATE` and `PLAN_START` constants (`new Date(2026,7,30)` and `new Date(2026,5,8)`) anchor the plan's calendar. This `PROGRAM` array is the source of truth for the training schedule — edit it directly rather than deriving the plan elsewhere.

Storage keys and internal functions use an `mt_*` prefix (`mt_state`, `_idbSave`, `_idbLoad`).
