# jjz-skills

Open-source [Claude Code](https://claude.com/claude-code) skills, distributed as a plugin marketplace. English is the default/primary language for this project; some skills also ship a full Chinese translation as a separate, independently invokable skill.

## Install

In Claude Code:

```
/plugin marketplace add jjz/skills
/plugin install adsense-review
/plugin install seo-geo
/plugin install perf-optimization
/plugin install dev-testing
```

(Replace `jjz/skills` with the actual GitHub path once published, e.g. `github.com/jjz/skills` or `jjz/skills` if using the short form Claude Code resolves against GitHub.)

## Available plugins

### adsense-review

Audit a website against Google AdSense/Publisher Policies (content, behavior, privacy, technical requirements), item by item. Useful for a general AdSense compliance review, investigating a specific ad policy violation notice, or a pre-launch check before enabling ads on a new page.

Ships two independently invokable skills:
- `adsense-review` — English (default)
- `adsense-review-zh` — 中文完整版

See [`plugins/adsense-review/skills/adsense-review/SKILL.md`](./plugins/adsense-review/skills/adsense-review/SKILL.md) for the full content.

### seo-geo

Audit or wire up SEO (search engines) and GEO (generative/AI answer engine) fundamentals for a website — metadata, structured data, sitemaps, redirects, i18n URLs, Core Web Vitals, and AI-crawler access control. Useful when adding a page, renaming/removing a route, launching a new site or locale, or doing a pre-launch SEO/GEO check.

Ships two independently invokable skills:
- `seo-geo` — English (default)
- `seo-geo-zh` — 中文完整版

See [`plugins/seo-geo/skills/seo-geo/SKILL.md`](./plugins/seo-geo/skills/seo-geo/SKILL.md) for the full content.

### perf-optimization

Audit and improve Core Web Vitals (LCP/CLS/INP) and page weight for any website — images, third-party scripts, JS bundle size, fonts, and caching. Useful before/after adding hero images, carousels, embeds, third-party scripts, or content-heavy pages, or for a general PageSpeed/Lighthouse review.

Ships two independently invokable skills:
- `perf-optimization` — English (default)
- `perf-optimization-zh` — 中文完整版

See [`plugins/perf-optimization/skills/perf-optimization/SKILL.md`](./plugins/perf-optimization/skills/perf-optimization/SKILL.md) for the full content.

### dev-testing

Plan and write unit, integration, and end-to-end tests across languages — the test pyramid (L1/L2/L3), docker-compose-bound integration tests, per-language runners (Jest/Vitest/Bun test, pytest, Go, Rust, Solidity), Playwright E2E, and PDPO-safe test data. Useful when choosing what to test for a change, writing or reviewing tests, or debugging a flaky/slow suite. A developer-facing skill, separate from the website/SEO skills above.

Ships two independently invokable skills:
- `dev-testing` — English (default)
- `dev-testing-zh` — 中文完整版

See [`plugins/dev-testing/skills/dev-testing/SKILL.md`](./plugins/dev-testing/skills/dev-testing/SKILL.md) for the full content.

## Repository layout

```
.claude-plugin/marketplace.json   — marketplace manifest listing all plugins below
plugins/<plugin-name>/
  .claude-plugin/plugin.json      — plugin manifest
  skills/<skill-name>/SKILL.md    — one directory per skill, auto-discovered
```

## Contributing

Each skill should be self-contained, itemized, and grounded in primary sources (link to the actual policy/doc pages rather than restating them from memory) so it stays useful as source policies change. Skills that are relevant in both English and Chinese should ship as two parallel, independently invokable skill directories (see `adsense-review` / `adsense-review-zh`) rather than one bilingual file, since Claude Code discovers skills by scanning for `SKILL.md` files by directory.

## License

MIT — see [LICENSE](./LICENSE).
