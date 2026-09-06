---
name: adsense-review
description: Audit a website against Google AdSense/Publisher Policies (content, behavior, privacy, technical requirements), item by item. Use when asked to do an AdSense compliance review, investigate an ad policy violation notice (e.g. an ADS-CONTENT-01/ADS-SITE/ADS-BEHAVIOR notice), or before enabling ads on a new page.
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

**A common blind spot**: it's easy to audit only "does the privacy policy page mention Google/AdSense/children" and stop there, missing the CMP / EU User Consent Policy category entirely. These are two separate things: privacy-policy *text disclosure* is only one Publisher Policy requirement; a deployed *consent management platform* is a separate, independently-enforced requirement under the EU User Consent Policy. Text alone does not satisfy it. Walk through all four categories below every time — don't just re-check whatever was flagged last time.

Go through every item below, not a subset. When you find a violation, report which specific policy it breaks + evidence (file/URL/screenshot).

## 1. Content Policies
Check every page for: illegal content / IP infringement (piracy, counterfeit goods) / hate & discrimination / harassment & bullying / self-harm, violence, threats / extortion & exploitation / animal cruelty or endangered-species products / misrepresentative content (false affiliation, concealed identity) / unreliable & harmful claims (elections, health, climate) / phishing & deceptive engagement / manipulated media / hacking & surveillance promotion / sexually explicit content / compensated sexual services / mail-order brides / child sexual abuse & exploitation (CSAE).
- For a content/blog-style site, focus especially on: IP attribution (are quotes/images clearly sourced?) and misrepresentative claims (are performance/marketing numbers in copy verifiable and sourced?).

## 2. Behavioral Policies — the easiest category to accidentally violate
- **Ad interference**: does an ad overlap or sit adjacent to navigation/buttons in a way that causes accidental clicks? Does it obscure content? Is it placed on a "no-escape" screen (loading page, error page)?
- **Ad valuable inventory** (the most common real-world violation category):
  - Low-value screens: blank pages, "under construction" pages, pure navigation pages shouldn't carry ads.
  - Out-of-context placement: no ads in background/hidden iframes or inactive tabs.
  - **Duplicate content**: the same copy/body text appearing 2+ times in the rendered HTML (a common cause: writing one variant per responsive breakpoint, or an un-converted CMS export leaving a duplicate fragment). See "How to check this on your own site" below — this is one of the most common real-world causes of an ad-value flag.
  - **Ad-to-content ratio**: does ad area/count exceed the actual content on a given screen?
  - Language: is the page's primary content in an AdSense-supported language?
- **Deceptive requests**: does the metadata your ad request sends (page language, content category) match what's actually on the page?

