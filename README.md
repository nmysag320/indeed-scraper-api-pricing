# indeed scraper api: How to Pull Job Listings Without Getting Blocked, What Each Search Really Costs, and Which Plan Fits Your Volume

Searching for an indeed scraper api usually ends the same way: you copy a Python tutorial, run it, and get a 403 instead of job listings. That's the current reality. Indeed's old Publisher Job Search API is gone — `api.indeed.com` no longer even resolves in DNS — and the site now screens automated traffic behind Cloudflare, TLS fingerprint checks, and rate limits.

So the useful question is which scraping API can actually get you the listings, and what each page of results costs once you account for rendering and residential proxies. This guide covers both, with ScraperAPI's current plan lineup as the reference point, plus a few billing details most comparison posts skip.

> **Quick take:** For most job-data projects, the realistic setup is JS rendering + residential proxies, which runs about **25 credits per search page** on ScraperAPI. The Hobby plan ($49/month, 100,000 credits) covers roughly 4,000 of those pages. If you want to test the waters first, 👉 [start the 7-day trial with 5,000 free credits](https://www.scraperapi.com/signup?fp_ref=coupons) — no credit card required.

## Why your own scraper gets a 403

Direct scraping fails at the network edge, before any HTML arrives. A documented test from mid-2026 spells it out: a `requests` + BeautifulSoup script hitting Indeed's `/jobs` and `/viewjob` pages returned HTTP 403 behind a Cloudflare "Security Check - Indeed.com" page — and that held even with a real Chrome User-Agent and `Accept-Language` headers set. The challenge page itself was about 59,000 bytes of JavaScript, not job data.

The reason a fake UA doesn't help is TLS fingerprinting. Libraries like `requests` have a TLS handshake signature (the JA3 fingerprint) that looks nothing like a real browser, so Indeed flags the request as a bot before reading a single header. Behind that first layer sit CAPTCHA pages, HTTP 429 rate limits when one IP requests too fast, and server-side layout personalization that changes the page depending on your geography and A/B test group.

And Indeed doesn't exactly hide its position: the robots.txt explicitly disallows `/jobs`, `/viewjob`, and the mobile equivalents, with company pages under `/cmp/` restricted as well. For a compliant crawler, the search and listing pages are off-limits by the site's own rules — which is why anyone pulling job data at scale ends up weighing the tradeoffs described later in this guide.

## What an Indeed search actually costs on ScraperAPI

ScraperAPI charges in credits, and Indeed sits in the standard tier: a base request is 1 credit. The site isn't on ScraperAPI's special-domain list (Amazon runs 5, Google and Bing 25, LinkedIn 30), so the base cost is the cheapest category. The real cost comes from the parameters you need to get past the blocks.

Here's the parameter pricing from the official docs (last updated August 2026):

| Request configuration | Credits per request |
| --- | --- |
| Plain request | 1 |
| `render=true` (JS rendering) | 11 |
| `premium=true` (residential proxy) | 1 |
| `render=true` + `premium=true` | 25 |
| `ultra_premium=true` | 31 (paid plans only) |
| `ultra_premium=true` + `render=true` | 75 (paid plans only) |
| Cloudflare / Datadome / PerimeterX bypass (when triggered) | +10 |

Since Indeed builds its listings with JavaScript and blocks datacenter IPs, the practical configuration is render + premium proxy: **25 credits per page**. Plan your budget around that number, not the "from 1 credit" figure.

A few billing details that affect your real spend, straight from the documentation:

- You're only charged for successful requests — HTTP 200 and 404 responses. Failed requests on the API's side cost nothing.
- If you cancel a request yourself within 70 seconds, it still counts as billed. Let jobs finish or use async mode.
- Every response includes a `sa-credit-cost` header showing exactly what that request consumed. Check it on your first few calls to catch configuration mistakes before they burn through a plan.
- `country_code`, `session_number`, `device_type`, and `output_format` are all free add-ons — geo-targeting costs nothing extra.

Before any large run, you can price a URL precisely. There's a Domain Cost Estimator in the dashboard, an API Playground for testing requests, and a plain endpoint: `https://api.scraperapi.com/account/urlcost?api_key=YOUR_KEY&url=...&render=true&premium=true`. It accepts all the regular parameters, so you can confirm the exact per-request cost before committing thousands of credits. The `max_cost` parameter then acts as a hard ceiling on any single request.

Want to run the numbers against your own volume? 👉 [check the current plan prices and credits](https://www.scraperapi.com/pricing?fp_ref=coupons) and divide your monthly page target by 25.

## ScraperAPI plans and pricing

ScraperAPI currently lists seven paid tiers plus a free plan. Every paid plan includes JS rendering, premium proxies, CAPTCHA and anti-bot handling, JSON auto-parsing, rotating proxy pools, custom headers and sessions, universal retries, unlimited bandwidth, and a 99.9% uptime guarantee.

| Plan | Monthly | Annual (per month) | Credits/month | Concurrent threads | Link |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [ Try the free plan](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [ Start Hobby trial](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [ Start Startup trial](https://www.scraperWee1v1) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [ Start Business trial](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [ Start Scaling trial](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [ Start Professional trial](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,0,0 | 500 | [ Start Advanced trial](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [ Contact sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

Paying annually takes 10% off every tier. Two limits worth reading before you pick:

- **Geography:** Hobby and Startup are restricted to US & EU geo-targeting; Business and above unlock country-level targeting worldwide. If you need Indeed results from specific countries outside the US and EU, Hobby won't do it.
- **Concurrency:** exceeding your thread limit returns HTTP 429 — requests get rejected rather than queued, and you're not charged for them. For a job crawler, concurrency is your throughput ceiling, so a bigger plan means faster collection even at the same credit math.

When credits run out, scraping pauses by default until renewal. Hobby through Professional accounts can enable auto-upgrade, and Scaling and above can switch on pay-as-you-go to keep scraping at a fixed per-credit rate with a monthly spending cap you set.

## How ScraperAPI stacks up against the other Indeed routes

There are three other common approaches, and each has a clear use case.

**ScrapingBee** works much the same way. A community-built npm package (`indeed-scraper-api`) wraps its API: you pass a job title and city, and it returns a JSON array with title, company, and location, with headless rendering and residential proxies handled server-side. The cost math is nearly identical — residential proxies plus JS rendering come to 25 credits per call there too. Its free tier is smaller though: 1,000 credits with no card, which works out to about 40 rendered Indeed pages. ScraperAPI's 7-day trial gives 5,000 credits — roughly 200 pages at the same configuration — which is a better sandbox for figuring out whether your parsing logic holds up at volume.

**Free open-source scrapers** exist on GitHub, and one popular project returns surprisingly rich data — salary ranges, experience level, benefits, skills — as clean JSON without an API key or proxies. The catch is maintenance: Indeed changes its markup and defenses regularly, and an open-source scraper only works until the next update breaks it. For a one-off dataset or a weekend experiment, it's a reasonable starting point. For anything you need running next month, an API that absorbs the blocking changes for you tends to age better.

**Apify** hosts dedicated Indeed actor templates with filters for remote/hybrid, experience level, posting age, and country domain — handy if you want a no-code setup and don't mind paying per actor run.

None of these services have a special Indeed deal. Whichever you pick, you're paying rendering-plus-proxy rates, so per-request cost rarely decides it. Reliability, concurrency limits, and the tools for monitoring credit burn usually matter more.

## Running your first Indeed scrape

The workflow is short. Once you have an API key from signing up, the sequence looks like this:

1. **Price the target first.** Call the `urlcost` endpoint with your exact parameters — `render=true&premium=true` — and confirm the per-request cost. With Indeed, that should come back at 25 credits.
2. **Send the request through the API.** Point it at an Indeed search URL, with rendering and premium proxy enabled. The API rotates IPs and solves the challenge on its side; you receive the rendered HTML.
3. **Parse the job cards.** In the returned HTML, listings live in `div.job_seen_beacon` and the location under `data-testid='text-location'`.
4. **Keep the `jk` value.** Every listing carries a job key (`data-jk`) — a stable ID that drives the `/viewjob?jk=` detail page. Store it from day one; it's what you'll use to deduplicate results and re-crawl individual postings later.
5. **Watch your burn rate.** Read the `step 5` header on each response, and set `max_cost` so a misconfigured request can't silently eat your balance.

A detail worth flagging: salary data appears on result cards inconsistently and is often missing, and posting dates arrive in relative form ("3 days ago"). If you need structured salary or full descriptions, plan a second pass over the `/viewjob` pages for the postings you care about, rather than trying to extract everything from search results.

## The legal and policy side, briefly

Scraping Indeed sits in a gray zone you should understand before building a business on it. On the technology layer, the site says no: 403s, robots.txt disallow directives on the pages you want. On the legal layer, the picture is more mixed — in the hiQ Labs v. LinkedIn case, the Ninth Circuit held that scraping publicly accessible pages doesn't violate the CFAA, and the Supreme Court's Van Buren decision narrowed that statute further. But hiQ itself still ended up liable for $500,000 for breaching LinkedIn's user agreement, which is a useful reminder that contract terms are a separate risk from computer-crime law.

The practical guardrails: stay on publicly visible pages, don't touch anything behind a login, avoid collecting personal data about individuals (which triggers GDPR in Europe), and get legal advice before any commercial deployment. Pulling job listings for market analysis is a very different posture from harvesting applicant personal information.

## FAQ

**Does Indeed still have an official API?** No. The Publisher Job Search API has been retired and `api.indeed.com` no longer resolves. Programmatic access now runs only through Indeed's partner and OAuth channels, aimed at approved job aggregators.

**Is the free ScraperAPI plan usable for Indeed?** Barely. You get 1,000 credits and 5 concurrent threads, so at 25 credits per rendered page that's around 40 search pages — enough to validate your parsing, not enough for a real dataset. Note that `ultra_premium` requests require a paid plan, so if standard premium proxies don't clear the block on your target pages, the free tier can't escalate.

**What happens when my credits run out mid-project?** Scraping pauses until your credits renew. Auto-upgrade (opt-in, Hobby through Professional) bumps you to the plans automatically, while pay-as-you-go (Scaling and up) keeps scraping at a fixed per-credit rate under a monthly cap you define. Annual plans can also renew early for a fresh batch.

**Why did my request cost more than 25 credits?** Check whether a bypass triggered — Cloudflare, Datadome, or PerimeterX bypasses each add 10 credits when they activate. The `sa-credit-cost` response header tells you exactly where the credits went.

## Which plan makes sense for you

Match the plan to your page volume at 25 credits per rendered search page, and the decision mostly makes itself:

- **Just validating an idea:** the free plan's 1,000 credits, or better, the 7-day trial's 5,000 — about 200 pages, enough to prove your parser works.
- **A recurring job tracker for one or two roles:** Hobby at $49/month covers roughly 4,000 pages, plenty for scheduled searches across a few cities. If you're outside the US/EU, you'll need Business for country targeting instead.
- **A job board or market dashboard pulling daily across many locations:** Startup ($149) to Business ($299) territory, with 1M–3M credits and 50–100 threads for reasonable collection speed.
- **Large-scale, always-on collection:** Scaling and above, where pay-as-you-go and country-level targeting stop being optional extras.

Before you commit to a paid tier, two things to verify: run the `urlcost` check with your exact parameter set so your math uses real numbers, and confirm your monthly page estimate against the concurrency limit — a plan with lots of credits but too few threads can be the actual bottleneck. When the numbers check out, 👉 [activate the 7-day trial with 5,000 free credits](https://www.scraperapi.com/signup?fp_ref=coupons) and test your full pipeline before the first invoice arrives.
