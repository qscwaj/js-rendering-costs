# web scraping javascript websites: When You Actually Need a Headless Browser, What Rendering Really Costs, and How to Pick a Plan That Fits

Most JavaScript-heavy sites don't need a headless browser. The reason your scraper comes back empty on React, Vue, or Next.js pages is that the data lives in an XHR call or an embedded JSON blob — and in a large share of cases you can pull it with a plain HTTP request once you know where to look. For the rest, a scraping API with built-in rendering like ScraperAPI handles the browser work for you, at a price you can calculate in advance instead of paying for a server that runs Chromium all day.

This guide covers why `fetch` + Cheerio fails on client-rendered sites, how to check whether you actually need rendering, what the realistic options cost, and — since rendering is billed by the credit — a full breakdown of ScraperAPI's plans and credit math so you don't burn your quota on pages you could have scraped for 1 credit.

> **Quick pick:** if you're scraping ordinary pages and occasionally need JavaScript rendered, the Hobby plan at $49/month (100,000 credits, 20 concurrent threads) covers roughly 9,000 rendered pages or 100,000 plain ones, and annual billing brings it to $44.10/month. Every plan includes JS rendering, premium proxies, and CAPTCHA handling. There's also a 7-day free trial with 5,000 credits, no credit card required. 👉 [See ScraperAPI plans and the free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Why your scraper returns nothing on JS websites

A client-side rendered (CSR) page is really three round trips: the HTML, the JavaScript bundle, and then the data. When you request the raw HTML with `axios`, `fetch`, or a basic HTTP client, you get the shell — an almost-empty `<div id="root"></div>` and a script tag — and none of the content your browser shows you.

Your browser looks complete because it runs the JavaScript, which fires off network requests, receives the data, and builds the DOM on your screen. A static HTTP client does none of that. So the parsed "page" contains a mount point, a title, and nothing else. That's the entire mystery behind "web scraping javascript websites" searches: people had a working parser, pointed it at a modern site, and got an empty result.

There's a distinction worth keeping in mind. Server-side rendered (SSR) pages deliver content in the initial HTML, so plain scraping works fine. Static site generators pre-render at build time and behave like SSR. Only true CSR pages hide the data behind a second network step. The test for any target site is one question: is the data you want present in the HTML a plain HTTP client receives?

## Check before you render: most targets don't need a browser

A headless browser is the most expensive and most detectable tool in your toolkit. Before reaching for one, run through a quick checklist that takes minutes:

1. **Open DevTools, press Ctrl+Shift+P, choose "Disable JavaScript," and reload.** If the content is still there, the site is server-rendered and plain HTTP works.
2. **Curl the URL and read the raw bytes.** Look for three things: data written directly into the HTML; an embedded JSON blob in a script tag (`__NEXT_DATA__` for Next.js, `__NUXT__` for Nuxt, `window.__INITIAL_STATE__`, or `application/ld+json` structured data — which on product pages often contains exactly the product data you want); or just a mount point and a bundle URL, meaning the data sits behind a network call.
3. **Open the Network tab, filter by Fetch/XHR, and reload.** The requests the page makes to fill itself are your data layer. If one of them is a plain GET returning JSON, you can call it directly — no rendering needed.

The embedded-JSON and internal-API routes are the cheapest by far. Grep for framework signatures and parse the JSON; find the XHR endpoint and reproduce it in your HTTP client (usually you just need a browser-like User-Agent and an `Accept: application/json` header, sometimes a Referer). One commonly cited rule of thumb among practitioners: roughly 70% of "JS-rendered" targets fall on these first two rungs with nothing more than `requests` and a DevTools window.

Only when the data genuinely requires executing page scripts — infinite feeds, canvas-heavy apps, strict bot walls — do you move to rendering.

## Your four options, and what each one costs

| Approach | How it works | Speed | Detection risk | Rough cost |
| --- | --- | --- | --- | --- |
| Plain HTTP + parser | Fetch HTML or embedded JSON | Fastest | Lowest | Effectively free |
| Internal XHR/API | Call the endpoint the page itself calls | Fast | Low–medium | Free |
| Rendering API | Send URL to a service that runs the browser for you | 1–2s/page | Medium | ~$1–5 per 1,000 pages |
| Self-hosted headless browser | Run Puppeteer/Playwright yourself | 3–8s/page | Highest | ~$2–8 per 1,000 pages in compute |

A rendering API sends the target URL with a parameter like `render=true`, and the provider spins up Chromium on their infrastructure, executes the scripts, and returns finished HTML. You skip installing Puppeteer, managing proxies, and solving CAPTCHAs — all of that is the provider's problem. Self-hosting gives you full control and no per-request pricing, but you pay in server resources (RAM and CPU per page) and you own the entire anti-bot arms race.

For most people scraping JS sites occasionally or at moderate volume, the rendering API route is the sane middle ground. Which brings us to what it actually costs.

## ScraperAPI plans: the full price list

ScraperAPI prices everything in API credits. A standard page request costs 1 credit; everything else depends on the target domain and the parameters you send. Here are all the plans currently listed on the official pricing page:

| Plan | Price (monthly / annual) | API credits | Concurrent threads | Buy |
| --- | --- | --- | --- | --- |
| Free | $0 (1,000 one-time credits) | 1,000 | 5 | [Start free](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | $49 / $44.10 | 100,000 | 20 | [Get Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 / $134.10 | 1,000,000 | 50 | [Get Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 / $269.10 | 3,000,000 | 100 | [Get Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling (most popular) | $475 / $427.50 | 5,000,000 | 200 | [Get Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 / $877.50 | 10,500,000 | 300 | [Get Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 / $1,777.50 | 21,500,000 | 500 | [Get Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | 22,000,000+ | 500+ | [Contact sales](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Every paid plan includes JS rendering, premium residential and mobile proxies, JSON auto-parsing, rotating proxy pools, CAPTCHA and anti-bot detection, custom session support, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. The differences between tiers are volume, concurrency, and geotargeting scope: Hobby and Startup can only geotarget US and EU, while Business and above unlock country-level geotargeting and unrestricted analytics history.

Before committing, you can test with the 7-day free trial (5,000 credits, no credit card), and there's a permanent free plan that grants 1,000 credits at signup with up to 5 concurrent connections. 👉 [Try ScraperAPI free for 7 days](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What JS rendering actually costs in credits

This is where a lot of articles go vague, and where your bill lives. Rendering is a parameter surcharge, not a separate plan:

- `render=true` adds **10 credits** per request. A rendered ordinary page costs 11 credits total.
- `premium=true + render=true` costs **25 credits** per request.
- `ultra_premium=true + render=true` costs **75 credits** per request (paid plans only) — this is the tier for pages behind the toughest protection.

On top of that, some domains carry their own multiplier regardless of rendering: Amazon, Walmart, and eBay cost 5 credits per request; Google and Bing (including all subdomains) cost 25; LinkedIn costs 30. Bypassing protection systems — Cloudflare, Cloudflare Turnstile, Datadome, PerimeterX, Akamai — adds another 10 credits per scrape. And several parameters are free: `wait_for_selector`, `country_code`, `session_number`, `device_type`, `output_format`, `keep_headers`, and `autoparse` don't touch your quota.

Run the numbers on Hobby. At 11 credits per rendered standard page, 100,000 credits buys about 9,000 rendered pages per month. Plain requests stretch the same quota to 100,000 pages. That gap is the entire argument for checking whether your target needs rendering at all.

Three tools keep this from getting out of hand:

- Every response includes a `sa-credit-cost` header showing exactly what that request consumed.
- The dashboard's Domain Multiplier and API Playground let you check a URL's cost and verify results before running anything at scale.
- There's also a cost-check API endpoint (`api.scraperapi.com/account/urlcost`) that accepts all parameters, so you can compute the exact price of `url + render=true` programmatically. You can additionally set a `max_cost` per scrape as a hard ceiling.

Official guidance is blunt about this: use JS rendering only when you absolutely need it, because rendering increases latency and can reduce success rates, which reduces the volume of requests you can process. One practical detail: if rendered pages still miss elements, add `wait_for_selector=x` — but it only works alongside `render=true`; the API silently ignores it otherwise.

## Billing quirks worth knowing before you sign up

A few rules live in the docs rather than on the pricing page, and each one changes how you'd use the service:

- **Failed requests are free.** ScraperAPI only charges for successful responses — status codes 200 and 404 — plus requests you cancel yourself before 70 seconds. Retries on failures don't drain your quota.
- **Rendered requests have a burst limit of 10 per second.** This caps how many renders you can *start* each second, not total concurrency: at 10 req/sec with 25-second renders you'd have up to 250 renders in flight. Higher render concurrency is Enterprise-only.
- **Exceeding thread limits returns a 429** with no charge — requests are rejected rather than queued.
- **When credits run out, scraping pauses by default.** On Hobby through Professional you can opt into auto-upgrade (auto-moves you to the next tier mid-cycle); on Scaling and above you can enable pay-as-you-go at a fixed per-credit rate with a monthly cap you set.
- Annual billing saves 10%, and annual plans can renew early for a fresh batch of credits.

## How ScraperAPI compares on JS rendering pricing

Two competitors worth naming, on information alone:

- **Zenrows** lists its Starter plan at $49/month for 5,000 credits with JS rendering bundled into the base request cost.
- **ScrapingBee** lists its Freelance plan at $49/month for 5,000 credits, with JS rendering consuming 5 credits per request.

On paper, at the $49 tier the competitors' credit pools look small next to ScraperAPI Hobby's 100,000 credits. The honest caveat: those plans' credits may already include proxy costs that ScraperAPI bills separately through its premium/ultra-premium surcharges, so a credit at each provider doesn't buy the same thing. Compare against your actual target sites — run them through a cost estimator before deciding whose credit is worth more for your workload. If your targets are mostly protected pages needing the roughest treatment, either way, test both with trials before committing to an annual plan.

## FAQ

**Does ScraperAPI's free plan include JavaScript rendering?**
Yes. JS rendering is included with all plans, and the free plan (1,000 credits at signup, 5 concurrent connections) can use it — just remember each rendered request consumes 11 credits on a standard page, so the free quota covers around 90 rendered pages.

**How many rendered pages do I get per month on Hobby?**
About 9,000 on standard domains (11 credits each), fewer on Amazon-type domains where the base request already costs 5 credits (16 per rendered page, roughly 6,250 pages). Plain requests on standard domains: 100,000.

**Do I need to install anything to scrape JS sites with ScraperAPI?**
No. You send an HTTP request with `render=true` and get back the fully rendered HTML. The Chromium instance runs on ScraperAPI's side. There's also a proxy-port mode where parameters are passed through the username (for example `scraperapi.render=true`) so you can use it with existing HTTP client setups.

**Can I click buttons or scroll during rendering?**
Yes. The Render Instruction Set supports clicking elements, typing input, scrolling, and waiting for specific events before the rendered response is returned — useful for search forms and infinite-scroll feeds.

**What happens if a page never finishes loading?**
A request you cancel before 70 seconds is billed, so set reasonable timeouts on your side. Failures that ScraperAPI itself can't complete aren't charged — only 200 and 404 responses consume credits.

## Which setup fits your situation

Here's the decision path, based on the pricing and rules above:

- **Your target's data appears with JavaScript disabled, or in `__NEXT_DATA__` / an XHR endpoint** → skip rendering entirely. Any HTTP client works; a scraping API is optional.
- **You need rendering occasionally, under ~10,000 rendered standard pages a month** → Hobby at $49 ($44.10 annual) is sized almost exactly for that, with 20 concurrent threads. 👉 [Check current Hobby pricing and start the 7-day trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
- **Volume above 100,000 credits monthly, or you need country-level geotargeting outside US/EU** → Business ($299, 3M credits, 100 threads, national geotargeting) is the first tier with both.
- **You're running high volumes of protected pages that need bypasses on nearly every request** → cost-check with the `urlcost` endpoint first, then look at Scaling ($475) or higher, where pay-as-you-go keeps overruns predictable.

One last check before you buy: run two or three of your real target URLs through the free trial with `render=true` and watch the `sa-credit-cost` header. If a "rendered" page you expected to cost 11 credits costs 36 (rendering plus a bypass), you'll want that number in your math before picking a tier, not after. 👉 [Start the free ScraperAPI trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
