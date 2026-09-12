# Web Scraping Notes

Notes distilled from a `/scrp/` (Web Scraping General, 4chan `/g/`-style)
thread — libraries/frameworks, AI-assisted extraction, commercial APIs,
proxies, CAPTCHA services, and the legal landscape around scraping.

See [LINKS.md](LINKS.md) for the full raw link list.

## Not legal advice — read this first

Web scraping itself is legal in broad strokes (public data, no login, per
*hiQ Labs v. LinkedIn* in the US), but the specifics matter a lot and vary
by jurisdiction:

- **US**: scraping public data without logging in generally doesn't violate
  the CFAA — but continuing *after* a cease-and-desist or IP block creates
  real legal exposure, and breaching a clickwrap Terms of Service is
  separately actionable as a contract claim even when no criminal statute
  applies.
- **EU**: GDPR applies to scraping *personal* data regardless of whether it
  was public, and applies extraterritorially — scraping EU residents'
  data triggers compliance obligations no matter where you are. Facial
  recognition scraping specifically has drawn €20–30M+ fines.
- **UK**: UK GDPR + Computer Misuse Act 1990, with less scraping-specific
  case law than the US.
- **Australia**: Privacy Act 1988 + Copyright Act 1968, EU-style rules for
  biometric data.
- **`robots.txt` and ToS**: neither has independent legal force — `robots.txt`
  is a voluntary convention, browsewrap ToS is weak, but a clickwrap
  agreement (you had to actively click "I agree") can form a real contract.
  Courts tend to treat ToS violations as evidence of bad faith rather than a
  standalone violation on their own.
- **The factors that actually escalate risk**: scraping behind a login,
  scraping personal/biometric data, hammering a server hard enough to
  degrade it, and continuing after you've been told to stop (C&D letter or
  active IP block). Avoid all four and you're in comparatively safe
  territory; do any of them and you're in real legal gray area regardless
  of jurisdiction.

## Picking a tool — the thread's own rule of thumb

- **Static site, no JS** → `requests`/`httpx` (Python) + `lxml`/BeautifulSoup
  for parsing. Cheapest and fastest option when it applies.
- **JS-heavy or anti-bot-protected** → Playwright/Puppeteer with a stealth
  patch, plus proxies.
- **Large structured crawl** → Scrapy (Python, the mature standard) or
  Crawlee (built-in queues/proxy rotation).
- **The single most valuable habit**: before reaching for full browser
  automation, open the Network tab and see if the page is just calling a
  JSON API under the hood — hitting that API directly with `requests`/
  `httpx` is an order of magnitude cheaper and faster than driving a real
  browser, and browser automation should be treated as a prototyping tool,
  not a production-scale solution.

## Core tool landscape

- **HTTP clients**: `requests` (Python sync standard), `httpx` (async,
  HTTP/2), `curl_cffi` (specifically impersonates real browser TLS
  fingerprints — matters because sites fingerprint your TLS handshake, not
  just your User-Agent header), `axios` (Node).
- **Parsers**: BeautifulSoup (most beginner-friendly), `lxml` (fast, C-based),
  `parsel` (what Scrapy uses internally), `selectolax` (fastest, built on
  the Lexbor engine), Cheerio (Node, jQuery-like API).
- **Full frameworks**: Scrapy (Python, the production standard), Crawlee
  (queues/proxy rotation built in), Colly (Go), StormCrawler (Java,
  distributed via Apache Storm — for genuinely large-scale crawls).
- **Browser automation**: Playwright (multi-browser, the modern default),
  Puppeteer (Chrome/Chromium via CDP), Selenium (oldest, widest
  compatibility), nodriver (built specifically to be hard to fingerprint).
- **Anti-detection patches**: Patchright (patched Playwright), SeleniumBase
  UC Mode (built for Cloudflare/CAPTCHA-heavy sites), stealth plugins for
  Puppeteer/Playwright, Camoufox (an anti-fingerprint Firefox fork).

## New/emerging (2024–2026) — where the field is actually moving

