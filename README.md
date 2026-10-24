# price scraping python: how to keep your script unblocked, and what each request actually costs before you commit to a plan

Price scraping with Python comes down to two problems: parsing the price out of the page, and getting your script to the page at all. The first takes about ten lines of code. The second is where scripts die — rate limits, IP blocks, geo-cloaked prices, and JavaScript-rendered numbers that don't exist in the raw HTML. This guide covers both: a working minimal scraper, why price pages fight back harder than most pages, and a full breakdown of ScraperAPI's plans and credit costs so you don't pick a tier based on a number that won't survive contact with Amazon.

**Quick picks, if you're in a hurry:**

- Just learning or testing the waters: start with the free 1,000 credits (5 concurrent requests), no card needed. The 7-day trial adds 5,000 more credits.
- Price monitoring on standard e-commerce sites at up to ~200,000 pages/month: Hobby at $49/month (or $44.10 billed annually) is the realistic entry point.
- Amazon/Google-heavy workloads: check the credit math section below before choosing — domain multipliers change everything.

👉 [Check ScraperAPI's current plans and prices](https://www.scraperapi.com/pricing?fp_ref=coupons)

## A minimal price scraper takes one HTTP client and one CSS selector

The smallest working version of a price scraper is a GET request plus an HTML parser pointed at a single stable CSS selector. Install two libraries and you're set:

bash
pip install httpx selectolax


The parsing side is genuinely simple. Pick a class name or a `data-` attribute that the site is unlikely to restructure, and avoid long selector chains like `div > div > div` — those break the next time the site redesigns its product page. Then normalize the result: a raw string like `"$1,299.00"` or the European format `"1.299,00 €"` needs cleaning before you convert it to a `Decimal`, and the currency symbol should be stored separately rather than assumed to be USD.

Two small details that separate scripts that work from scripts that get a 403 on the first request: send a realistic browser User-Agent (a Chrome one, not the default `python-requests` string) with an `Accept-Language` header, and store every observation as a row with a timestamp and currency so you can track price changes over time instead of keeping disconnected snapshots.

The parsing is the easy part. What breaks price scrapers in practice is access.

## Price pages are the most defended pages on the internet, for a reason

Your price scraper competes for the same pages that sneaker bots, scalper bots, and credential-stuffing attacks target, so e-commerce sites defend them with the same industrial-strength anti-bot stack. The numbers behind this: Imperva's 2025 Bad Bot Report found that automated traffic crossed 51% of all web traffic in 2024 — the first time in a decade it exceeded human activity — with malicious bots accounting for 37%.

Four specific things will bite a price scraper:

1. **Rate limits and user-agent filtering** — hammer a price page from one IP and you'll get 429s or silent blocks. Retry logic should use exponential backoff with random jitter (a few seconds doubling per attempt works) on 429 and 503 responses.
2. **Geo-cloaking** — the same product URL returns different prices, currencies, or even a "not available in your region" page depending on which country your request appears to come from. A scraper running from one datacenter sees exactly one market's pricing, which is useless if you're comparing prices across regions.
3. **Datacenter IP reputation** — hosting provider IP ranges are known and pre-flagged. Price pages are among the first to rate-limit or block them outright.
4. **JavaScript-rendered prices** — plenty of modern stores load the price via JS after the initial HTML arrives, so your parser finds nothing. You either reverse-engineer the underlying JSON API (fast and stable when the API endpoint isn't itself protected), run a headless browser like Playwright, or hand rendering off to a service.

Handling all of that yourself means maintaining a proxy rotation pool, retry logic, and browser infrastructure. This is the gap a scraping API fills: you send the target URL, the service handles proxies, CAPTCHAs, and retries, and returns the HTML or parsed JSON. ScraperAPI is one of the most established options here, with a pool of over 40 million residential, datacenter, and mobile IPs across 50+ countries.

## ScraperAPI plans and pricing: the full current lineup

Every paid plan includes the same feature set — JavaScript rendering, premium residential and mobile IPs, advanced anti-bot bypassing, structured data endpoints, DataPipeline, and full crawler access. Plans differ on credits, concurrency, geotargeting granularity, and analytics history. Annual billing takes 10% off every tier.

| Plan | Monthly price | Billed annually | Credits/month | Concurrent threads | Purchase |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [Sign up free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Start Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Start Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Start Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Start Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Start Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Start Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Contact sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

Two limits that live in the fine print and matter for price scraping specifically:

- **Geotargeting on Hobby and Startup is US & EU only.** The full 70-country list, selectable by two-letter country code, unlocks at Business and above. If your price comparison spans Asian or Latin American markets, the $49 tier won't get you there.
- **Hobby and Startup keep only 30 days of analytics history**; Business and up get unlimited. Minor for most, relevant if you audit your own scraping costs monthly.

Getting started costs nothing: the free plan gives 1,000 credits with 5 concurrent connections, and there's a 7-day trial with 5,000 credits that doesn't ask for a card — enough to run a real price-monitoring job against your actual target sites before paying anything.

👉 [Test your target sites with the 5,000-credit free trial](https://www.scraperapi.com/?fp_ref=coupons)

## The credit math the pricing page doesn't lead with

Here's the part almost every price-scraping tutorial skips: the "100,000 credits" on a plan is not 100,000 product pages. A credit is the cost of a *standard* request, and several things multiply it.

**Domain multipliers, applied automatically.** Per ScraperAPI's pricing documentation: a standard page costs 1 credit, Amazon costs 5, Google and Bing (including their subdomains) cost 25, and LinkedIn costs 30. You don't opt into these — hitting an Amazon product page spends 5 credits whether you knew about it or not. Sites behind Cloudflare, Datadome, or PerimeterX protection add 10 credits per request when the bypass succeeds.

**Parameter multipliers, applied when you enable features.** JavaScript rendering (`render=true`) costs 10 credits, premium proxies 10, the combination 25, and ultra-premium proxies 30 (75 with rendering added). This stack hits price scrapers disproportionately, because price scraping is exactly the workload that needs both rendering and premium pools most often.

Run the numbers on the Startup plan's 1,000,000 credits:

| Workload | Cost per request | Real coverage on 1M credits |
| --- | --- | --- |
| Plain HTML price page | 1 | 1,000,000 pages |
| Amazon product page | 5 | 200,000 pages |
| Amazon page + JS rendering | ~15 | ~66,000 pages |
| Rendered + premium on a protected site | 25–75 | 13,000–40,000 pages |

That last row is the gap between the sticker price and reality: one independent 2026 review calculated that on Amazon search pages with rendering enabled, 1M credits might cover only 40,000–100,000 actual pages — a per-page cost 20–30× the headline $0.000149. Their advice is worth repeating: budget against your real per-page cost, not the credit count on the pricing page.

Three ways to keep this under control:

- **Domain Cost Estimator** in the dashboard shows the exact credit cost for any URL before you scrape it at scale.
- **`max_cost`** sets a ceiling per request, so a misconfigured call can't silently eat 75 credits.
- **Failed requests are free.** ScraperAPI only deducts credits for successful requests; retries it performs on your behalf don't count against the balance. And when you exceed your concurrency limit, you get a 429 rejection rather than a queue — annoying but not billed.

## Which plan fits which price-scraping workload

Match the plan to your target sites and volume, not to the credit numbers:

- **Standard e-commerce sites, moderate volume** (boutiques, Shopify stores, unprotected retailers): a plain 1-credit request is often enough, no rendering needed. Hobby's 100,000 credits covers ~100,000 page fetches monthly — a serious amount for a monitoring job checking a few hundred products daily. At $49/month ($44.10 annually), this is the plan most solo developers should start on.
- **Amazon, Walmart, eBay price tracking:** use the structured data endpoints instead of scraping raw HTML. `/structured/amazon/search`, Walmart Search, eBay Product, and Google Shopping return parsed JSON with 18+ fields (price, ratings, seller info, BSR) across 21 Amazon regional marketplaces. You skip maintaining HTML parsers entirely, and when the retailer redesigns its page, your code keeps working. This changes the calculus on higher tiers: Startup's 1M credits covers ~200,000 Amazon product fetches without touching rendering.
- **Cross-market price comparison:** you need `country_code` beyond US/EU, which means Business ($299) or above. Also note the sticky session (`session_number`) expires 15 minutes after last use — fine for a scrape run, not for anything needing long-lived IP identity.
- **Mixed workloads where credits run out mid-month:** Hobby through Professional support opt-in auto-upgrade (automatically bumps you to the next tier when credits deplete), while Scaling and up support pay-as-you-go at a fixed per-credit rate up to a monthly cap you set. Neither happens by default — by default, scraping pauses until credits renew, which for a daily monitoring job means silently missing price data. Decide which behavior you want before your first budget crisis.

One honest caveat: if your targets sit behind heavy bot management like Cloudflare's or Akamai's, and you need this at high volume, expect the premium multipliers to dominate your costs. Reviews consistently flag this as ScraperAPI's weak spot for hostile targets — it shines on standard e-commerce and search targets where the default pool clears most requests without premium routing.

## When a scraping API isn't the right tool at all

For completeness: if your volume is tiny and your targets are unprotected, a script plus a proxy list billed by bandwidth can cost less than a credit system, since you'd be paying the API's multiplier overhead for requests that never needed it. Decodo (formerly Smartproxy) is the commonly cited alternative for straightforward residential proxy access billed by bandwidth. At the other end, developers comparing managed APIs in 2026 side-by-sides list Apify from about $29/month with a free plan, Oxylabs from about $30/month, Zyte from about $100/month, and Bright Data from about $499/month, per their published pricing — the high-volume, heavily-protected-target workloads tend to land on Zyte or Oxylabs. None of these change the recommendation for the typical use case here: a Python price-monitoring script against mainstream e-commerce targets, where a managed API saves you from building proxy rotation and retry infrastructure yourself.

## FAQ

**Does ScraperAPI work with BeautifulSoup and Scrapy?**
Yes — it's a plain HTTP endpoint, so any Python HTTP client works. Swap your request URL for the ScraperAPI endpoint with your API key and the target URL as parameters, add `render=true` or `country_code=us` when needed, and parse the returned HTML exactly as you would have. Official integrations exist for six languages.

**What happens if my credits run out mid-month?**
By default, scraping pauses until credits renew. Hobby through Professional can opt into auto-upgrade to the next plan; Scaling and above can enable pay-as-you-go up to a self-set monthly cap. Annual plans can renew early for a fresh batch of credits.

**Is the free trial actually usable for price scraping?**
The 7-day 5,000-credit trial is real and needs no credit card. On unprotected sites that's 5,000 page fetches; on Amazon product pages, 1,000. Enough to validate your selectors and measure your real per-page credit cost before committing.

**Does it only charge for successful requests?**
Yes, per its documentation: failed attempts after ScraperAPI's own retries don't consume credits, and concurrency-limit rejections (429) aren't billed either.

## Before you pick a plan

Three things worth doing in order. First, run your actual target URLs through the dashboard's Domain Cost Estimator on the free plan — your real per-page cost determines everything else. Second, size the plan against credits ÷ your typical request cost, not the raw credit figure; a Hobby plan on plain retail sites beats a misjudged Startup plan burning 25 credits per Amazon page. Third, if your pricing data spans regions outside the US and EU, start your comparison at Business, since the lower tiers are geo-limited.

If that describes your workload, 👉 [compare the full plan lineup on ScraperAPI's pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons) and start with the free 1,000 credits before spending anything.
