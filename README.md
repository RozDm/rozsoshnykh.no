# rozsoshnykh.no

Portfolio and blog of Dmytro Rozsoshnykh (sysadmin / DevOps, Norway), live at
https://rozsoshnykh.no. Norwegian (`nb-NO`) at the root, English (`en-GB`, see
`INTL_LOCALE` in `lib/shared/i18n.ts`) twin under `/en/`. Written almost
entirely in sessions with Claude Code, and set up so such a session can change
the site and ship it. Dmytro sets the direction and the rules and answers for
every change, so the commits are in Dmytro's name; the sessions write the code
and ship it through CI, smoke tests and automatic rollback. Code and docs are
English.

**Stack:** Next.js 16 static export (App Router, React 19, TypeScript,
Tailwind v4) on Cloudflare Workers static assets, plus a small Worker for
`/api/*` (D1, KV, Analytics Engine, Email Routing, Turnstile), all on the free
plan.

## Run, test, deploy

```
npm ci
npm run dev        # Next dev server (pages only; /api/* is not served)
npm run check      # format + lint + typecheck + tests; must pass before a PR
npm run build      # static export to out/
npm start          # serve out/ (scripts/serve-out.mjs; PORT, default 4173); needs a build
npm run test:e2e   # Playwright; needs a build with
                   # NEXT_PUBLIC_TURNSTILE_SITE_KEY=1x00000000000000000000AA
```

Deploy: PR → `check` (CI) → squash auto-merge → `deploy.yml` (build, secrets,
`wrangler deploy`, `scripts/smoke.sh`; a red smoke rolls back). Don't deploy by
hand from a sandbox; `npm run deploy` is for a machine with `wrangler login`.

## Docs

- [`ARCHITECTURE.md`](ARCHITECTURE.md): how the pieces fit, what each directory
  owns, where the rules live.
- `CLAUDE.md` (and one per area): the working rules for agents and people.
- `docs/`: `decisions.md` (the owner's standing decisions), `history.md`
  (incidents behind the rules), `gotchas.md` (root config),
  `cloudflare-security.md` (the zone).
- `docs/reference/` (`worker`, `frontend`, `ci`, `app`): the detail behind each
  area's `CLAUDE.md`.
- `node scripts/check-docs.mjs --map` prints the module map (path and purpose).

## Use it as a template

Fork it, then replace the personal parts: `lib/shared/site.ts`, `data/`,
`content/`, the privacy pages in `app/`, and `CONTACT_ADDRESS` in
`lib/shared/contact.ts`. The domain is also written out in
`public/.well-known/security.txt`, `public/_headers` (`Reporting-Endpoints`),
`.github/workflows/status-watchdog.yml`, `.github/workflows/zone-rules.yml`, and
`wrangler.jsonc` (ids, routes and custom domains). Secrets and the zone setup
are described in `.github/CLAUDE.md` and `docs/cloudflare-security.md`. Start
with `robots.index: false` until you want to be found.

## Licence

The code is MIT (`LICENSE`). The posts under `content/`, the images and the
personal texts (name, bio, privacy notices) are (c) Dmytro Rozsoshnykh, all
rights reserved.
