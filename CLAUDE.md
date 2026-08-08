# duan — Guidelines

## Status: unpublished

`duan.life` no longer serves the app — it serves the blank page in [offline/](offline/),
deployed by [.github/workflows/blank.yml](.github/workflows/blank.yml). The app's
pipeline is preserved but inert in `.github/workflows/deploy.yml.disabled`. Everything
below still describes the app, which is untouched; just don't assume a live site.
See the "Déploiement" section of [README.md](README.md) to republish.

## Stack

- React 19 + Vite 8 + TypeScript 5 (strict) + Tailwind 3.4.
- Routing: `react-router-dom` **BrowserRouter** (clean URLs, no `#`), `basename`
  = `import.meta.env.BASE_URL`. GitHub Pages SPA fallback via
  [public/404.html](public/404.html) + a decode shim in [index.html](index.html).
- Markdown: `react-markdown` (no raw HTML).
- Crypto: `@noble/hashes` (Argon2id) + WebCrypto (AES-GCM) for the encrypted PAT.
- i18n: tiny in-house provider ([src/i18n/](src/i18n/)). **Always-on bilingual** —
  every label renders as "中文 / English" at once; there is no language switch.
  Strings in `translations.ts` (`zhTW` is typed against `en`'s keys, so a missing
  key fails the build).
- No IndexedDB, no service worker, no backend. Single editor (the professor).

## Sections

- **Home** (`/`) — static showcase placeholder.
- **Blog** (`/blog`, `/blog/:slug`; editor at `/editor`, `/editor/:slug`) — public
  reading; the professor writes / edits / deletes once signed in.

## Data model & persistence

- One data repo: the PUBLIC blog repo `duan-blog`. Layout: `articles/<slug>.json`
  (full `Article`) + `articles/index.json` (`ArticleMeta[]` manifest).
  ([src/blog/articlesRepo.ts](src/blog/articlesRepo.ts))
- Anonymous visitors read via `raw.githubusercontent`
  ([src/github/publicRead.ts](src/github/publicRead.ts)); the professor reads +
  writes via the authenticated Contents API ([src/github/client.ts](src/github/client.ts)).
  `saveArticle` writes the post file then refreshes the index; `rebuildIndex()`
  regenerates the manifest from the files if they drift.
- Repo coordinates are non-secret → [src/config.ts](src/config.ts). Only the PAT
  is encrypted, in `public/secrets.json` (built by
  [scripts/build-secrets.mjs](scripts/build-secrets.mjs), gitignored).

## Auth

- App is **public by default**. `AuthProvider` holds `{status:"locked"}` until the
  professor signs in via the non-blocking `LoginScreen` (one password → decrypts
  the PAT → `{status:"unlocked", pat}`). Nothing persisted; reload returns to
  locked. `/editor*` routes redirect to `/` when locked.

## Conventions

- Mobile-first responsive. NavBar collapses behind a hamburger < md; dialogs are
  bottom-sheet on mobile.
- Neutral wireframe palette: CSS vars `surface`/`content`/`accent`, dark mode via
  `<html class="dark">` (toggle in NavBar, synchronous boot script in index.html).
- UI strings come from i18n (`t("…")`); never hardcode user-facing text. Default
  locale zh-TW — write idiomatic Traditional Chinese + English, not literal
  translations. Code/comments in English.
- Conflicts (409/422) are rare (single editor): the index re-reads the SHA and
  retries once (see `writeIndex`).

## Commands

```sh
npm run dev | npm run build | npm run typecheck
SYNC_PAT=… SYNC_PASSWORD=… node scripts/build-secrets.mjs   # local secrets bundle
```
