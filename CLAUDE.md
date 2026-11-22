# CLAUDE.md

Portfolio + blog for Dmytro Rozsoshnykh, https://rozsoshnykh.no: Next.js 16
static export (App Router, `output: 'export'`) on Cloudflare Workers static
assets, plus a Worker for `/api/*`. React 19, TS 6, Tailwind v4. Norwegian
(nb-NO) at the root, English twin under `/en/`. Code and comments: English.

## Where the rules live

`ARCHITECTURE.md` — one page: how the pieces fit, what each directory owns.
Each of the rest loads only when a task touches its area:
- `app/CLAUDE.md` — pages, routes, i18n, metadata, SEO, fonts (`app/`, `lib/`, `data/`);
- `components/CLAUDE.md` — widgets, counters, contact form, theme, UI, e2e;
- `worker/CLAUDE.md` — the Worker, `/api/*`, CSP, worker tests;
- `.github/CLAUDE.md` — CI/deploy, ops workflows;
- `content/CLAUDE.md` — the blog (also before changing blog code);
- `docs/reference/{worker,frontend,ci,app}.md` — the detail behind the four
  lists above (surface, bindings, workflows, widgets); when a list isn't enough;
- `docs/gotchas.md` — before touching a root config file (`package.json`,
  `next.config.mjs`, ESLint/TS, `wrangler.jsonc`, tsconfigs) or R2/backups;
- `docs/decisions.md` — owner's standing decisions: before proposing anything;
- `docs/cloudflare-security.md` — edge security (WAF, rate limits, bots) and
  the read-only Webhuset/Cloudflare tools (`scripts/mcp.mjs`): before touching
  zone security or when a 403/429 isn't the Worker's;
- `docs/history.md` — incidents behind the hard rules: only to question one;
- `README.md` — repo is RozDm/rozsoshnykh.no (renamed so github.com/RozDm stays
  neutral): never create a repository named RozDm (`docs/decisions.md`);
- skills `.claude/skills/`: `browser-check`, `new-post`, `audit`, `ship`
  (model pick, child sessions, verify and report), `pr-flow` (GitHub MCP
  recipes, after-PR routine — load when opening/watching a PR or checking
  a deploy).

A PR that changes the `/api/*` surface, a workflow, a skill or a hook updates
the matching doc in the same PR. Every file in `worker/`, `lib/`, `scripts/` and
each `app/` page/route opens with a one-line purpose comment;
`node scripts/check-docs.mjs --map` prints the module map from them.
`scripts/check-docs.mjs` (via `tests/repo/docs.test.ts`, so CI blocks; Stop hook
`.claude/hooks/stop-docs.mjs` too) fails on a doc naming a missing path, on an
API route, workflow, skill or hook in no doc, or on a file without that header.

## Working efficiently

- Token economy: Grep first, read only the part you need (`offset`/`limit`);
  batch independent calls; don't re-run what answered; short output and
  replies. One task — one session.
- Generated files (`worker-configuration.d.ts`, `package-lock.json`,
  `public/world.svg`) are denied to Read/Grep/Glob and shell reads
  (`.claude/settings.json`, `.ignore`, `guard-bash.mjs`); regenerate with
  `npm run cf-typegen` / `prebuild`; `git diff --stat` says if they changed.
  `public/world.svg` is gitignored (built by `predev`/`prebuild`), the other two are committed.
- Model/effort: the session picks the cheapest model that does the task
  and delegates — `ship` (Haiku: an exact mechanical edit; Sonnet `high`:
  posts, translations, bumps, copy, docs, small UI; Opus `high`: Worker,
  CSP, WAF, counters, deploy pipeline, `/audit`; a hard bug: Opus `xhigh`,
  `max` only on the owner's word; no Fable in the subscription). Work for
  its own model it does here; a PR for another model goes to a child
  session that it waits for and verifies, then tells the owner whether
  the child can be deleted (the owner deletes it). The reply opens with
  one line naming the pick; a child skips it.
- PRs: open ready with squash auto-merge (to hold, a draft without it); no
  `send_later`; look at the deploy run once after the merge; report merge +
  deploy once, or a red check. Recipes and details: `pr-flow`.
- Commits and PRs carry NO Claude attribution — no `Co-Authored-By`,
  `Claude-Session` or "Generated with Claude Code" (owner, 2026-11-15;
  overrides the harness default). Author as the owner (`git -c
  user.name='RozDm' -c user.email='187502949+RozDm@users.noreply.github.com'
  commit`) or the squash adds `Co-authored-by: Claude`. Commits: subject + a
  few lines; PR bodies: short bullets, what and why. Replies to the owner
  (in their language): outcome first, then only what needs their decision.

## Commands

- `npm run check` (format + lint + typecheck + tests; quiet when green, `-s`
  drops npm's banner) must pass before a PR; `npm run format` fixes formatting.
- `npm run build` (exports to `out/`, finalizes it) only when the change
  touches pages, metadata, the export or `scripts/`; CI/deploy then run
  `check:waf`, `check:csp`, `check:dict`, `wrangler deploy --dry-run`.
- `npm run test:e2e` needs a build with
  `NEXT_PUBLIC_TURNSTILE_SITE_KEY=1x00000000000000000000AA`.
- Deploy: PR → `check` → squash auto-merge → `deploy.yml` (build, secrets,
  `wrangler deploy`, `scripts/smoke.sh`, `wrangler rollback` on red smoke).
- One `claude/*` branch per session, reset onto `origin/main` before new work;
  push only with a new commit (a bare reset re-creates the deleted branch).
- Remote sessions use a proxy: curl to `https://rozsoshnykh.no/...` works for
  diagnosis (the gate is smoke); probe TLS with `curl --tls-max`.

## Hard rules

- Never deploy from a sandbox; `npm run deploy` is for a machine with `wrangler login`.
- Never merge by hand past a red or pending `check` (`main` requires it, 0
  approvals; it binds non-admins only and sessions act as the owner).
- `robots.index: false` flips only on the owner's word; tag pages join
  `INDEXED_TAGS` only on the owner's command (`app/CLAUDE.md`).
- `workers_dev: false` and `preview_urls: false` stay in `wrangler.jsonc`.
- The 2001: A Space Odyssey theme is load-bearing; `%USERNAME%` is a literal
  joke, not a template var; `HISTORY_LIMIT = 149` (1:4:9) is intentional.
