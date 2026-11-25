# Target Scraper: Buy a Dedicated Target Scraper or Build on a General Scraping API? Akamai, ZIP-Level Stock, Credit Math and the Full ScraperAPI Plan Lineup

Target.com puts Akamai Bot Manager in front of its pages and loads price, pickup and stock through delayed XHR modules, so an unrendered request comes back as a shell with no price in it. Every scraper you consider has to clear both hurdles, and that narrows the field to two purchases: a dedicated Target scraper that charges per delivered product and hands you finished rows, or a general scraping API that charges per request and leaves the parsing to your code.

What follows is the second route priced out properly, using ScraperAPI's published plan allowances, the credit cost of a rendered Target request, and the field-level details (TCIN, DPCI, variant stock, ZIP-dependent pricing) that decide whether the output is worth having. The comparison against per-item Target scrapers sits near the end, with the numbers that matter for a build-versus-buy call.

> Most teams fit on Hobby. Its 100,000 monthly credits cover roughly 2,850 fully rendered Target product pages, or around 68,000 products if you collect through search result pages at 24 products per page. If you need listing-level price tracking rather than full detail on every item, that allowance is generous. Test it first: the 7-day trial includes 5,000 credits and needs no card. 👉 [Start the free ScraperAPI trial](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Full ScraperAPI plan lineup

Two things decide whether a plan fits a Target job: the monthly credit allowance, which sets how many rendered requests you can make, and the concurrent thread count, which sets how fast a large catalogue finishes. Dollar rates differ by billing cycle and change with promotions, so the table below sticks to the allowances and links each plan to the page that shows its current rate.

| Plan | API credits per month | Concurrent threads | Geotargeting | Current rate |
| --- | --- | --- | --- | --- |
| Free trial | 5,000 total, 7 days | — | — | [Trial details](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 20 | US & EU | [Hobby rate](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU | [Startup rate](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level | [Business rate](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level | [Scaling rate](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Country-level | [Professional rate](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Country-level | [Advanced rate](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Country-level | [Enterprise rate](https://www.scraperapi.com/pricing?fp_ref=coupons) |

One detail worth catching: US and EU geotargeting is included from Hobby upward, and country-level targeting only starts at Business. Target.com is a US site priced in USD, so `country_code=us` does everything you need, and geotargeting adds no credit cost. You do not have to buy Business for location reasons.

Pay-as-you-go overflow is available on Scaling, Professional, Advanced and Enterprise, at a fixed per-credit rate up to a monthly cap you set. Auto-upgrade, which moves you to the next plan when credits run out, is opt-in on Hobby, Startup, Business, Scaling and Professional.

## Where Target.com breaks a naive scraper

Akamai is the first wall. Target's own product pages, store pages and sitemaps refuse some datacenter IPs outright, and the working configurations for Target specify US residential proxies alongside the bot-protection bypass. A setup that only rotates datacenter IPs will look fine in testing and then return 403s in production.

The second wall is the page itself. Target does not put price, fulfillment or reviews in the main HTML document. Those arrive in XHR calls to `deferred_enrichment/modules`, with module types such as `ProductDetailWebDatasourceFulfillmentAndVariations`, `ProductDetailWebDatasourceWithStore` and `ProductDetailReviewsAndQuestions`. A working scraper renders JavaScript, waits for that XHR pattern to appear, and allows a few seconds of settling time before reading the DOM. If the module is missing, the parse fails, and there is no price to fall back on.

The third wall is credentials. Target's own aggregation endpoints live on `redsky.target.com`, and `store_location_v1` requires both a `key` and a `visitor_id`. Those are not published anywhere. The common approach is a warm-up request to the Target homepage, capturing the `store_location_v1` XHR from that session, extracting the two parameters, then calling `product_summary_with_fulfillment_v1` with the same session and the store and ZIP you care about.

With ScraperAPI, `render=true` handles the JavaScript, `premium=true` puts residential and mobile IPs behind the request, and `session_number` keeps the same proxy across the warm-up and the follow-up call. You still write the credential capture and the parser. That is the trade: you own the Target-specific logic, and you pay per request rather than per row.

## What one Target request costs in credits

ScraperAPI prices by request and by the features that request needs. The published rates:

| Request type | Credits |
| --- | --- |
| Standard page | 1 |
| E-commerce fast path (Amazon, Walmart, eBay) | 5 |
| SERP (Google, Bing) | 25 |
| Bot-protection bypass, including Akamai | +10 |
| JS rendering (`render=true`) | +10 |
| Premium residential proxies (`premium=true`) | +10 |
| Premium proxies plus JS rendering | 25 |
| Ultra premium | 30 |
| Ultra premium plus JS rendering | 75 |
| Geotargeting, sessions, headers, device type, output format | no extra cost |

Target is not on the 5-credit e-commerce list, so a Target request starts at the standard 1 credit and adds features. A rendered product page through premium residential IPs with Akamai bypass comes to roughly 25 to 35 credits: 25 for the premium proxy and rendering combination, plus 10 when the Akamai bypass kicks in. Every response carries its actual cost in the `sa-credit-cost` header, the dashboard's Domain Cost Estimator prices any URL before you send it, and `max_cost` caps a single request so a surprise cannot drain an allowance.

That puts 1,000 rendered product pages at 25,000 to 35,000 credits, or 25 to 35 percent of Hobby's monthly allowance. 👉 [See how many credits each plan includes](https://www.scraperapi.com/pricing?fp_ref=coupons)

### Bulk through search pages, then fetch detail only where you need it

The cost per product drops sharply if you stop treating one request as one product. Target's search results paginate in steps of 24 (`Nao = page * 24` in the URL), so a single rendered search page carries about two dozen products with title, brand, current price and comparison price visible in the listing cards. At 25 to 35 credits per page, 1,000 products costs roughly 1,100 to 1,500 credits instead of 25,000 to 35,000.

Use the search pages for price monitoring and reserve product-page requests for the items where you need DPCI, UPC, specifications or variant-level stock. That split is the difference between Hobby covering a few thousand products a month and Hobby covering tens of thousands.

## What you can actually get from a Target product page

The fields that make Target data joinable across runs, with a real record as the example:

| Field | Example | Why it matters |
| --- | --- | --- |
| `tcin` | 89110863 | Target's own product ID, taken from the `A-` segment of the URL; the stable key for joining runs |
| `dpci` | 072-04-0231 | Target's internal classification; absent on Target Plus marketplace items |
| `upc` | 810002864582 | Matches your own catalogue and supplier feeds |
| `gtin13` | 0194644278939 | Kept as text so leading zeros survive a CSV export; US codes are usually the 12-digit UPC with a leading zero |
| `price` / `regularPrice` / `isOnSale` | 69.99 / 69.99 / false | Sale prices always land in `price`; promotion objects only appear when Target attaches one |
| `variants[].tcin` | per color and size | Each variant has its own TCIN, price and sold-out state |
| `shippingQuantity` | search results only | Category and TCIN lookups return store pickup quantity instead |

The sample record: a Gourmia 6qt Digital Window Air Fryer, TCIN `89110863`, price 69.99, regular price 69.99, not on sale, availability `IN_STOCK`, rating 4.1 from 2,250 ratings and 991 reviews, category Air Fryers, specifications 6 Quart and 1500 Watts, badges "Rarely returned" and "Highly rated".

Two traps sit in that table. A parent TCIN returns the lowest price among its variants and no store availability at all; per-variant stock and sold-out status require the child TCINs listed in `variants`. And marketplace items sold by Target Plus sellers have no DPCI, so a pipeline that keys on DPCI will drop them silently unless you handle the null.

## No ZIP code, no stock

Prices, pickup, delivery and shipping on Target.com all depend on location. The same item can show a different price and a different set of fulfillment options depending on which store serves the ZIP you pass. Default setups commonly use `10001`, which resolves to the Chelsea 23rd and 8th store in New York (store ID `3412`).

Pick your ZIP deliberately. If you monitor a competitor's pricing in Texas, a New York ZIP gives you New York numbers. If you track online-only availability, note that store-specific stock quantities are a different thing from the published availability value, and some tools return only the latter. 👉 [Check plans with US geotargeting included](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Volume caps you hit before your credits run out

Target shows at most 1,200 products for a single search term or a single category. Past that, results repeat or stop. Collecting 5,000 products from one keyword is not a budget problem, it is an impossibility, so plan on splitting into narrower keywords and subcategories.

Tool-level caps sit under that ceiling. One Apify Target scraper caps category collection at 40 source pages, roughly 960 products before duplicates, with a `limit` of 1 to 2,000 and a default of 96. The other allows up to 10,000 products per run but collects search terms first, then categories, then TCINs, so a low limit can be exhausted before your product list is reached.

## ScraperAPI against dedicated Target scrapers

Both Apify tools below deliver finished Target rows; ScraperAPI delivers rendered HTML and leaves parsing to you. Neither model is better in the abstract, and the per-item prices land close enough that the deciding factor is usually who maintains the parser.

| Option | Pricing model | What you get | Where it fits |
| --- | --- | --- | --- |
| Apify Target scraper (automly) | $1.55 per 1,000 products plus a small per-run start fee; $5 free monthly usage covers about 3,200 products | Finished rows with TCIN, DPCI, UPC, variants, badges; ZIP parameter defaults to 10001; up to 10,000 products per run | Teams that want clean data without writing a parser |
| Apify Target scraper (s-r) | $0.001 per run start plus $0.0015 per delivered product, i.e. $1.50 per 1,000; free users get 10 rows per run | Category and product URL input; detail fetch included at no extra charge; category capped at 40 pages, about 960 products | Smaller jobs and catalogue matching on EAN codes |
| ScraperAPI | Credits per request: 1 standard, +10 rendering, +10 premium proxies, +10 Akamai bypass | Rendered HTML for any Target URL, US geotargeting, sessions, your own parser | Teams already parsing sites, or mixing Target with other retailers on one bill |

A rough conversion: 1,000 finished Target products costs about $1.50 to $1.55 on the dedicated scrapers. On ScraperAPI, the same 1,000 products costs 25,000 to 35,000 credits through product pages, or roughly 1,100 to 1,500 credits through search listings. Which of those is cheaper depends on your plan's per-credit rate, so run the credit figure against the current plan price rather than comparing headline numbers.

The honest split: if Target is your only data source and you want rows today, a dedicated scraper at $1.50 per 1,000 is hard to beat on time-to-value. If you scrape several retailers, want one billing relationship, or need control over which fields get parsed and how often, the general API wins on flexibility.

## FAQ

**Is there an official Target API?** No public one. Target's developer portal is open to partners and sellers, and every Target scraper on the market reads public Target.com pages instead. None of them are affiliated with or endorsed by Target.

**Is Target Australia the same site?** No. Target Australia runs on a separate domain, prices in AUD, lists 24,000+ products across 120+ stores, and returns store-level stock counts that the US site does not publish. A scraper built for one will not work on the other.

**Why does a parent TCIN return no stock?** Because stock lives at the variant level. A parent TCIN covers several colors or sizes, each with its own TCIN, and Target only publishes the lowest price at the parent level. Pull the child TCINs to get per-variant availability.

**Why can't I get more than 1,200 products for a keyword?** That is Target's own display cap for a single search term or category, not a scraper limit. Split the query into narrower terms or subcategories and merge the results by TCIN.

## Which one to buy

If you need finished Target rows this week and Target is your only source, buy a dedicated Target scraper. At roughly $1.50 per 1,000 products with parsing included, building your own pipeline costs more in engineering hours than the data costs in subscription fees.

If you already run scrapers against other sites, need custom fields, or want Target prices alongside Walmart and Amazon on one invoice, take the general API and start on the trial. Five thousand credits covers 140 to 250 rendered Target product pages, or several thousand products if you test the search-page route first. That is enough to measure your real credit burn before choosing a plan, and enough to confirm that US residential IPs plus rendering actually clear Akamai on the URLs you care about.

Two things to confirm before you pay: run your own Target URLs through the Domain Cost Estimator so the 25 to 35 credit estimate becomes an exact figure for your pages, and set `max_cost` per request so a single expensive URL cannot flatten the month's allowance. 👉 [Compare ScraperAPI plans and start the 7-day trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
