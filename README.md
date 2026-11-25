# Craigslist Scraper API: What Does 10,000 Listings Cost, Why Do You Get a 403, and Is the Output JSON or HTML?

Scraping Craigslist runs into three walls, and only one of them is something a scraping API can fix. Craigslist cuts any single search off at 10,000 results, no matter what tool you use. It throttles or blocks IPs that hit it too fast, which on a cloud host usually looks like a bare HTTP 403. And whatever comes back is page HTML that still needs a parser before it becomes rows in a table.

ScraperAPI handles the middle problem, the one that stops most first attempts: rotating proxies and rendering so your requests come from somewhere Craigslist will answer. It bills per request in credits, with a standard page at 1 credit. The plan with a published monthly price is Startup at **$149/month for 1,000,000 credits**, which at 1 credit per request works out to roughly 99 full crawls of a 10,000-listing search. Two things it does not do: it won't get you past Craigslist's 10,000-result ceiling, and it won't hand you a Craigslist listing schema, so budget time for the parser.

Below: every current plan with prices, what a Craigslist crawl actually burns in credits, the three site limits, and the cases where a free local script beats any subscription.

> **Quick pick:** most people building a scheduled Craigslist pipeline should start on **Startup**: 1,000,000 credits/month for **$149** ($134.10/month if billed annually, $1,609.20/year), 50 concurrent threads. Still testing? The free plan gives 1,000 credits and 5 threads, and the trial gives 5,000 credits for 7 days with no card.
> 👉 [See Startup pricing and the annual rate](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Every ScraperAPI plan, and what each one is actually for

Credits reset when your subscription renews, so the monthly figure is a hard ceiling, not a bank you draw down over time.

| Plan | API credits/month | Price | Threads | Best for |
| --- | --- | --- | --- | --- |
| Free | 1,000 | $0 | 5 | Smoke tests, a few hundred pages |
| Hobby | 100,000 | See pricing page | 20 | Small side projects, one city |
| Startup | 1,000,000 | $149/mo or $134.10/mo annual | 50 | A recurring Craigslist pipeline  [Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | 3,000,000 | See pricing page | 100 | Several metros, daily runs |
| Scaling | 5,000,000 | $475/mo or $427.50/mo annual | 200 | Multi-source operations |
| Professional | 10,500,000 | See pricing page | 300 | Heavy continuous crawling |
| Advanced | 21,500,000 (+500K bonus credits) | $1,975/mo or $1,777.50/mo annual | 500 | Pipeline plus priority routing |
| Enterprise | 22,000,000+ | Custom quote | 500+ | Negotiated volume |

Hobby, Business and Professional are priced on the same page as the rest; Enterprise goes through sales. Annual billing saves about 10% across the three tiers where the numbers are public: $178.80/year on Startup, $570 on Scaling, $2,370 on Advanced.

What differs between tiers beyond volume:

- **Geotargeting.** Free, Hobby and Startup cover US and EU only. Business and above add country-level targeting worldwide.
- **Analytics history.** 30 days on Free, Hobby and Startup; unlimited from Business up.
- **Pay-as-you-go** (keep scraping past your credit cap at a fixed per-credit rate) is available on Scaling, Professional, Advanced and Enterprise. Auto-upgrade to the next plan is available on Hobby, Startup, Business, Scaling and Professional.
- **Concurrency is a ceiling, not a target.** Going over it returns a 429 and the request is refused rather than queued, with no charge. Craigslist throttles fast crawlers anyway, so 200 threads is headroom for other targets, not extra Craigslist speed.

## What a Craigslist request really costs in credits

The base rate is 1 credit for a standard page. Add-ons stack on top.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| JS rendering (`render=true`) | 10 |
| Premium proxies (`premium=true`) | 10 |
| Bot-protection bypass (Cloudflare, DataDome, PerimeterX, Akamai) | 10 |
| Premium proxies + JS rendering | 25 |
| Ultra premium | 30 |
| Ultra premium + JS rendering | 75 |
| Geotargeting, sessions, output format, auto parsing | free |

Craigslist search and listing pages are plain HTML, so a bare request should land at 1 credit. If ScraperAPI has to clear a bot check on that URL, the same page can cost 11. That gap decides your budget, and you don't have to guess: the dashboard has a Domain Cost Estimator that prices any URL before you spend, and the `max_cost` parameter caps a single request. Use both on a Craigslist URL before you commit to a tier.

## The three Craigslist limits no API can remove

**10,000 results per search.** Craigslist itself stops at 10,000, so no proxy pool changes it. Split large queries by subarea (borough, neighbourhood) or by price band and run them as separate searches. A full 10,000-result search is about 84 result pages at 120 listings each.

**Rate limiting and 403s.** Craigslist slows down or blocks crawlers that hit it too fast, and requests from datacentre IPs often get a flat 403. Two fixes work together: keep concurrency low, and route through residential IPs. This is the part ScraperAPI is actually selling you.

**Contact details.** Phone numbers and reply emails sit behind Craigslist's reply button and are not in the page HTML. Some tools parse phone numbers and emails out of the posting body and label them as contact info, which is accurate only when the seller typed them into the description. Nobody is reading the reply button without clicking it.

One legal note, briefly: scraping public listings is generally treated as lawful in the US, but Craigslist's terms of service, plus GDPR/CCPA and fair-housing and equal-employment rules for housing and job ads, stay your responsibility.

## JSON or HTML: the parsing bill

ScraperAPI is priced per request and per feature, not per listing. You send a URL, you get that page, and you write the extractor for title, price, post ID, location and posted date. Auto parsing is listed as free on all plans, but the plan structure is request-based, so plan for parser work rather than a ready-made Craigslist schema.

That is the main structural difference in this market. Apify's Craigslist actors return one JSON record per listing, with fields like `price`, `priceText`, `currency`, `latitude`, `longitude`, `attributes`, `images` and `postedAt` already separated, and one of them detects deleted or flagged posts with an `isDeleted` flag. The omkarcloud open-source scraper exposes a local HTTP API that also returns JSON, 120 listings per page. Oxylabs' Craigslist scraper returns the raw page HTML in a `content` field, which puts you in the same position as ScraperAPI: parse it yourself.

The pricing follows from that. The per-listing Apify actors charge around $1.30 per 1,000 listings, dropping to $1.00 on their higher tier. A request-based API charges per page you fetch. Whether that is cheaper depends entirely on your credit cost, which is the next section.

## Run the numbers for your own volume

Take one full search of 10,000 listings and fetch every detail page: about 10,084 requests including the 84 result pages. At 1 credit each, that is ~10,084 credits. At 11 credits (bot-protection bypass triggering on each), ~110,924.

On Startup's 1,000,000 credits a month:

| Workload | At 1 credit/request | At 11 credits/request |
| --- | --- | --- |
| One full search with details | ~10,084 credits | ~110,924 credits |
| Full searches per month | ~99 | ~9 |
| Cost per 1,000 listing pages | ~$0.15 | ~$1.64 |

Read those two numbers against the market rate of $1.00–$1.30 per 1,000 listings for a ready-made JSON actor. At the 1-credit rate, ScraperAPI is roughly an order of magnitude cheaper per page, and you pay for it in engineering time. At the 11-credit rate you land slightly above the per-listing tools while still doing your own parsing, which is the scenario worth catching before you buy rather than after.

Twenty full searches a month is ~201,680 credits at 1 credit and ~2.2 million at 11. The first fits Startup comfortably. The second does not, and pushes you to Business's 3,000,000.

For daily re-crawls there is one more cost nobody mentions. Every request is billed, including pages you already scraped yesterday. A per-listing actor with a cache can skip known listings and charge only for new rows; with a request-based API, deduplication is a job you build on your side. On a daily run over the same searches, that difference compounds quickly.

## When ScraperAPI is the wrong buy

**You need a few thousand listings once.** Clone the omkarcloud scraper, run `python run.py`, hit `localhost:8000`. It is free, needs no API key, covers 700+ Craigslist sites across 79 countries and 134 categories, and returns JSON. The catch: on a cloud server you will hit 403s and need to point `CRAIGSLIST_PROXY` at a residential proxy you pay for separately.

**You want structured JSON without writing a parser.** The Apify actors above are the shorter path; you trade per-page cost for per-listing cost and skip the extraction work entirely.

**You only want HTML to parse yourself and volume is low.** Any generic scraping API will do it, and you can compare them on plain per-request pricing.

**Your volume is under ~50,000 pages a month and you will write the parser.** Startup is the first tier with a published price, and 1,000,000 credits covers that several times over. Going lower means Hobby, which you should price on the page rather than assume scales linearly.

## Which plan to pick

- **Testing whether Craigslist will answer you at all** → free plan, 1,000 credits and 5 threads, or the 7-day trial with 5,000 credits and no card.
- **A daily or weekly crawl of a handful of metros** → Startup, $149/month, 1,000,000 credits. Check the Domain Cost Estimator on one Craigslist URL first; if it reports 11 credits rather than 1, halve your expected volume.
- **Twenty-plus full searches a month, or a bypass on every request** → Business's 3,000,000 credits, and set `max_cost` so one bad URL can't eat a chunk of the month.
- **Multiple data sources, not just Craigslist** → Scaling or above, where pay-as-you-go stops a credit cap from killing a run mid-pipeline.

Three things to confirm before you enter a card: what your specific Craigslist URLs cost per request (estimator plus `max_cost`), that unused credits do not roll over at renewal, and that the 7-day no-questions-asked refund applies to the plan you chose. That refund window is enough to run a real crawl and see the actual credit burn.

👉 [Start the 7-day trial with 5,000 credits and price a Craigslist URL yourself](https://www.scraperapi.com/?fp_ref=coupons)
