---
name: perf-optimization
description: Audit and improve Core Web Vitals (LCP/CLS/INP) and page weight for any website — images, third-party scripts, JS bundle size, fonts, and caching. Use when asked to optimize page performance/PageSpeed/Lighthouse scores, or before/after adding hero images, carousels, embeds, third-party scripts, or content-heavy pages.
---

# Website Performance Optimization

*[中文版本 / Chinese version →](../perf-optimization-zh/SKILL.md)*

Authoritative sources (these change — re-check, don't rely on memory):
[web.dev Core Web Vitals](https://web.dev/articles/vitals) ·
[web.dev LCP](https://web.dev/articles/lcp) ·
[web.dev INP](https://web.dev/articles/inp) ·
[web.dev CLS](https://web.dev/articles/cls) ·
[web.dev responsive images](https://web.dev/articles/serve-responsive-images) ·
[web.dev font best practices](https://web.dev/articles/font-best-practices) ·
[PageSpeed Insights](https://pagespeed.web.dev/) ·
[MDN: Resource hints (preload/preconnect/prefetch)](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Resource_hints) ·
[HTTP Archive / caching guidance](https://web.dev/articles/http-cache).

Before applying anything below, check the actual stack (framework, meta-framework version, image/CDN pipeline, bundler) rather than assuming — API names and defaults for things like image components, script-loading directives, and font loaders change between major versions and between frameworks, and applying a stale or wrong-framework API is a common source of "fixes" that don't compile or silently no-op. If the repo has its own docs/AGENTS.md/CLAUDE.md calling out framework-version quirks, read those first.

**Lesson**: a shallow performance pass that only checks "is an image component being used at all" misses two subtler, higher-impact classes of bugs: (a) load-priority hints (`priority`/`fetchpriority`/`preload`) applied to the wrong image — an off-screen image marked high-priority competes with the real hero image and makes both slower; (b) a heavy library (rich-text/markdown renderer, carousel, charting, syntax highlighting) imported unconditionally in code that ships to the client, so pages that never use it still pay to download it. Go through every section below on each review, don't just spot-check whether images use an optimized component.

## 1. Images (highest-leverage for LCP/bandwidth — check first)

Find every image tag/component in the codebase (`<img>`, the framework's image component, CSS `background-image`) and check each one:

1. **Is it going through an actual optimization pipeline** (the framework's image component, an image CDN, a build-time optimizer) or is it a raw `<img>`/CSS background pointing straight at a full-size file? A raw tag isn't automatically wrong — a small icon, or an image from a domain the optimizer can't reach, is a legitimate reason — but it should be a deliberate, documented exception, not the default. Grep for existing "intentionally unoptimized" markers (a comment, a lint-disable) before "fixing" one; changing it without also fixing the underlying reason (e.g. adding the domain to the image allowlist) can break the build.
2. **Priority/preload hints must go on the actual LCP candidate, not on whatever "seems important."** Only the single largest above-the-fold image (almost always the hero) should be eagerly/high-priority loaded; every other image should lazy-load. Judge this by rendering order and viewport visibility, not by how visually prominent a component *feels* in the source. A real recurring bug: a background image several sections down the page gets marked high-priority, which steals loading priority from the true hero image — both end up slower. In a carousel/list of images, the priority hint belongs on the first/active slide only — never on every item in a `.map()`/loop, since multiple simultaneous "highest priority" images is equivalent to none.
3. **Responsive images need a correct `sizes` (or equivalent) attribute, not a lazy default.** When an image scales with its container, `sizes` should describe the image's actual rendered width at each breakpoint — not the raw source dimensions. Don't reflexively write `sizes="100vw"` for anything with a full-width-looking class: check the nearest ancestor that constrains max-width (a centered `max-width: 880px` wrapper, a fixed-width card) and describe *that*. A wrong `sizes` causes the browser to request a much larger generated image than what's actually rendered — verify by checking the requested image's dimensions against the element's rendered box in DevTools, not by reading the class name and guessing.
4. **Format/quality settings**: prefer modern formats (AVIF/WebP) with a fallback chain the optimizer generates automatically rather than hand-maintaining multiple `<picture>` sources; know whether the pipeline enforces a fixed quality allowlist (some do) versus accepting any quality value.
5. **Check for oversized source files that never actually reach the optimizer**: raw source size doesn't equal bytes shipped to users if everything routes through an optimizing pipeline, but two failure modes still matter — (a) a huge source file referenced by a raw `<img>`/CSS background, which *does* ship at full size; (b) an orphaned huge file sitting in the public/static assets directory that nothing references at all. When searching for references, also check content/i18n/CMS data files, not just component source — image paths are frequently built from data (i18n JSON, CMS fields) rather than hardcoded in markup, and a component-only grep will miss those.

## 2. Third-party scripts & embeds

- Classify every third-party script by how soon it needs to run, and load it no earlier than that: analytics/ads that don't block rendering → load after the page is interactive; chat widgets, social embeds, anything not needed for the initial view → load lazily/on-idle or on-interaction; only truly render-blocking necessities (a cookie-consent gate, bot/fraud detection that must run before anything else) get synchronous/early loading, and even those should be as few and as small as possible.
- Any external data call used to gate rendering or personalize content (geolocation, consent status, A/B assignment) should be async and non-blocking for the initial paint — fetch it in an effect/after mount, don't await it before rendering the page shell.
- Self-host a script if licensing/ToS allow it and the vendor doesn't require their CDN for auto-updates — removes a DNS lookup + connection, and gets it under the site's own caching policy. Otherwise use `preconnect`/`dns-prefetch` for third-party origins that will definitely be used.

## 3. JS bundle size / code splitting

- Grep for actual import sites of every heavy dependency (a rich-text/markdown renderer, a carousel library, a charting/animation library, a syntax highlighter, a PDF/office-doc viewer) — a dependency listed in the manifest but never imported anywhere doesn't ship (tree-shaking handles that), but it's worth flagging as likely dead/leftover code in a report; don't unilaterally delete it or the component that used it without asking, since it may be intentionally kept for near-term reuse.
- **Whether code executes on the server or ships to the client is usually the highest-leverage lever, bigger than manual code-splitting.** In frameworks with a server/client component split (React Server Components, Astro islands, Vue server components, etc.): a heavy library used only inside a component that never needs client-side interactivity costs nothing if that component stays server-rendered-only. The instant that whole subtree is rendered under a client-boundary component — even if the heavy-library component itself has no client directive — everything under it ships to the browser. So: first identify whether a client boundary was actually necessary (real event handlers/hooks/browser APIs) — if not, removing it is strictly better than any downstream optimization. If a client boundary is genuinely required (e.g. to avoid a hydration/layout mismatch across breakpoints) but only *some* code paths inside it need the heavy library, wrap just that import in the framework's dynamic/lazy-import primitive so unrelated pages/branches don't download it. Keep server-side rendering enabled for any content that needs to be crawlable by search engines/AI answer engines (see the seo-geo skill) — disabling SSR to save client bytes trades a bundle-size win for an indexability loss.
- In frameworks without an SSR/CSR split (a plain SPA, a static site with vanilla JS), the equivalent move is route-based or interaction-based code splitting (dynamic `import()` behind a route change or a user action) instead of one monolithic entry bundle.

## 4. Fonts

- Self-host fonts (or use the framework's built-in font-loading primitive, which typically self-hosts automatically) instead of a hand-written `<link>`/`@import` to an external font host — avoids an extra cross-origin connection and lets the font ship with the same caching/CDN policy as the rest of the site.
- Watch for the same font being loaded twice from two different places (e.g. a duplicate font-loader call left behind in a layout file that turned out not to actually render `<html>`/`<body>` after a refactor) — a real, recurring bug class whenever layout/template nesting changes. Grep for all font-loading calls in the codebase after any layout restructuring.
- Prefer a variable font over shipping a separate file per weight/style when the format supports it.
- Set `font-display: swap` (or the loader's equivalent) so text renders in a fallback font immediately rather than staying invisible while the webfont downloads; most modern font-loading helpers default to this already — verify rather than assume.
- Subset fonts to the character sets actually used (especially relevant for CJK fonts, which are enormous unsubsetted) when the tooling supports it.

## 5. CSS & render-blocking resources

- Don't ship a large monolithic CSS bundle to every page when a bundler/framework can generate per-route or per-component styles instead — unused CSS on a page is bytes and parse time the browser pays for nothing.
- Critical above-the-fold CSS should be available without waiting on a blocking network request where the build tooling supports inlining it; non-critical CSS can load asynchronously.
- Reserve layout space for anything that loads asynchronously and has a size (images, ads, embeds, web fonts) — this is the direct, most common cause of CLS regressions. An explicit width/height (or `aspect-ratio`) on the element beats a CSS trick applied after the fact.

## 6. Caching & delivery

- Static assets (images, fonts, hashed JS/CSS bundles) should carry long-lived, immutable cache headers (`Cache-Control: public, max-age=31536000, immutable`) — safe because the build hashes the filename on content change.
- HTML/API responses need cache headers deliberately chosen for how often the content actually changes — don't let a framework's default silently cache something that needs to be fresh, or leave something cacheable with no cache headers at all.
- Confirm compression is actually active end-to-end (Brotli preferred, gzip fallback) — check response headers in production, not just that the hosting platform claims to support it.
- Serve static assets and images from a CDN/edge network when the hosting setup allows it, so round-trip time doesn't scale with the visitor's distance from a single origin server.

## 7. Mobile

Treat mobile as the default target, not an afterthought checked last — for most sites it's where most real traffic and most Core Web Vitals field data comes from, and it's the harder environment (slower CPU, slower/metered network), so a page that's fast on mobile is fast everywhere but not vice versa.

- **PageSpeed Insights/CrUX field data is mobile-heavy by default**, and Lighthouse's default mobile profile applies both CPU and network throttling — a desktop-only Lighthouse run can show a comfortably passing LCP/INP that never reflects what most real visitors experience. Always run (and report) the mobile profile, not just desktop.
- **Mobile CPUs are commonly 4–6x slower main-thread throughput** than a developer's laptop, which is why heavy JS that looks harmless when profiled unthrottled on desktop can blow the INP budget on a mid-range phone. Profile with CPU throttling enabled (Chrome DevTools' "4x slowdown"/"6x slowdown" or Lighthouse's mobile preset), not only unthrottled.
- **`<meta name="viewport" content="width=device-width, initial-scale=1">` must be present and correct** — a missing or wrong viewport tag makes mobile browsers render the page at a desktop width and scale it down, which both looks broken and produces misleading (or badly regressed) CLS/LCP measurements. Check this once per project as a baseline, especially on any custom/non-framework-generated HTML shell.
- **Responsive image breakpoints (section 1.3) need to actually cover mobile viewport widths**, not just be a `sizes` string that happens to resolve correctly at desktop widths — verify the generated `srcset` includes small-enough candidates that a narrow phone screen doesn't download a desktop-sized image over a slow connection.
- **Page weight is a stricter budget on mobile** than "does it feel fast on office wifi" — a metered/slow mobile connection makes every extra KB of JS/image/font payload cost real, felt latency; when in doubt about whether to lazy-load or defer something, the mobile case is the one to optimize for.
- **INP on mobile is dominated by touch/tap responsiveness**, not hover states (which mostly don't exist on touch devices) — audit tap/click handlers for synchronous heavy work (large state recalculation, unthrottled re-renders) rather than assuming desktop hover-interaction profiling covers the same ground.

## 8. Verification

Don't stop at "the diff looks like it should be faster." Build for production and run the production build locally or on a real deploy preview (a dev-mode server has different — usually worse and non-representative — performance characteristics). Then:

- Run Lighthouse/PageSpeed Insights against the production build **using the mobile device/throttling profile as the primary check** (see section 7), before and after, and compare LCP/CLS/INP directly — not just the overall score, which can mask a regression in one metric offset by an improvement in another. Run the desktop profile too if desktop traffic is significant, but don't let a passing desktop score stand in for a mobile check.
- Use the browser DevTools Performance/Network panels (with mobile CPU/network throttling enabled) to confirm which resource is actually the LCP element and when it starts/finishes loading, especially after any hero-image or above-the-fold change.
- Lab data (Lighthouse) and field data (real-user Core Web Vitals, e.g. from CrUX/PageSpeed Insights' field section, or a RUM tool already wired into the site) can disagree — trust field data for whether real visitors are actually affected, and treat a lab-only improvement with the right amount of skepticism until field data confirms it. Check the field data's device breakdown if available — a mobile-specific regression can hide inside an aggregate "all devices" number that desktop improvements offset.
- If the change touches content that renders differently across breakpoints or client/server boundaries, also verify no duplicate content was introduced in the rendered HTML (see the seo-geo skill, sections 6–7) — a common side effect of "fixing" a layout-shift or hydration issue by rendering something twice and toggling visibility with CSS.
