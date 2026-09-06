---
name: seo-geo-zh
description: 审查或搭建网站的 SEO（传统搜索引擎）与 GEO（生成式/AI 问答引擎）基础能力——元数据、结构化数据、sitemap、重定向、多语言 URL、Core Web Vitals、AI 爬虫访问控制。用于新增页面、改名/下线路由、上线新站点或新语言版本，或做上线前的 SEO/GEO 检查。
---

# SEO / GEO 基础实践

*[English version →](../seo-geo/SKILL.md)*

权威来源（这些会更新，不要凭记忆）：
[Google SEO 新手指南](https://developers.google.com/search/docs/fundamentals/seo-starter-guide) ·
[结构化数据简介](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data) ·
[Sitemap 指南](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap) ·
[规范化 URL](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls) ·
[hreflang / 本地化版本](https://developers.google.com/search/docs/specialty/international/localized-versions) ·
[robots.txt 简介](https://developers.google.com/search/docs/crawling-indexing/robots/intro) ·
[Core Web Vitals](https://web.dev/articles/vitals) ·
[Schema.org](https://schema.org/) ·
[Google 爬虫/User-Agent 总览](https://developers.google.com/search/docs/crawling-indexing/overview-google-crawlers) ·
[Anthropic 爬虫说明](https://support.claude.com/en/articles/8896518) ·
[OpenAI GPTBot](https://platform.openai.com/docs/gptbot) ·
[llms.txt 规范](https://llmstxt.org/) ·
[GEO: Generative Engine Optimization（论文）](https://arxiv.org/abs/2311.09735)

SEO（在传统搜索引擎里排名）和 GEO（被 ChatGPT/Perplexity/Google AI Overviews/Claude 这类 AI 问答引擎引用/摘录）的基础大部分是共通的：干净的语义化 HTML、真实的结构化数据、有实质信息量的正文，比任何针对单一目标的"技巧"都管用。两者分歧的地方在第 7 节明确列出。下面每一节都要过一遍，不要只看和这次请求"看起来相关"的那一节——大部分实际发生的回归问题（重复 URL、改名后出现死链、正文被样板文字稀释）都来自一处"看起来跟 SEO 无关"的改动。

## 1. 元数据

- 每个页面要有唯一的 `<title>` 和 `<meta name="description">`——不要让模板在一堆 URL 上悄悄输出同一句标题（Search Console 里"重复标题"警告的经典成因）。
- 每个页面只有一个 canonical URL（`<link rel="canonical">`），规范版本自引用自己。不要让 canonical 指向另一个页面的内容，也不要同时输出多个 canonical。
- 如果站点用不同 URL 提供多语言/多地区版本，每个本地化页面都需要完整的 `hreflang` 互相引用：每个语言版本要列出**全部**语言版本（包括自己），且各版本列出的集合必须完全一致——单向或不完整的 hreflang 集合比完全不写还糟，因为这等于告诉搜索引擎这几个版本互相不认可对方是自己的替代版本。
- Open Graph（`og:title`/`og:description`/`og:image`/`og:url`）和 Twitter Card 标签决定分享出去的卡片长什么样——这不是排名问题，但缺了会是一个真实可见的缺陷（分享预览空白/损坏）。
- **`<meta name="keywords">` 标签毫无作用**——Google 从 2009 年起就不用它排名。不用花力气维护这个标签；也别把它和真正该体现在标题/标题层级/正文里的关键词**策略**混为一谈。
- 如果框架支持，把元数据生成收敛到一个函数/模块里（比如统一的 `buildMetadata()`/`generateMetadata()`）——每个页面手写这几个字段正是 hreflang/canonical 出现遗漏的常见原因。

## 2. 结构化数据（JSON-LD）

- 优先用 JSON-LD（`<script type="application/ld+json">`），不用 microdata/RDFa——最容易写对，也最不容易污染可见标记。
- 全站级实体（`Organization`、`WebSite`）只放一次，通常放在根布局里——不要每个页面都重复插入。
- 按页面类型选择 schema：普通页面用 `WebPage`；内容/详情页用 `Article`/`BlogPosting` + `BreadcrumbList`；真正的问答类内容用 `FAQPage`；电商用 `Product`/`Review`。只标记页面上真实存在的内容——Google 的垃圾内容政策明确禁止结构化数据描述页面里不存在的内容。
- 必填字段要较真：一个空白/占位符式的 `description` 字段往往比直接不写这个字段更糟——要检查这个值实际解析出来是什么（这里的很多 bug 来自一条兜底链最终落到空字符串，而不是下一个真实兜底值）。
- 改动后用 [Rich Results Test](https://search.google.com/test/rich-results) 或 [Schema Markup Validator](https://validator.schema.org/) 校验——`@type` 写错一个字母会静默失败（渲染成一段无效 JSON，页面上不会报错）。

## 3. Sitemap 与 robots.txt

- `sitemap.xml` 里只放规范的、可索引的、状态码 200 的 URL。绝不放：会重定向的 URL、标了 `noindex` 的 URL、会 404 的 URL——每一条都是对整份文件可信度的一次小损耗，爬虫确实会核实。
- `lastmod` 必须来自真实的修改时间戳。编一个"现在"的假时间戳会主动误导（搜索引擎会拿它跟真实的重新抓取结果核对），比不写这个字段更糟。
- `robots.txt` 只是禁止抓取，不等于把 URL 从索引里移除（一个被禁止抓取但仍被链接到的 URL 依然可能出现在搜索结果里，只是没有摘要）。真正要让页面不被收录，用 `noindex`（meta 标签或响应头），不能只靠 `robots.txt`。
- 不要禁止抓取页面渲染需要的 CSS/JS/图片路径——这在历史上是个真实常见的错误，因为过去的抓取器不执行 JS；现在的主流爬虫会渲染页面，屏蔽资源会让爬虫看到的渲染结果显得残缺或空白。

## 4. URL 与重定向

- 选定一种规范形式并坚持下去：`www` 还是裸域名、带不带结尾斜杠、`http` 还是 `https`——把所有非规范变体都 301（永久）重定向到规范版本，不要只靠 canonical 标签去掩盖内部链接本身写得不一致的问题。
- 多语言：每个语言版本要有真实的、可独立抓取的 URL（路径前缀 `/zh/...`、`/en/...`，或子域名)——不要在同一个 URL 下靠 cookie/`Accept-Language` 切换内容，那样搜索引擎/AI 问答引擎实际上只能看到其中一个版本。
- **改名或下线一个路由是一次涉及多处的改动，不是改一行就完事。** 当前技术栈里用到的以下几处要一起改，漏掉一处就会出现死链或重定向死循环：
  1. 重定向规则本身（框架配置、反向代理、edge 中间件）——如果同时存在带语言前缀和裸路径两种形式，两种都要写。
  2. 任何写死了旧路径的正则/白名单（edge 中间件的预检逻辑、基于路径的功能开关）。
  3. sitemap 生成逻辑。
  4. 组件/模板里硬编码的链接——全仓库搜一遍旧路径的字面字符串，不要只凭记忆改自己想得到的几处。
  5. 避免链式重定向：如果 A 已经重定向到 B，新增一条 C 的规则应该直接指向 B，而不是指向 A。
- 改完之后要用生产构建 + 真实服务器验证——"构建成功"不是测试通过。对每个新旧路径 `curl -I`（或 `curl -L -o /dev/null -w '%{http_code}\n'`），确认真实状态码：旧路径落在真正的 3xx 或 410、新路径/未受影响路径落在 200、确实已下线的内容落在 404——不能是"假 404"（状态码 200 但文字显示"未找到"），也不能是重定向死循环。

## 5. Core Web Vitals 与技术基础

- 当前阈值（去 [web.dev/vitals](https://web.dev/articles/vitals) 核实最新值，这些会变）：LCP ≤ 2.5s、INP ≤ 200ms、CLS ≤ 0.1，以真实访问的第 75 百分位衡量。
- 移动端友好是基本门槛，不是加分项——Google 索引是移动优先的，也就是说页面在移动端渲染出的版本才是真正参与评估的版本。
- 不要用不必要的同步脚本阻塞首屏渲染；图片/广告/字体不要在没预留空间的情况下引起布局跳动（CLS 恶化的直接原因）。

## 6. 内部链接与内容结构

- 用有描述性的锚文本（不要用"点击这里"）——锚文本是搜索引擎和 AI 爬虫理解被链接页面内容、以及理解站点结构的真实信号。
- 避免孤立页面（只能通过 sitemap 到达、没有任何可抓取的页面链接过去）——新内容至少要从一个相关的现有页面链接过去。
- 面包屑导航（可见 UI + `BreadcrumbList` 结构化数据）能帮用户和爬虫理解层级较深站点的结构。

## 7. GEO——面向 AI 问答引擎的优化

AI 问答引擎（ChatGPT/Perplexity/Google AI Overviews/Claude）大多是靠抽取和摘要文本工作的，抽取逻辑往往比现代搜索引擎的渲染器更朴素、更粗暴。这会带来一些传统 SEO 审查查不出来的问题：

- **样板文字稀释是实际发生频率最高的 GEO 缺陷。** 如果同一段导航/页脚/菜单文字在 DOM 里被重复了好几份——比如每个响应式断点各写一份完整的导航，靠 CSS class 切换显示哪一份，而不是"一份 DOM + 响应式 CSS"——朴素的抽取工具会把每一份都算作"页面正文"，把真正独特的内容淹没在重复的样板文字里。检查方法：用普通 HTTP 客户端（不是浏览器）抓渲染后的 HTML，数一段导航/页脚里有辨识度的短语出现了几次，应该只出现一次。
- **`display:none`/`visibility:hidden`/`opacity:0` 对文本抽取工具不生效**——文字依然在 HTML 里。对于真正按条件显示的内容（未展开的下拉菜单、未打开的折叠面板），优先做成"打开时才挂载"（`{isOpen && <Content/>}`），而不是"一直渲染、只用 CSS 隐藏"，除非当前断点下出于可访问性/SEO 的原因必须在初始 DOM 里就存在。
- **深度和直接性比技巧更重要。** 骨架式的浅内容页面（"关于我们"/"联系方式"/政策页只有几十个字）没什么可供抽取引用的东西。尽早、明确地回答访客/AI 带着来到这个页面时真正想问的问题——这本身也是良好的 SEO 实践，但对 GEO 是更硬性的要求，因为不像排名有时候"信息存在于站内某处"能拿到部分分数，GEO 没有这种"部分分数"。
- **引用来源，不要把过期数字当成当前数字呈现。** 一个曾经正确、现在已经过期的具体数字（价格、费率、技术参数）比完全不写更糟，因为 LLM 可能把它当作事实原样复述，读者根本无从判断它已经过期。优先链接到权威的实时来源（一个 API、一个区块浏览器、一个官方登记处），而不是硬编码一个会漂移的数字。
- **长篇内容优先用真正的 Markdown/纯文本源文件，而不是转义成字符串塞进 JSON/i18n 文件**——Markdown 源文件既更容易保持内容准确，也更容易被抽取工具干净地解析；大段转义的行内 HTML 字符串也更容易在不同断点之间被改错或重复（见第一条）。
- **AI 爬虫的访问权限是一个独立于搜索引擎爬虫的决策**——要在 `robots.txt` 里明确控制，不要假设现有规则已经覆盖了它：
  - 训练类爬虫（喂给模型训练，不针对某一次具体提问）：`GPTBot`（OpenAI）、`ClaudeBot`（Anthropic）、`Google-Extended`（Google）、`Applebot-Extended`（Apple）、`Meta-ExternalAgent`（Meta）、`CCBot`（Common Crawl）。
  - 检索/引用类爬虫（响应用户的一次具体提问实时抓取页面——直接屏蔽这些等于把自己从那次回答的引用来源里去掉）：`OAI-SearchBot`/`ChatGPT-User`（OpenAI）、`Claude-SearchBot`/`Claude-User`（Anthropic）、`PerplexityBot`/`Perplexity-User`（Perplexity）。
  - 一个常见的有意为之的策略：屏蔽训练类爬虫、放行检索/引用类爬虫——这样既不贡献训练数据，又保留了被实时回答引用的资格。这里没有唯一正确答案，关键是明确站点选的是哪一种、为什么。
  - 有需要时，拿厂商公开的 IP 列表核实爬虫真实身份（例如 [Anthropic 的 bots.json](https://claude.com/crawling/bots.json)）——单看 User-Agent 字符串是可以被无关爬虫伪造的。
- **`llms.txt`** 是一个新兴（尚未被普遍采纳）的约定：在 `/llms.txt` 放一个纯 Markdown 文件，给 AI agent 一份精选、简洁的站内关键内容地图——对文档类或面向 agent 的站点最有价值；对一个简单的营销站点，如果 sitemap 和干净的页面内容已经覆盖了同样的需求，可以不做。
- 第 2 节的结构化数据对 GEO 同样有用——`Organization`/`Article`/`FAQPage` 这类标记能给 LLM 一个无歧义、机器可读的"这是什么实体/页面"的答案，降低被误判或摘要出错的概率。

## 8. 新增页面 / 路由变更检查表

1. 元数据：唯一的标题/描述、canonical、如果做了本地化则要完整的 hreflang 集合、OG/Twitter 标签。
2. 结构化数据：按页面类型选对 JSON-LD，用 Rich Results Test 校验。
3. Sitemap：补上新的规范路径（或确认它已被现有的 slug 遍历路由覆盖）；确认没有误加入非规范/会重定向的 URL。
4. 如果是替换/下线旧路由：走第 4 节完整的多处联动重定向清单——不要改完主路由配置就停下。
5. 正文：按第 7 节的标准写真实、具体、有实质信息量的内容——不是骨架文案，也不是把导航结构复制粘贴当正文。
6. 用生产构建 + 真实服务器验证：`curl` 每一个新旧路径，确认真实 HTTP 状态码——构建通过不等于测试通过。
7. 如果站点希望被 AI 问答引擎看到，确认新路径没有被现有某条针对其他爬虫的宽泛 `robots.txt` 禁止规则意外覆盖。
