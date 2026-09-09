---
name: adsense-review
description: Audit a website against Google AdSense/Publisher Policies (eligibility, ownership, content depth & originality, trust pages, behavior, privacy, technical requirements), item by item. Use when asked to do an AdSense compliance review, investigate an ad policy violation notice (e.g. an ADS-CONTENT-01/ADS-SITE/ADS-BEHAVIOR notice), or before enabling ads on a new page.
---

# AdSense Publisher Policy Review

*[中文版本 / Chinese version →](../adsense-review-zh/SKILL.md)*

Authoritative sources (policy text changes — always re-check these pages, don't rely on memory):
[Program policies](https://support.google.com/adsense/answer/48182) ·
[Behavioral policies](https://support.google.com/adsense/answer/2753860) ·
[Publisher Policies detailed breakdown](https://support.google.com/adsense/answer/10502938) ·
[Getting started guide](https://support.google.com/adsense/answer/23921) ·
[EU User Consent Policy](https://support.google.com/adsense/answer/7670013) ·
[EEA/UK/Switzerland consent requirements](https://support.google.com/adsense/answer/13554116)

**Three common blind spots** (the reasons a review usually comes back "not yet ready"):
1. **CMP vs. privacy text**: it's easy to audit only "does the privacy policy page mention Google/AdSense/children" and stop there, missing the CMP / EU User Consent Policy category entirely. These are two separate things: privacy-policy *text disclosure* is only one Publisher Policy requirement; a deployed *consent management platform* is a separate, independently-enforced requirement under the EU User Consent Policy. Text alone does not satisfy it.
2. **Thin / low-value content** (ADS-CONTENT-03): the single most common real-world rejection reason. A site can be 100% policy-clean (no piracy, no hate, no adult content) and still be rejected because pages are too shallow, too templated, or too few. Word count alone isn't the bar — a page that's all "upload → click → download" boilerplate with no distinct, useful information is a thin page even at 400 words. Always measure and call out content depth, not just content legality.
3. **Missing trust pages** (ADS-UX-05): About, Contact, and Privacy (plus Terms where relevant) are the first thing a reviewer checks for credibility. "Privacy policy exists" is not enough if there is no About page, no working contact channel, and no footer link to any of them.

## Output format
Report in severity order — **Blocker → High → Medium** — then a full checklist, then a "needs account-holder confirmation" list. For each finding, cite the specific requirement ID + evidence (file/URL/screenshot). Use these IDs so results can be cross-checked against policy:

- **ADS-ELIG** — eligibility; **ADS-OWN** — ownership; **ADS-SITE** — site list; **ADS-TXT** — ads.txt; **ADS-CONTENT** — content; **ADS-UX** — trust/UX; **ADS-CRAWL** — crawlability; **ADS-PROG** — program/behavior; **ADS-PUB** — publisher; **ADS-REST** — restricted content; **ADS-PRIV** — privacy.
- Mark anything that only the account holder can verify as **Unknown / self-check** — never treat "code can't see it" as "therefore compliant" or "therefore N/A".

## Checklist (go through every item, not a subset)

### A. Eligibility & ownership (mostly account-level self-check)
- **ADS-ELIG-01** — applicant is 18+ (or has a guardian-managed account). Tool cannot verify; ask the owner.
- **ADS-ELIG-02** — no duplicate AdSense account. New sites belong on an *existing* account; a second account gets banned. Self-check.
- **ADS-ELIG-03** — content overall meets program + publisher policies (summary gate — pass only if the items below are clean).
- **ADS-ELIG-04** — hosted platforms (Blogger/YouTube) use the hosted-account flow; N/A for self-hosted domains.
- **ADS-OWN-01** — the owner can inject code into `<head>` (source/CMS access; on a builder, confirm custom-code support). Self-check.
- **ADS-OWN-02** — domain is owned and verifiable (DNS under the owner's control). Don't apply with someone else's site. Self-check.
- **ADS-OWN-03** — pages render JS correctly with complete HTML structure (fetch the real HTML).

### B. Site & technical
- **ADS-SITE-01** — site has been added to the AdSense "Sites" list and approved. Dashboard-only action. Self-check.
- **ADS-SITE-02** — at least one ownership-verification method is deployable (code snippet / ads.txt / meta). Self-check.
- **ADS-TXT-01** — if `ads.txt` exists, it contains the Google authorization line.
- **ADS-TXT-02** — `ads.txt` is published (recommended to prevent inventory spoofing).
- **ADS-CRAWL-01** — site is online, key pages return real HTTP 200 (no 404/5xx on home or sampled pages).
- **ADS-CRAWL-02** — `robots.txt` does not block Google/AdSense crawlers; no login wall on content pages.
- **ADS-CRAWL-03** — pages reachable via GET (no POST-only).
- **ADS-CRAWL-04** — no fragile/excessive redirect chains.
- **ADS-CRAWL-05** — stable, clean URLs (no session IDs / one-off params).
- **ADS-CRAWL-06** — DNS/TLS/host response stable; HTTPS works.
- **ADS-CRAWL-07** — sitemap present with a stable crawl path.

### C. Content — originality AND depth
- **ADS-CONTENT-01** — original, useful, valuable content (no scraped / AI-rewritten / placeholder text).
- **ADS-CONTENT-02** — not purely reposted/embedded/affiliate-feed; has original viewpoint or data.
- **ADS-CONTENT-03** — **sufficient depth, no large number of thin pages.** This is a top rejection reason. Sampled pages should carry genuinely distinct, useful information — not the same "How to use" skeleton reused across dozens of pages. Guidance below under "Content depth remediation."
- **ADS-CONTENT-04** — not "under construction" / an empty shell (check for placeholder dates, "coming soon", lorem ipsum, 404 links from nav).
- **ADS-CONTENT-05** — ads/affiliate/promotional content does not exceed the site's own content.
- **ADS-CONTENT-06** — primary content language is AdSense-supported.
- **ADS-CONTENT-07** — comments/UGC are moderated; no spam/abusive content.
- **ADS-CONTENT-08** — no keyword stuffing or doorway pages.
- **Credibility signals** (part of CONTENT-04/05): no future/placeholder dates on posts, no visibly half-finished pages, no "About not found", no concealed identity.

### D. Trust & UX
- **ADS-UX-01** — clear, usable navigation.
- **ADS-UX-02** — visitor can tell what the site does and find content.
- **ADS-UX-03** — no deceptive navigation: fake download buttons, irrelevant redirects, links to non-existent content.
- **ADS-UX-04** — no malicious behavior: forced redirects, auto-downloads, pop-under/up, tampering with user settings.
- **ADS-UX-05** — **trust pages exist and are linked in the footer**: About, Contact, Privacy (and Terms where relevant). A site with a privacy policy but no About page or contact channel is a credibility flag, not a pass.
- **ADS-UX-06** — no ad-dominant/confusing layout (ads first, content buried).

### E. Behavior / program
- **ADS-PROG-01** — no self-clicking, no impression/click inflation. Behavior red line; self-check.
- **ADS-PROG-02** — no encouraging clicks ("click to support us", arrows toward ads).
- **ADS-PROG-03** — ads clearly separated from content, neutral labels (e.g. "Ad" / "Sponsored").
- **ADS-PROG-04** — legitimate traffic sources (no PTC, click-exchange, spam email/comment traffic). Self-check.
- **ADS-PROG-05** — no modified ad code to inflate performance; paste the official snippet as-is.
- **ADS-PROG-06** — ads only on normal content pages (not in software, popups, email, or empty pages). Self-check.
- **ADS-PROG-07** — WebView monetization requirements (N/A for ordinary sites).
- **ADS-PUB-01** — no illegal content or facilitation of illegal activity.
- **ADS-PUB-02** — no infringement / piracy / counterfeit goods.
- **ADS-PUB-03** — no hate, harassment, self-harm incitement, or glorification of violence.
- **ADS-PUB-04** — no animal cruelty or endangered-species products.
- **ADS-PUB-05** — no concealing/misrepresenting site identity, purpose, or affiliation.
- **ADS-PUB-06** — no phishing / harvesting personal info / fake get-rich promises.
- **ADS-PUB-07** — no document forgery / exam-taking / cracking / spyware (the "dishonesty" edge). See the security-tool note below.
- **ADS-PUB-08** — no sexual services / mail-order brides / adult themes slipped into family-oriented content.
- **ADS-PUB-09** — publisher info and ad-request data are truthful (ads.txt consistent with identity).
- **ADS-PUB-10** — ads don't obscure content/navigation; no dead-end screen that forces a click.
- **ADS-PUB-11** — no ads on no-content / low-value / construction / pure-repost pages.
- **ADS-PUB-12** — ads don't appear out of context (back-end pages, off-screen).
- **ADS-PUB-13** — no election-disrupting or anti-consensus health/climate false claims.
- **ADS-PUB-14** — no synthetic/manipulated media misrepresenting public-issue topics.
- **ADS-PUB-15** — nothing harmful to children (veto-level).
- **ADS-PUB-16** — sensitive events (disasters/crises) not exploited or mocked.

### F. Restricted content (each needs an explicit pass, not an assumption)
- **ADS-REST-01** — sexually explicit content / adult products / adult-oriented supplements.
- **ADS-REST-02** — gore / profanity as a significant element.
- **ADS-REST-03** — firearms and manufacturing/modification tutorials.
- **ADS-REST-04** — tobacco / recreational drugs / usage tutorials.
- **ADS-REST-05** — online alcohol sales or encouraging abuse.
- **ADS-REST-06** — online gambling / pay-to-play games of chance.
- **ADS-REST-07** — prescription drug sales / online pharmacies / unapproved drugs & supplements.
- **ADS-REST-08** — ad-overlay implementations (sticky/floating-video violations).

### G. Privacy
- **ADS-PRIV-01** — privacy policy published, disclosing cookie/data collection.
- **ADS-PRIV-02** — privacy policy discloses third-party (advertiser) cookie/identifier use.
- **ADS-PRIV-03** — no PII passed in ad requests (URL params, data layer must not carry email/phone/etc.). Self-check when wiring the ad code.
- **ADS-PRIV-04** — EEA/UK/Switzerland traffic needs a certified CMP / consent banner (see the CMP deep-dive below).
- **ADS-PRIV-05** — precise location collection requires prior notice + consent; N/A if the site doesn't request location.
- **ADS-PRIV-06** — child-directed content must be marked and interest-based targeting disabled (account-level toggle + privacy-policy statement).
- **ADS-PRIV-07** — no setting/tampering cookies on google.com domains (only a custom ad proxy/injection could trigger this).
- **ADS-PRIV-08** — no sensitive info (health/religion/sexual orientation/etc.) used for targeting or audience-building.
- **ADS-PRIV-09** — US/Canada: housing/employment/credit ads must not be targeted by demographic attributes.
- **ADS-PRIV-10** — personalized ads require data rights + interest-based-ads (IBA) disclosure in the privacy policy.

## Content depth remediation (Blocker-level for tool/feature sites)
A site of many structurally-identical "tool" or "generator" pages is the classic low-value/ doorway pattern. Fix it before applying:
- **Distinguish every page**: rewrite each page as an independent, information-rich guide, not a shared template. Cover use cases, processing logic, parameter meanings, limitations, privacy handling, and a real example — differently per page.
- **Length target**: 800–1500 words per page for content-bearing pages, with structured subheadings. A 242-word `/help` or 460-word tool page won't clear the depth bar.
- **De-template the "How to use" block**: never reuse the same steps verbatim across pages.
- **Grow the content asset**: thin tool sites should add real articles (guides, "how it works under the hood", troubleshooting) and interlink them; a sitemap that's 95% tool URLs reads as a doorway set.
- **Add trust/transparency content**: an "about" page with a real operator/history, plus a "how we handle your files / data lifecycle" page — this is the single highest-value trust signal for a tool site.

## Security-type tools — the "dishonesty" edge (ADS-PUB-07)
Tools like *remove password*, *remove certificate signature*, *sanitize*, *unlock* have legitimate uses but sit right on the policy line. They read as "facilitating circumvention" unless each page states the legitimate-use boundary explicitly. Add a visible disclaimer on those pages, e.g.:
> "This tool is intended for files you own or are authorized to modify (e.g. documents you created, or files whose owner gave you permission). Do not use it to bypass protections you don't have the right to remove."

For tool sites this materially lowers both rejection risk and misuse risk.

## The CMP / EU User Consent Policy deep-dive
- **Is a certified CMP integrated with the IAB Europe TCF actually deployed** (Google's own CMP is now folded into the AdSense dashboard's "Privacy & messaging" tab; third-party options include Cookiebot, OneTrust, Quantcast Choice)? Privacy-policy text alone, with no real consent prompt or consent-signal delivery, is non-compliant.
- The consequence of non-compliance isn't "ads stop entirely" — it's that traffic from that region is restricted to **non-personalized/limited ads**, which lowers revenue. Google's automated review can still flag "no CMP detected" as an open issue even though ads keep running.
- Actually turning on a certified CMP (generating the message, getting the real script parameters) is an **AdSense-account-level action**, not something pure code can complete — code should only prepare a clean insertion point. Never fabricate a CMP script's contents (the parameters are account-specific; a fabricated script is a fake consent prompt, worse than having none).
- How a CMP actually behaves (to avoid misreading it as "every visitor must click through a modal every time"):
  - **Triggered by visitor region**: a CMP typically checks the visitor's location (usually via IP) and only shows the consent prompt to EEA/UK/Switzerland visitors; everyone else proceeds straight to the normal personalized-ads flow, no prompt shown.
  - **Asked once, cached locally**: the choice is written to a cookie/localStorage as a TC String (the IAB TCF's standard encoding); the same browser won't be re-prompted unless consent expires or the message content changes.
  - **A rejection (or no answer yet) is not a broken site**: the only consequence is that visit gets non-personalized ads; every other page function and essential cookie keeps working normally. What actually triggers a policy violation is *having no consent mechanism at all* — not a visitor choosing to decline.
  - **This is industry-standard, not an AdSense-specific invention** — the same "we use cookies, accept/manage preferences" banner seen across the web.
  - **Automated scanners most likely check for `window.__tcfapi`**: the IAB TCF spec requires a certified CMP to expose a global `__tcfapi()` function that ad code reads the consent signal from. Google's automated crawl is very likely checking for the presence of that function, not parsing privacy-policy prose — which is exactly why "the privacy policy mentions AdSense uses cookies" has zero effect on this specific check.
- **Child-directed content declaration** (also commonly missed): the privacy policy needs a clear, standalone "this site is not directed at children under 13" statement; the AdSense dashboard has a separate child-directed toggle that only the account holder can confirm. Report it as needing account-holder confirmation.

## How to check this on your own site
- **Ad integration points**: find the `google-adsense-account` meta tag and the actual ad-rendering component. Confirm each ad slot renders exactly once per page (check any change didn't turn it into a duplicate render).
- **Content-depth check**: crawl the sitemap, sample several inner pages, and measure the *distinct* body word count (strip the shared header/footer/nav/template). Median below ~500 words, or multiple pages below 250, or a shared "How to use" skeleton across pages, is a thin-content flag — report it as High/Blocker even if nothing is policy-illegal.
- **Duplicate-content check**: build for production and run a real server (e.g. `next build && next start`, or your framework's equivalent) — don't eyeball a dev-mode page. Fetch the rendered HTML for real and count how many times a distinctive string of body copy appears; visual inspection misses content duplicated across responsive breakpoints or hidden via CSS. Use real HTTP status codes, not "the page looked fine."
- **Client-mounted ad units**: if your ad component is client-only (common for React/Next.js SPA-style ad slots), the ad DOM (e.g. Google's `ins.adsbygoogle`) only appears **after hydration** — checking raw server-rendered HTML will always show zero ad elements regardless of whether it's actually working. Verify in a real browser after the page finishes loading, not by grepping SSR output.
- **Trust pages**: check the footer of the home page for working About / Contact / Privacy / Terms links; a link is not enough if the target is a stub or 404s.
- **CMP and child-directed flag**: if these account-level settings can't be inspected from code, say so explicitly — report "needs account-holder confirmation in the dashboard," never treat "code can't see it" as "therefore compliant."

## Self-check items the tool cannot verify (report as a list for the owner)
These are exactly the "Unknown" items to hand back to the site owner — confirm each, don't mark them pass/fail:
- ADS-ELIG-01/02 — age, no duplicate account.
- ADS-OWN-01/02 — `<head>` injection access, own the domain.
- ADS-SITE-01/02 — site added in the dashboard, ownership verification deployable.
- ADS-PROG-01 — never click own ads, never inflate impressions/clicks.
- ADS-PROG-04 — traffic sources are legitimate (no PTC/exchange/spam).
- ADS-PROG-06 — ads only on content pages.
- ADS-PRIV-03 — no PII in ad requests.
- ADS-PRIV-05 — location collection only with consent (or N/A).
- ADS-PRIV-06 — child-directed toggle confirmed in dashboard.
- ADS-PRIV-08/09/10 — no sensitive-info targeting; US/CA housing/employment/credit rules; IBA disclosure.

## Optional: a stopgap without a CMP vendor account
Deploying a certified CMP requires opening an account with a vendor (Cookiebot/OneTrust/etc.) or configuring Google's own Privacy & messaging tab — both require the account holder's own login and can't be done as a pure code change. If you want a safer default in the meantime (not a substitute for the real requirement, just damage control), two patterns are worth knowing:

1. **Global conservative default**: before every `adsbygoogle.push({})` call, set `window.adsbygoogle.requestNonPersonalizedAds = 1` (documented at [support.google.com/adsense/answer/9042142](https://support.google.com/adsense/answer/9042142)). This forces non-personalized ads everywhere until a real CMP is wired up — lower revenue, but never serves personalized ads without consent.
2. **Region-gated, self-built consent prompt** (still not a certified CMP — does not unlock personalized ads for gated regions on its own, and should be clearly labeled as such in code comments so a future reviewer doesn't mistake it for compliance): client-side, check the visitor's country via a keyless IP-geolocation lookup, cache the result locally with a TTL, and only show a simple accept/reject prompt to visitors in EEA/UK/Switzerland; store their choice locally and feed it into the `requestNonPersonalizedAds` decision above. Give users a way to revisit/clear that choice later (e.g. a button on the cookie-policy page). Watch out for a real UI bug this pattern invites: a banner fixed to the top of the viewport can end up sitting on top of (and hiding) a fixed site header at the same z-index — a bottom-fixed banner avoids this and also matches where most real-world consent banners (including Google's own Funding Choices UI) are placed.