Splitting into two distinct directions:

1. **AI-driven browser automation** — the model itself drives the browser:
   **browser-use** (LLM drives Playwright directly), **Stagehand** (mixes
   deterministic code with AI, caches selectors so it doesn't re-reason
   every run), **Skyvern** (uses a vision model reading screenshots instead
   of parsing the DOM — notable since it doesn't care about DOM structure
   at all), **AgentQL** (semantic queries instead of XPath/CSS selectors),
   **Maxun** (no-code, record-your-clicks, self-hosted).
2. **LLM-friendly output formatting** — classic scraping, but the output is
   pre-cleaned for AI consumption: **Firecrawl** and **Crawl4AI** (convert
   pages to clean Markdown/JSON for RAG pipelines), **ScrapeGraphAI**
   (returns typed JSON with no selector-writing required), **Scrapling**
   (adaptive — tries to keep working when a site's HTML changes shape).

Plain Scrapy and Playwright remain the foundation underneath most of this —
the new tools are mostly AI-assisted layers on top, not replacements.

## MCP servers (for Claude/agent use specifically)

- **Fetch MCP** (Anthropic's own) — retrieves a URL, returns cleaned
  markdown. The simplest, lowest-capability option.
- **Playwright MCP** (Microsoft) and **Chrome DevTools MCP** — real browser
  control (click, fill forms, screenshot), the latter with devtools access
  too.
- **Browserbase MCP** — cloud headless browser + Stagehand integration.
- **mcp-chrome** — connects to your *own already-logged-in* Chrome session,
  which sidesteps a lot of anti-bot/login friction entirely by using your
  real authenticated session rather than a fresh automated one.
- **Scrapfly MCP** — managed anti-bot scraping as a paid service, MCP-wrapped.
- Caveat from the source: **Puppeteer MCP** (the original reference
  implementation) is archived/unmaintained — check commit dates before
  adopting any MCP server in this space, several move fast or die fast.

## Commercial platforms, proxies, and CAPTCHA services

- **Commercial scraping APIs**: Bright Data (biggest, full proxy+scraping
  suite), Oxylabs/Decodo (proxy-focused), ScraperAPI/Scrapfly/ScrapingBee
  (anti-bot + proxy + rendering bundled), Apify (pre-built "Actors" on a
  pay-per-use model), SerpApi (search-results-specific), Diffbot (AI/CV-based
  structured extraction), Zyte (Scrapy hosting + AI extraction), Browserbase
  (remote Chromium for agent use). **Caveat from the source**: pricing shifts
  often, impersonation scams exist in this space, and services can shut
  down or mishandle data unexpectedly — don't treat any vendor as
  permanent infrastructure.
- **Proxy providers, community-ranked** (deliberately excluding the big
  enterprise players above): budget tier (Webshare, PacketStream — cheap
  but lower ban-avoidance rates), mid-tier "sweet spot" (IPRoyal, Soax,
  MarsProxies, Evomi), premium/static-ISP tier (NetNut, Rayobyte).
  **Explicit warning from the source**: most online proxy "comparisons" are
  affiliate spam, Evomi/Infatica have a reputation for hidden fees, and the
  single best piece of advice is to buy the smallest trial plan before
  committing to anything larger.
- **CAPTCHA-solving services**: 2Captcha, Anti-Captcha, CapSolver,
  CapMonster Cloud, DeathByCaptcha, NopeCHA — spanning human-solver and
  AI-solver models, ~$0.30–$3 per 1,000 solves, covering
  reCAPTCHA/hCaptcha/Turnstile. **The thread's own strongest recommendation
  is to avoid needing these at all**: correct TLS/JA3 fingerprinting
  (`curl_cffi`), quality residential proxies, a stealth browser
  (Patchright/Camoufox/nodriver), and human-like rate limiting reportedly
  avoid 80–90% of CAPTCHA challenges before a solver is ever needed —
  cheaper, faster, and lower legal/ethical exposure than paying to defeat a
  site's anti-abuse system outright.
