---
name: seo-geo
description: Audit or wire up SEO (search engines) and GEO (generative/AI answer engines) fundamentals for a website — metadata, structured data, sitemaps, redirects, i18n URLs, Core Web Vitals, and AI-crawler access. Use when adding a page, renaming/removing a route, launching a new site or locale, or doing a pre-launch SEO/GEO check.
---

# SEO / GEO Fundamentals

*[中文版本 / Chinese version →](../seo-geo-zh/SKILL.md)*

Authoritative sources (these change — re-check, don't rely on memory):
[Google SEO starter guide](https://developers.google.com/search/docs/fundamentals/seo-starter-guide) ·
[Structured data intro](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data) ·
[Sitemaps](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap) ·
[Canonicalization](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls) ·
[hreflang / localized versions](https://developers.google.com/search/docs/specialty/international/localized-versions) ·
[robots.txt intro](https://developers.google.com/search/docs/crawling-indexing/robots/intro) ·
[Core Web Vitals](https://web.dev/articles/vitals) ·
[Schema.org](https://schema.org/) ·
[Google crawlers/user-agents overview](https://developers.google.com/search/docs/crawling-indexing/overview-google-crawlers) ·
[Anthropic crawlers](https://support.claude.com/en/articles/8896518) ·
[OpenAI GPTBot](https://platform.openai.com/docs/gptbot) ·
[llms.txt spec](https://llmstxt.org/) ·
[GEO: Generative Engine Optimization (paper)](https://arxiv.org/abs/2311.09735)

SEO (ranking in traditional search) and GEO (getting cited/quoted by AI answer engines — ChatGPT, Perplexity, Google AI Overviews, Claude) share most of their foundation: clean semantic HTML, real structured data, and substantive content beat tricks aimed at either audience. Where they diverge is called out explicitly in section 7. Go through every section below, not just whichever one prompted the request — most real-world regressions (duplicate URLs, dead links after a rename, content diluted by boilerplate) come from a change that only *looked* unrelated to SEO.

## 1. Metadata

- Every page needs a unique `<title>` and `<meta name="description">` — don't let a template silently repeat the same title across many URLs (a classic cause of "duplicate title" warnings in Search Console).
- Exactly one canonical URL per page (`<link rel="canonical">`), self-referencing on the canonical version. Don't emit a canonical that points to a different page's content, and don't emit more than one.
- If the site serves multiple languages/regions on separate URLs, every localized page needs full `hreflang` reciprocity: each language version lists *all* language versions (including itself), and the sets must match exactly — a one-directional or partial hreflang set is worse than none, because it tells search engines the versions disagree about which pages are alternates of each other.
- Open Graph (`og:title`/`og:description`/`og:image`/`og:url`) and Twitter Card tags control how the page looks when shared — missing these isn't a ranking issue but is a real, visible defect (broken/blank share previews).
- **The `<meta name="keywords">` tag does nothing** — Google has ignored it for ranking since 2009. Don't spend effort maintaining it; don't confuse it with the keyword *strategy* that should inform titles/headings/body copy instead.
- Centralize metadata generation in one function/module if the framework allows it (e.g. a shared `buildMetadata()`/`generateMetadata()` helper) — hand-writing these fields per page is exactly how hreflang/canonical gaps happen.

## 2. Structured Data (JSON-LD)

- Prefer JSON-LD (`<script type="application/ld+json">`) over microdata/RDFa — easiest to generate correctly and to keep out of visible markup.
- Sitewide entities (`Organization`, `WebSite`) go once, typically in the root layout — don't repeat them on every page.
- Per-page-type schemas: `WebPage` for generic pages, `Article`/`BlogPosting` + `BreadcrumbList` for content/detail pages, `FAQPage` for genuine Q&A content, `Product`/`Review` for commerce. Only mark up content that's actually visible on the page — Google's spam policies explicitly disallow structured data describing content the page doesn't contain.
- Required fields matter: a blank/placeholder `description` field is often silently worse than omitting the field — check what the real value resolves to (many bugs here come from a fallback chain that ends in an empty string instead of the next real fallback).
- Validate with the [Rich Results Test](https://search.google.com/test/rich-results) or [Schema Markup Validator](https://validator.schema.org/) after any change — a typo'd `@type` fails silently (renders as inert JSON, no error in the page).

## 3. Sitemap & robots.txt

- `sitemap.xml` should list only canonical, indexable, 200-status URLs. Never include: URLs that redirect, URLs marked `noindex`, or URLs that 404 — each one is a small credibility hit against the whole file, and crawlers do check.
- `lastmod` must come from a real modification timestamp. A fabricated "now" makes the field actively misleading (search engines verify it against actual re-crawls) and is worse than omitting it.
- `robots.txt` disallows crawling; it does not remove a URL from an index (a disallowed-but-linked URL can still appear in results with no snippet). Use `noindex` (meta tag or header) to actually keep a page out, not `robots.txt` alone.
- Never disallow CSS/JS/image paths that the page needs to render — this was a real, common mistake historically because renderers used to not execute JS; modern crawlers do render pages, and blocking assets can make the rendered version look broken or empty to the crawler.

## 4. URLs & Redirects

- Pick one canonical form and stick to it: `www` vs bare domain, trailing slash vs not, `http` vs `https` — redirect every non-canonical variant to the canonical one (301/permanent), don't rely on canonical tags alone to paper over inconsistent internal links.
- i18n: serve each locale at a real, independently crawlable URL (path prefix `/en/...`, `/zh/...`, or subdomain) — don't switch content by cookie/`Accept-Language` under one URL, since that makes only one version visible to crawlers/AI answer engines at all.
- **Renaming or removing a route is a multi-place change, not a one-liner.** Whichever of these apply to the stack in use, all of them need updating together — missing one produces either a dead link or a redirect loop:
  1. The redirect rules themselves (framework config, reverse proxy, edge middleware) — cover both the locale-prefixed and bare-path forms if both exist.
  2. Any hardcoded regex/allowlist that references the old path (edge middleware pre-checks, path-based feature flags).
  3. The sitemap generator.
  4. Hardcoded links in components/templates — grep the whole repo for the literal old path string, don't rely on remembering where it's used.
  5. Avoid chains: if A already redirects to B, a new rule sending C to A should instead send C straight to B.
- After the change, build for production and run a real server — don't stop at "the build succeeded." `curl -I` (or `curl -L -o /dev/null -w '%{http_code}\n'`) every old and new path and confirm real status codes: old paths land on a real 3xx or 410, new/unaffected paths land on 200, genuinely-gone content lands on 404 — not a soft-404 (200 with "not found" text) and not a redirect loop.

## 5. Core Web Vitals & Technical Basics

- Current thresholds (check [web.dev/vitals](https://web.dev/articles/vitals) for updates — these move): LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1, measured at the 75th percentile of real visits.
- Mobile-friendliness is table stakes, not a bonus check — Google indexes mobile-first, meaning the mobile-rendered version of the page is what actually gets evaluated.
- Don't block the initial render with unnecessary synchronous scripts; don't ship layout shifts from images/ads/fonts without reserved space (the direct cause of CLS regressions).

## 6. Internal Linking & Content Architecture

- Use descriptive anchor text (not "click here") — anchor text is a real signal for what the linked page is about, for both search and AI crawlers trying to understand site structure.
- Avoid orphan pages (reachable only via sitemap, not linked from anywhere crawlable) — link to new content from at least one relevant existing page.
- Breadcrumbs (both visible UI and `BreadcrumbList` structured data) help both users and crawlers understand hierarchy on deep sites.

## 7. GEO — Optimizing for AI Answer Engines

AI answer engines (ChatGPT/Perplexity/Google AI Overviews/Claude) mostly work by extracting and summarizing text, often with simpler, more naive extraction than a modern search-engine renderer. This creates failure modes that don't show up in traditional SEO audits:

- **Boilerplate dilution is the single most common real-world GEO defect.** If the same nav/footer/menu text is duplicated in the DOM multiple times — e.g. writing one full copy of navigation per responsive breakpoint and toggling visibility with CSS classes instead of one DOM copy + responsive CSS — every one of those copies gets counted as "page text" by a naive extractor, burying the actually-unique content under repeated boilerplate. Audit: fetch the rendered HTML with a plain HTTP client (not a browser) and count occurrences of a distinctive phrase from the nav/footer; it should appear once.
- **`display:none`/`visibility:hidden`/`opacity:0` does not hide content from a text extractor** — the text is still in the HTML. For content that's genuinely conditional (a closed dropdown, an unopened accordion), prefer mounting it only when opened (`{isOpen && <Content/>}`) over always-rendering it and hiding it with CSS, unless it must be in the initial DOM for accessibility/SEO reasons at the current breakpoint.
- **Depth and directness beat cleverness.** Thin pages (skeleton "about us"/"contact"/policy pages with a few dozen words) give an extractor nothing to cite. Answer the actual question a visitor/AI would come to the page with, early and explicitly — this is also just good SEO, but it's a harder requirement for GEO since there's no partial credit for "the info exists somewhere on the site" the way there sometimes is for ranking.
- **Cite sources and avoid stale numbers presented as current.** A specific figure (a price, a rate, a technical parameter) that used to be true but is now wrong is actively worse than not stating it, because an LLM may repeat it as fact with no way for the reader to know it's outdated. Prefer linking to the authoritative live source (an API, a blockchain explorer, an official register) over hardcoding a number that will drift.
- **Prefer real prose over HTML-escaped strings jammed into JSON/i18n files** for anything long-form — Markdown/plain-text source is both easier to keep accurate and easier for extractors to parse cleanly; heavily-escaped inline HTML strings are also more likely to end up mangled or duplicated across breakpoints (see the first bullet).
- **AI crawler access is a separate decision from search-engine access** — control it explicitly in `robots.txt` rather than assuming your existing rules cover it:
  - Training crawlers (feed model training, not tied to a specific user query): `GPTBot` (OpenAI), `ClaudeBot` (Anthropic), `Google-Extended` (Google), `Applebot-Extended` (Apple), `Meta-ExternalAgent` (Meta), `CCBot` (Common Crawl).
  - Retrieval/citation crawlers (fetch a page live in response to a user's question — blocking these directly removes you from that answer's citations): `OAI-SearchBot`/`ChatGPT-User` (OpenAI), `Claude-SearchBot`/`Claude-User` (Anthropic), `PerplexityBot`/`Perplexity-User` (Perplexity).
  - A common deliberate strategy: block the training crawlers, allow the retrieval/citation crawlers — this opts out of being training data while staying eligible to be cited in live answers. There is no single universally-correct choice; state which one the site is making and why.
  - Verify actual crawler identity against the vendor's published IP list when it matters (e.g. [Anthropic's bots.json](https://claude.com/crawling/bots.json)) — user-agent strings alone can be spoofed by unrelated bots.
- **`llms.txt`** is an emerging (not yet universally adopted) convention: a plain Markdown file at `/llms.txt` giving AI agents a curated, concise map of the site's key content — most valuable for documentation-heavy or agent-facing sites; skip it for a simple marketing site where the sitemap and clean page content already cover the same need.
- Structured data (section 2) helps GEO too — `Organization`/`Article`/`FAQPage` markup gives an LLM an unambiguous, machine-readable answer to "what is this entity/page," reducing the chance of misattribution or a wrong summary.

## 8. Checklist for a New Page or Route Change

1. Metadata: unique title/description, canonical, full hreflang set if localized, OG/Twitter tags.
2. Structured data: the right per-page-type JSON-LD, validated with the Rich Results Test.
3. Sitemap: add the new canonical path (or confirm it's covered by an existing slug-enumeration route); confirm no non-canonical/redirecting URL got added.
4. If replacing/removing an old route: work through the full multi-place redirect checklist in section 4 — don't stop after the main router config.
5. Content: real, specific, substantive body text per section 7 — not a boilerplate skeleton, and not navigation structure copy-pasted as "content."
6. Verify with a production build + real server: `curl` every old and new path, confirm actual HTTP status codes — a green build is not a passing test.
7. If the site wants AI-answer-engine visibility either way, confirm the new path isn't accidentally caught by an existing broad `robots.txt` disallow rule aimed at a different crawler.
