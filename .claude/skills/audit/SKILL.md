---
name: audit
description: Whole-project audit producing a prioritized improvement list (P0/P1/P2/P3) with risk and time estimates, scoped to this site's free-tier Cloudflare constraints — code, security, performance, accessibility, content, SEO against the goal of being found in Norway, and launch readiness. Use when the user asks to analyze/audit the project, check SEO or launch readiness, find what can be improved, what's outdated, or what to do next.
effort: high
---

# Project audit

Produce a focused, opinionated improvement list for `rozsoshnykh.no`. The output is a punch list the user can pick from, not an essay.

## Scope

Audit everything that ships:
- App: `app/`, `components/`, `lib/`, `data/`, `content/`, `public/`, `app/globals.css`.
- Worker: `worker/` — it serves only `/api/*`; pages are asset-served with a build-time CSP (`scripts/finalize-export.mjs`). Layout: `index.ts` dispatch, `routes/*` HTTP only, `domain/*` rules and stores (`*-store.ts`), `http.ts`, `csp.ts`, `ip.ts`, `validate.ts`, `api-types.ts`; `node scripts/check-docs.mjs --map` prints the current module map. Unknown `/api/*` paths answer JSON 404 `not_found` and wrong methods 405 + `Allow` (PR #42) — by design, don't re-flag.
- Build/deploy: `package.json`, `next.config.mjs`, `wrangler.jsonc`, `.github/workflows/*`, `scripts/*`, `tsconfig*.json`.
- Docs: `ARCHITECTURE.md`, `CLAUDE.md`, `app/CLAUDE.md`, `components/CLAUDE.md`, `worker/CLAUDE.md`, `.github/CLAUDE.md`, `content/CLAUDE.md`, `docs/reference/{worker,frontend,ci,app}.md`, `README.md`, `docs/decisions.md`, `docs/gotchas.md`, `docs/history.md`, `docs/cloudflare-security.md` (drift against code or the live zone is a finding; narrative belongs in history.md, rules in the CLAUDE.md files).
- Tests: `tests/` (Vitest, the Worker's in `tests/worker/`) and `e2e/` (Playwright).

Look for:
- **Security**: CSP, headers, secrets handling, rate limits, validation, Turnstile gating, DMARC/DNSSEC posture.
- **Performance**: initial JS, image weight, RSC/Server Component opportunities, dynamic-import candidates, font/CSS payload. Only from a measurement (see Evidence).
- **Correctness**: dead code, stale comments, drift between docs and code, broken types, untested paths.
- **Modernization**: Next 16, React 19, TypeScript 6 features actually used; obsolete patterns; dependencies a major behind.
- **SEO**: the technical checks (canonicals, sitemap/robots, JSON-LD, OG images, hreflang) plus the goal fit below.
- **Accessibility** (universell utforming is a signal Norwegian employers notice): axe-core through the `browser-check` skill; the keyboard path (intro skippable by key, visible focus); contrast in both themes; `lang` on `/en/` pages; alt text.
- **Content and conversion**: thin prose on the home page (one hero paragraph, the rest widgets); no clear CTA or availability line; post cadence and topic clusters; internal links from posts to home and contact. Facts about the owner's experience come only from the owner — never write or invent them. An experience section and a case-study post about the site are the owner's to bring later (2026-11-22): don't raise them.
- **Operational**: free-tier limits (KV 1k writes/day, D1 100k writes/day), monitoring blind spots, missing alerts.

## SEO with a goal

The site's job is to be found and to convince an employer or client in Norway. The owner's head terms, "IT-spesialist Norge", "DevOps Norge", "systemadministrator Norge", return job boards, salary sites, courses and meetups (finn.no, NAV, Indeed, Jooble, Payscale): a personal site won't rank there and the intent doesn't match. Realistic targets:
- the name query, "Dmytro Rozsoshnykh";
- Bergen-area local terms: "DevOps Bergen", "systemadministrator Bergen", "IT-drift Askøy";
- long-tail Norwegian technical posts.

Check title, description, H1, body and internal links against these; check the search intent of each target (WebSearch is a US API, so its results are indicative, not google.no); title ≤ 60 characters. The site targets employers and clients alike, positioned on Bergen with reach across all of Norway (owner, 2026-11-22, `docs/decisions.md`): the local terms come first, and national reach comes from long-tail posts and the description.

## Launch gate

Only while `robots.index` is false. A checklist section in the report; owner steps marked **(owner)**:
- Flip `robots.index` on the owner's word only, in its three places: `app/layout.tsx`, the e2e test "pre-launch: every page tells search engines noindex", the home check in `scripts/smoke.sh` (**owner**).
- Google Search Console and Bing Webmaster Tools, verified by a DNS TXT record (**owner**: sessions' Cloudflare token is read-only); submit the sitemap.
- Rich Results Test for Person, BlogPosting and BreadcrumbList; hreflang pairs; the OG preview in LinkedIn Post Inspector.
- After indexing: the name query returns the site first.

## Hard constraints (do NOT recommend violating)

Read `docs/decisions.md` first and the root `CLAUDE.md` Hard rules: the owner's standing decisions (workers.dev off, Trends dots, forgeable counters, language switch, quiet uptime caption, no external monitor, no Required reviewers, web-storage dedupe, `robots.index`, `INDEXED_TAGS`, the noindex utility pages, the 2001 theme, `HISTORY_LIMIT`, `%USERNAME%`…) — never re-propose one. The technical constraints found nowhere else:

- **Free Cloudflare tier only.** R2 requires a payment card on file — never recommend it. KV writes are scarce — never add a KV write per request.
- **DNSSEC stays on.** Never recommend disabling it without first removing DS at the registrar (would cause global SERVFAIL).
- **Sandbox egress varies.** Newer remote sessions can `curl` production through the agent proxy; older web sandboxes have no egress. Never make a recommendation *depend* on reaching production — the deploy gate is `scripts/smoke.sh` in CI.
- **`package.json` is not `type: module`.** Don't propose making it one — it breaks Next 16 / turbopack config load.
- **ESLint stays on 9.** `eslint-plugin-react` via `eslint-config-next` is not ESLint-10-compatible. Don't propose a bump.
- **`/status` is a 301 to `/#status`** (`public/_redirects`), not a page.
- **`StatusDashboard`, `GeoMap` and `TrendsChart` are code-split** (`Lazy*.tsx`), so the smoke-checked headings must stay in the server components `Status.tsx`, `Visitors.tsx` and `Trends.tsx`.

## Evidence

Every finding cites `file:line` or a measured number, in the Item or Why cell (no extra column). Performance findings come only from a measurement: build output sizes, or a fresh Playwright context on production (first visit, so the intro plays). Chrome's LCP ignores the intro overlay, so the intro doesn't show up in LCP. The intro plays for every first-time visitor, search referrals included (owner, 2026-11-22): don't propose skipping it.

Check the live output, not only the code, when production is reachable: `curl https://rozsoshnykh.no/robots.txt`, sitemap.xml, response headers and meta robots. The live robots.txt is `app/robots.ts` plus Cloudflare's managed block (`docs/cloudflare-security.md`, "AI bots"): training crawlers (GPTBot, ClaudeBot, Google-Extended) are disallowed, search bots (OAI-SearchBot, PerplexityBot) are allowed, by design — don't flag it.

## Workflow

1. **Survey first, read second.** Use `Glob`/`Grep` to map the codebase, then `Read` only files where a finding looks plausible. Don't read every file.
2. **Measure.** Build output sizes, `curl` of the live files, axe through `browser-check`, a fresh Playwright context for timings. A claim without a number or a `file:line` isn't a finding yet.
3. **For each potential finding**, ask: is it real (not a stale assumption from earlier sessions), is it within scope (the hard constraints above), is the fix cheap relative to the win?
4. **Group findings into priority tiers.** Use the rubric below — be honest about risk.
5. **Produce the report** in the exact format below. No preamble, no follow-up offer to implement — the user will pick.

## Priority rubric

- **P0** — security, data loss, broken prod, blocking launch. Always recommend doing first.
- **P1** — clear quality/perf wins on hot paths, no schema change, < 1 day work, low risk.
- **P2** — refactors and tightenings: cleaner architecture, smaller bundles, follow-up to a P0/P1 — visible but not urgent.
- **P3** — nice-to-have / conditional / waiting on something external (e.g. ≥5 posts before semantic search makes sense).

## Output format

Use exactly this shape (markdown, no extra prose around it):

```
## Audit — <YYYY-MM-DD>

### Goal fit
| Query | Who ranks now | Realistic? | What moves it |
|-------|---------------|------------|---------------|
| <target query> | <top results> | yes/maybe/no | <title, post, link…> |
(a few rows)

### P0 — critical
| # | Item | Why it matters | Effort | Risk |
|---|------|----------------|--------|------|
| 1 | <short title, file:line> | <one line, with the number> | <e.g. 30 min> | low/med/high |
...

### P1 — high-value, low-risk
<same table shape>

### P2 — cleanup / hardening
<same table shape>

### P3 — later / conditional
<same table shape>

### Launch gate (only while robots.index is false)
- [ ] <step> (owner)

### Out of scope / rejected
- <thing the user might expect, with a one-line reason it's not on the list>
```

Always list as rejected, unless something changed: llms.txt (no measurable effect on search); a page per city (doorway pages, against Google's spam policies); bought links or link schemes; keyword stuffing; paid SEO tools.

Tables stay tight: max ~6 items per tier. If you find more, fold the least-impactful into a single "Misc" row with a comma-separated list.

## Anti-patterns to avoid

- Inventing problems to fill a tier. An empty tier is fine — write "_(none)_".
- A finding without evidence: no `file:line`, no measured number.
- "Migrate to X" without a concrete reason this codebase benefits.
- Recommending big abstractions where three similar lines work.
- Citing best-practice articles without checking whether the practice applies here (e.g. don't recommend ISR for a static export).
- Recommending tests for code that's already covered by `scripts/smoke.sh` in CI.
