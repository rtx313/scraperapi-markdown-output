# ScraperAPI output format markdown: how to request it, markdown vs text vs json/csv, and whether it costs extra credits (cURL and Python examples)

Add `output_format=markdown` to a ScraperAPI request and the response comes back as LLM-friendly markdown instead of raw HTML. The parameter sits in the query string next to `api_key` and `url`, and `output_format=text` does the same job with plainer output. Neither one adds credits to the request.

Below: the exact request syntax for cURL and Python, the rule that decides between markdown, text, json and csv, the limits the docs state outright, and the current plan prices so you can size the bill before writing any pipeline code.

> **Quick pick:** for an LLM ingestion job, start on Hobby at $49/month (100,000 credits, 20 concurrent threads) and move to annual billing once your parser is stable, which brings it to $44.10/month or $529.20/year. Continuous crawling at volume pushes you to Startup (1,000,000 credits, 50 threads, $149/month). Output format costs nothing on any tier, so the plan decision comes down to credits and concurrency alone.
> 👉 [Start on the Hobby plan](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Every ScraperAPI plan, priced

| Plan | Credits / month | Concurrent threads | Price | Start |
| --- | --- | --- | --- | --- |
| Free | 1,000 | 5 | $0 | [Free plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 20 | $49/mo, or $44.10/mo billed annually ($529.20/yr) | [Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | $149/mo, or $134.10/mo billed annually ($1,609.20/yr) | [Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | $299/mo, or $269.10/mo billed annually ($3,229.20/yr) | [Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | $475/mo, or $427.50/mo billed annually ($5,130/yr) | [Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 (+250,000 bonus credits on the current offer) | 300 | $975/mo, or $877.50/mo billed annually ($10,530/yr) | [Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 (+500,000 bonus credits on the current offer) | 500 | $1,975/mo, or $1,777.50/mo billed annually ($21,330/yr) | [Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom quote | [Talk to sales](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Annual billing knocks roughly 10% off every tier. Three billing details matter more than the headline price. Credit rollover isn't offered on any plan, so unused credits are gone at renewal. Auto-upgrade, which bumps you to the next tier when credits run out, is opt-in on Hobby through Professional and unavailable on Advanced and Enterprise. Pay-as-you-go past your limit exists only on Scaling and above.

Two feature lines split the cheap tiers from the rest. Hobby and Startup geotarget to the US and EU only; Business and above go country-level, which matters if you're collecting localized pages for training data. Analytics history is capped at the last 30 days on the two entry plans and unlimited from Business up.

A free plan gives you 1,000 credits and 5 threads per month, and a 7-day trial with 5,000 credits runs without a credit card.

## How to send a request that returns markdown

The base endpoint is `https://api.scraperapi.com/`, with all parameters passed as query parameters. ScraperAPI publishes the same example in cURL, Python and Ruby, so you can copy the one matching your stack.

cURL:

bash
curl "https://api.scraperapi.com/?api_key=API_KEY&url=https%3A%2F%2Fexample.com%2Farticle&output_format=markdown"


Python:

python
import requests

params = {
    "api_key": "API_KEY",
    "url": "https://example.com/article",
    "output_format": "markdown",
}

response = requests.get("https://api.scraperapi.com/", params=params)
print(response.text)


Swap the value to `text` and you get plain text back from the same URL. Nothing else in the request needs to change, which means you can add the parameter to an existing scraper and compare output on a handful of URLs before committing.

The scope is broader than the usual "supported sites" list. ScraperAPI's documentation describes markdown and text as LLM-friendly output available "for every URL on the web."

## markdown, text, json or csv: the rule that settles it

Four output modes exist and they don't cover the same URLs. That single difference decides most of the choice.

| Format | What comes back | Which URLs | How to trigger it |
| --- | --- | --- | --- |
| `markdown` | LLM-friendly structured response | Any URL | `output_format=markdown` |
| `text` | Plain text response | Any URL | `output_format=text` |
| `json` | Parsed structured fields | Domains on the autoparse support list only | `autoparse=true`, optionally with `output_format=json` |
| `csv` | The same parsed fields as CSV | Domains on the autoparse support list only | `autoparse=true`, optionally with `output_format=csv` |

Read that as two separate products. Markdown and text are rendering options: ScraperAPI fetches the page and strips it down to something a language model can consume without a BeautifulSoup pass. `autoparse` is a parsing product: it returns named fields, but only for domains the team has built extractors for, and the list lives on the documentation's json-response-autoparse page.

So the questions to ask are simple. Do you want the whole page in readable form, on any site? Use markdown, or text if you're chunking for embeddings and every heading marker is noise. Do you want a price, a title and a stock status as separate keys, from a retailer that's on the support list? Use autoparse with `output_format=json`, or csv if the output feeds a spreadsheet or a bulk import.

Anything else is a compromise. Running a generic markdown pass over a product page gives you the page, not the fields. Requesting json from a domain that isn't supported gives you nothing structured.

## Limits to know before you build on it

**Proxy mode isn't supported yet.** The documentation splits examples into API REQUEST, ASYNC REQUEST and PROXY MODE, and the proxy tab is labelled "COMING SOON." If your architecture routes through the proxy endpoint, you can't get markdown out of it as of the documentation's current version. Async requests are covered.

**One note about Premium is genuinely unclear.** The Customizing Requests page carries a "(Can not be combined with Premium)" annotation, and it isn't possible to tell from the published text which parameter that annotation attaches to. Don't assume `output_format` is blocked with `premium=true`, and don't assume it's fine either. Send one test request with both before you design around it.

**Several combinations are undocumented.** Nothing in the docs covers case sensitivity, passing multiple values, or pairing `output_format` with `render`, `country_code` or `screenshot`. The safe move is a small test matrix on your own target URLs rather than a guess.

**You can't preview the shape.** The cURL tutorial has "Sample Response: Markdown / Text" headings, but the blocks carry no content. There's no reference sample showing how a given page converts, so validate the output on real URLs from your own list before writing a parser that depends on specific formatting.

None of these are dealbreakers for a straightforward "fetch page, feed model" job. They matter if you're building something that has to run unattended across thousands of domains.

## Does markdown output cost extra credits?

No. ScraperAPI's credit table lists geotargeting, sessions, device type, output format and auto parsing as free, with no extra cost. The credits you spend are determined by the URL and the rendering features you switch on:

- Standard page: 1 credit
- E-commerce sites (Amazon, Walmart, eBay): 5 credits
- Google or Bing, including subdomains: 25 credits
- LinkedIn: 30 credits
- Bot-protection bypass (Cloudflare, DataDome, PerimeterX/HUMAN, Akamai): 10 credits
- `render=true`, `premium=true`, or `screenshot=true`: 10 credits each
- `premium=true` plus rendering: 25 credits
- `ultra_premium=true`: 30 credits, or 75 with rendering

That changes what a markdown pipeline actually costs to run. A 100,000-credit Hobby plan covers 100,000 plain page fetches, but if the pages you need sit behind JavaScript rendering, each one bills at the rendering rate and your effective page count drops accordingly. Check the `sa-credit-cost` response header on the first few calls to see what each URL really costs, and set `max_cost` per request so a single expensive domain can't burn through a month's allowance.

Going over your concurrency limit returns a 429 rather than an extra charge; the request is declined, not queued.

👉 [Check current plan pricing and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## FAQ

**Does `output_format=markdown` work with async requests?**
Yes. The documentation lists API REQUEST and ASYNC REQUEST as supported request types for LLM output formats, with proxy mode marked coming soon.

**Can I get markdown from any website?**
Per the docs, yes for every URL on the web. Structured json and csv are the restricted ones, limited to the autoparse support list.

**What's the difference between `autoparse=true` alone and `autoparse=true` plus `output_format=json`?**
`autoparse=true` turns parsing on. Adding `output_format=json` or `output_format=csv` selects the shape of the parsed result for supported domains.

**Do I need a paid plan to test the format?**
No. The 7-day trial includes 5,000 credits with no card required, and the free plan gives 1,000 credits and 5 threads per month. Output format costs nothing extra, so a trial is enough to confirm the markdown quality on your own URLs.

**Is there a cheaper way to run markdown over thousands of pages?**
The lever is the credit cost per request, not the output format. Dropping `render=true` where the target doesn't need it, and staying away from `ultra_premium` unless a site demands it, will cut your bill far more than any plan swap.

## Which setup fits your job

Testing whether markdown output suits your pipeline: free plan or the 7-day trial, one parameter, ten URLs you already know well. Nothing to lose.

Running a recurring LLM ingestion job on plain pages: Hobby at $49/month is enough to start, and the annual rate at $44.10/month is worth taking once the parser works.

Crawling JavaScript-heavy pages continuously: count your real per-request credits first, then size the plan from that number. Startup at 1,000,000 credits and 50 threads is the realistic floor for a job that runs daily across a few thousand URLs.

Needing country-level geotargeting for localized content: Business is the first tier that offers it, so the $299/month price includes a feature you can't buy on Hobby or Startup at any discount.

Two things to confirm before you pay. Test `output_format=markdown` combined with any other parameters you rely on, particularly `premium=true` given the unclear docs note. And remember that credits don't roll over, so pick the tier you'll actually consume each month rather than the one that looks generous on paper.

👉 [Pick your plan and run a test request](https://www.scraperapi.com/pricing?fp_ref=coupons)