## 3. Privacy Policies
- **CMP / EU User Consent Policy** (mandatory for EEA/UK since 2024-01-16, Switzerland since 2024-07-31 — the category most often missed):
  - Is a **certified CMP integrated with the IAB Europe TCF** actually **deployed** (Google's own CMP is now folded into the AdSense dashboard's "Privacy & messaging" tab; third-party options include Cookiebot, OneTrust, Quantcast Choice)? Privacy-policy text alone, with no real consent prompt or consent-signal delivery, is non-compliant.
  - The consequence of non-compliance isn't "ads stop entirely" — it's that traffic from that region is restricted to **non-personalized/limited ads**, which lowers revenue. Google's automated review can still flag "no CMP detected" as an open issue even though ads keep running.
  - Actually turning on a certified CMP (generating the message, getting the real script parameters) is an **AdSense-account-level action**, not something pure code can complete — code should only prepare a clean insertion point. Never fabricate a CMP script's contents (the parameters are account-specific; a fabricated script is a fake consent prompt, worse than having none).
  - How a CMP actually behaves (to avoid misreading it as "every visitor must click through a modal every time"):
    - **Triggered by visitor region**: a CMP typically checks the visitor's location (usually via IP) and only shows the consent prompt to EEA/UK/Switzerland visitors; everyone else proceeds straight to the normal personalized-ads flow, no prompt shown.
    - **Asked once, cached locally**: the choice is written to a cookie/localStorage as a TC String (the IAB TCF's standard encoding); the same browser won't be re-prompted unless consent expires or the message content changes.
    - **A rejection (or no answer yet) is not a broken site**: the only consequence is that visit gets non-personalized ads; every other page function and essential cookie keeps working normally. What actually triggers a policy violation is *having no consent mechanism at all* — not a visitor choosing to decline.
    - **This is industry-standard, not an AdSense-specific invention** — it's the same "we use cookies, accept/manage preferences" banner seen across the web. Cookiebot/OneTrust/Quantcast Choice/Google's own Privacy & messaging are different implementations of the same mechanism; the difference is only whether each is "Google-certified + IAB TCF-integrated."
    - **Automated scanners most likely check for `window.__tcfapi`**: the IAB TCF spec requires a certified CMP to expose a global `__tcfapi()` function that ad code reads the consent signal from. Google's automated crawl is very likely checking for the presence of that function, not parsing privacy-policy prose — which is exactly why "the privacy policy mentions AdSense uses cookies" has zero effect on this specific check.
- **Child-directed content declaration** (also commonly missed):
  - The privacy policy needs a clear, standalone "this site is not directed at children under 13" statement — not a passing mention buried in another sentence.
  - **Account-level setting**: AdSense/Ad Manager's dashboard content settings have a separate child-directed toggle that only the account holder can confirm/check — code cannot inspect or change it. Report explicitly that this needs manual account-holder confirmation; don't skip it just because code can't see it.
  - If the site might attract minors, ads must have interest-based targeting disabled — non-personalized ads only.
- Does the privacy policy page disclose: data collection/sharing/use arising from Google products, and third-party cookie usage?
- Does the cookie policy page explain cookie types (essential/analytics/advertising) and third parties involved (including AdSense)?
- Personalized-ad targeting restrictions: is any sensitive-category content (medical, negative financial info, race/religion, criminal history, sexual orientation, etc.) being used as a targeting signal? **US/Canada** have additional restrictions — housing, employment, and credit ads cannot be targeted by gender, age, parental status, marital status, or zip code.
- **AdChoices icon**: the "Ad choices" marker Google's ad code adds to personalized ads is automatic — never hide or clip it with custom CSS/overlays.
- Never pass personally identifiable information (PII) to Google; never set/read/tamper with cookies on the google.com domain.
- If collecting device location data: requires prior explicit consent + privacy-policy disclosure + encrypted transport.

## 4. Requirements & Standards
- Does `public/ads.txt` exist, is it correctly formatted, and does it list Google as an authorized seller (format: `google.com, pub-XXXXXXXXXXXXXXXX, DIRECT, f08c47fec0942fa0`)? Confirm this line wasn't accidentally removed or altered before making other changes to the file.
- Does the site meet Coalition for Better Ads standards (no pop-ups, no autoplay audio/video ads, no oversized interstitials, etc.)?
- Any malware/spyware indicators (unlikely, but worth a glance when changing third-party scripts/dependencies)?
- Does the site comply with Google's web spam policies and avoid "misleading experiences" (concealed real purpose, impersonating system prompts, etc.)?
- **Sanctions-region compliance**: no ad operations targeting Crimea, Cuba, Donetsk/Luhansk, Iran, or North Korea; sanctioned entities/individuals must not use the service.
- When touching the ad-loading script: confirm no artificial impression/click-inflation mechanism was introduced, and that nothing beyond the official `adsbygoogle.js` loading pattern was changed.

## How to check this on your own site
- **Ad integration points**: find the `google-adsense-account` meta tag and the actual ad-rendering component. Confirm each ad slot renders exactly once per page (check any change didn't turn it into a duplicate render).
- **Duplicate-content check**: build for production and run a real server (e.g. `next build && next start`, or your framework's equivalent) — don't eyeball a dev-mode page. Fetch the rendered HTML for real and count how many times a distinctive string of body copy appears; visual inspection misses content that's duplicated across responsive breakpoints or hidden via CSS. Use real HTTP status codes, not "the page looked fine."
- **Client-mounted ad units**: if your ad component is client-only (common for React/Next.js SPA-style ad slots), the ad DOM (e.g. Google's `ins.adsbygoogle`) only appears **after hydration** — checking raw server-rendered HTML will always show zero ad elements regardless of whether it's actually working. Verify in a real browser after the page finishes loading, not by grepping SSR output.
- **CMP and child-directed flag**: if these account-level settings can't be inspected from code, say so explicitly — report "needs account-holder confirmation in the dashboard," never treat "code can't see it" as "therefore compliant."

## Optional: a stopgap without a CMP vendor account
Deploying a certified CMP requires opening an account with a vendor (Cookiebot/OneTrust/etc.) or configuring Google's own Privacy & messaging tab — both require the account holder's own login and can't be done as a pure code change. If you want a safer default in the meantime (not a substitute for the real requirement, just damage control), two patterns are worth knowing:

1. **Global conservative default**: before every `adsbygoogle.push({})` call, set `window.adsbygoogle.requestNonPersonalizedAds = 1` (documented at [support.google.com/adsense/answer/9042142](https://support.google.com/adsense/answer/9042142)). This forces non-personalized ads everywhere until a real CMP is wired up — lower revenue, but never serves personalized ads without consent.
2. **Region-gated, self-built consent prompt** (still not a certified CMP — does not unlock personalized ads for gated regions on its own, and should be clearly labeled as such in code comments so a future reviewer doesn't mistake it for compliance): client-side, check the visitor's country via a keyless IP-geolocation lookup, cache the result locally with a TTL, and only show a simple accept/reject prompt to visitors in EEA/UK/Switzerland; store their choice locally and feed it into the `requestNonPersonalizedAds` decision above. Give users a way to revisit/clear that choice later (e.g. a button on the cookie-policy page). Watch out for a real UI bug this pattern invites: a banner fixed to the top of the viewport can end up sitting on top of (and hiding) a fixed site header at the same z-index — a bottom-fixed banner avoids this and also matches where most real-world consent banners (including Google's own Funding Choices UI) are placed.
