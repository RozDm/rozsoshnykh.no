# Pages, languages, metadata (app/, lib/, data/)

Traps only; read before `lib/` or `data/` too. Routes, i18n, SEO and font
detail: `docs/reference/app.md`; blog: `content/CLAUDE.md`; root configs:
`docs/gotchas.md`.

- `lib/` is split by where code runs: `lib/server/` (build/export time:
  `node:fs`, `content/`, OG cards, fonts, Markdown, feed), `lib/client/`
  (browser APIs, hooks, `/api/*` fetchers) and `lib/shared/` (pure, no
  environment; the Worker imports it too). A new file goes where it runs;
  who may import whom is enforced by ESLint (`eslint.config.mjs`). No
  `server-only` marker: the package isn't a dependency and tests/scripts
  import these files outside Next.
- ALL UI copy lives in `data/dictionary.ts` (`nb` defines the shape, `en` is typed
  against it, `tests/lib/shared/i18n.test.ts` checks parity), never in a component.
  Exceptions: `app/error.tsx` reads `ERROR_COPY` (`data/dictionary-error.ts`;
  importing `DICT` doubled the dictionary on every page) and the privacy
  notices' text stays in their pages. The Norwegian notice is authoritative:
  change both.
- Page links go through `localePath()`; Norwegian/English pages share the page
  components; bilingual data is `Localized` + `pick()`.
- The export always says `<html lang="nb">`; `scripts/finalize-export.mjs`
  rewrites `/en/` pages and the theme head script covers `/en/` 404s.
- Every page uses `pageMetadata()`. Next merges metadata SHALLOWLY: a page
  setting part of `openGraph`/`alternates` drops the rest. The root layout has
  no canonical/og:url (the 404 inherits it); canonicals keep the trailing slash.
- Name, roles, titles and `THEME_BG` live in `lib/shared/site.ts`, the contact
  address in `lib/shared/contact.ts`: never hard-code them. Positioning: Bergen
  in the title, all of Norway in the description; the JSON-LD address stays
  Askøy, Vestland (owner, 2026-11-22).
- `app/template.tsx` cross-fades route changes with OPACITY ONLY (a transform
  traps the fixed intro overlays and sticky header); both front pages opt out.
- `/.well-known/security.txt`: bump `Expires` yearly (a test fails 30 days
  ahead).
- `robots.index: false` in `app/layout.tsx` flips only on the owner's word
  (not with the first post), together with the e2e test "pre-launch: every
  page tells search engines noindex" and the home check in `scripts/smoke.sh`
  (all three, or CI or the deploy fails; steps: `/new-post`).
- Tag pages are noindex and out of the sitemap unless in `INDEXED_TAGS`
  (`data/tags.ts`): add one ONLY on the owner's command. `/kontakt`,
  `/personvern` and utility pages stay noindex for good; the sitemap never
  lists noindex URLs (pre-launch excepted).
- Body font: `next/font/local` from `@fontsource-variable/intel-one-mono`
  (no network at build). Never rename a family const through `declarations`
  (Turbopack names the family after it). Keep `display: 'optional'`,
  `preload: true`, `adjustFontFallback: false`. `next/og` can't read WOFF2.
- Tailwind v4 has no config file: `@theme` lives in `app/globals.css`.
