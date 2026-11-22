# Front end (components/)

Traps only. Conventions, widgets, counters, contact form, e2e:
`docs/reference/frontend.md`; file purposes: `node scripts/check-docs.mjs
--map`.

- The 2001 theme is load-bearing: intro `HEI %USERNAME%` (a literal joke),
  HAL idle/eye, the 404 and `error.tsx` copy, loader lines. Don't "fix" them.
  The status caption stays quiet, «Siste 149 sjekker» (owner, 2026-11-11).
  The intro plays for every first-time visitor, search referrals included:
  never propose skipping it for a group (owner, 2026-11-22).
- The hero has no `animate-fade-in` (opacity-0 delays LCP). Skill chips
  highlight on hover though not links: keep (owner).
- Accent TEXT is red-600 light / red-400 dark (red-500 fails AA as text).
  Secondary links set `prefetch={false}`.
- A theme pick equal to the OS's isn't stored (flipping back = back to the
  OS). `HashLink` scrolls natively (`scrollIntoView`, CSS
  `scroll-padding-top`, no rAF); keep `<html data-scroll-behavior="smooth">`.
- Every `/api/*` call is a Worker request: `cache: 'no-store'`, share fetches
  (`fetchGeoCountries`, `fetchStatus`, `fetchSeries`), mount and fetch only near the viewport
  (`useNearViewport`; `Lazy*.tsx` = `lazyNearViewport(() => import(...), Placeholder)`,
  placeholders keep the final height), poll
  only what's on screen.
- The Status/Visitors/Trends headings stay in server components (smoke greps).
- "Now" is the server's (`serverNow()`), never the visitor's clock;
  `VisitBeacon` alone uses the visitor's own day.
- Client counters count people: first trusted human input, never under
  `navigator.webdriver`, the dedupe flag set BEFORE the fetch (StrictMode,
  remounts, second tab). `e2e/counters.e2e.ts` holds both counters to it; the
  `visit-day` / `viewed:<slug>` storage dedupe stays (owner). Absolute numbers
  hide below `MIN_PUBLIC_VISITS` / `MIN_SHOWN`.
- Trends' SVG `viewBox` is its rendered width, never fixed. Map: never set a
  React-managed node's innerHTML by ref; keep BOTH the baked grey fill/stroke
  and the CSS `var()` fallbacks (an unstyled path is black).
- Contact form: Turnstile tokens are single-use, so reset the widget after
  every non-OK response and keep the typed text. A secret without a site key
  403s every POST. The honeypot is `HONEYPOT_FIELD` (`topic`); limits come from
  `lib/shared/contact.ts`. No-JS can't send: a `<noscript>` note, `method="post"`.
- Blog tables/code: keep the `.post-body` wrap and hyphenation rules.
- e2e: a UI change that breaks a selector updates the test in the same PR;
  blog tests take their post from the build's feeds, never by name; local
  builds need `NEXT_PUBLIC_TURNSTILE_SITE_KEY=1x00000000000000000000AA`;
  `@playwright/test` and `playwright-core` stay on one version.
