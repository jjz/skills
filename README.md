# jjz-skills

Open-source [Claude Code](https://claude.com/claude-code) skills, distributed as a plugin marketplace. English is the default/primary language for this project; some skills also ship a full Chinese translation as a separate, independently invokable skill.

## Install

In Claude Code:

```
/plugin marketplace add jjz/skills
/plugin install adsense-review
/plugin install seo-geo
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
