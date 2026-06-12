# EstiBot API

The EstiBot API lets you query the EstiBot system programmatically and receive
domain name information as normalized JSON.

> **Base URL:** `https://public-api.estibot.com/api`
>
> All examples in this document use this endpoint. Every request is a standard
> HTTP `GET` (parameters in the query string) and every response is a JSON
> object.

---

## Table of Contents

- [Access & Licensing](#access--licensing)
- [Authentication](#authentication)
- [Request Format](#request-format)
- [Response Format](#response-format)
- [Rate Limits](#rate-limits)
- [Errors](#errors)
- [Live vs. Cache Queries](#live-vs-cache-queries)
- [API Calls](#api-calls)
  - [appraise — Domain Appraisal](#appraise--domain-appraisal)
  - [lead — Lead Generator](#lead--lead-generator)
  - [drops — Drop Lists](#drops--drop-lists)
  - [zone_diff — Zone File Differences](#zone_diff--zone-file-differences)
  - [bid_tool — Search Volume / CPC](#bid_tool--search-volume--cpc)
  - [site_info — Web Site Info](#site_info--web-site-info)
- [Related Services](#related-services)

---

## Access & Licensing

- The API is available to higher level accounts such as **Advanced** or
  **Expert** customers only.
- Queries made through the API are treated exactly like queries run on the
  website and **increment your query counters** the same way.
- Data provided by EstiBot **cannot be republished, resold, or redistributed**
  without a republishing license from EstiBot.
- The API is offered for **personal and internal use only** and should not be
  used by organizations with more than 3 employees. If you plan to use the API
  within an organization, obtain an
  [enterprise license](https://www.estibot.com/contact).

---

## Authentication

Every request must include your API key in the `k` parameter:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=appraise&d=test.com
```

Replace `YOUR_API_KEY` with the key from your EstiBot account. Requests missing
a valid key are rejected (see [Errors](#errors)).

---

## Request Format

A valid request has the following components.

### Required parameters

| Parameter | Description |
|-----------|-------------|
| `k` | Your API key. |
| `a` | The tool you are calling — one of `appraise`, `lead`, `drops`, `zone_diff`, `bid_tool`, `site_info`. |
| `d` | The data to process, usually a domain or keyword. Multiple items are delimited with `>>` — for example `test.com>>fun.com>>cars.com`. |

### Optional parameters

| Parameter | Applies to | Description |
|-----------|------------|-------------|
| `t` | `appraise` (and others, where noted) | Query type: `live`, `cache`, or `auto`. For fastest responses use `cache`. Default is `auto`. |

Individual calls accept additional parameters — these are documented per call
in [API Calls](#api-calls).

**Example** — appraise `test.com`:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=appraise&d=test.com
```

---

## Response Format

Every response is a single JSON object with a consistent envelope.

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | `true` if the query succeeded, `false` if it failed. |
| `message` | string | Error message when the query failed; empty/blank otherwise. |
| `results` | array | Array of result objects. Each object's fields depend on the call. |
| `cache` | boolean | Whether the result was served from cache. |
| `bulk` | boolean | Whether this was a bulk query (more than one item). |
| `item_count` | integer | Number of items returned in `results`. |
| `result_count` | integer / null | Number of results (may be `null` for some calls). |
| `not_found` | array | Items not found in cache. Populated when `t=cache` and one or more requested items were not in the cache database. |

> **Note on field locations:** in the live JSON, `results` is the array of
> result objects, and `cache`, `bulk`, `item_count`, `result_count`, and
> `not_found` are **top-level** fields alongside `results` (not nested inside
> it). Parse against the sample below rather than assuming a nested shape.

<details>
<summary><strong>Sample response — appraise <code>test.com</code> (click to expand)</strong></summary>

```json
{
  "success": true,
  "message": "",
  "results": [{
    "id": "1195959",
    "domain": "test.com",
    "sld": "test",
    "tld": "com",
    "words": "test",
    "appraised_value": "574368",
    "appraised_wholesale_value": "34124",
    "appraised_monetization_value": "0",
    "appraised_ignore_tm_value": "574368",
    "appraised_no_sales_value": "573368",
    "appraisal_data_source_id": "1",
    "price_range_retail": "22",
    "price_range_no_tm": "22",
    "is_cctld": "0",
    "is_ntld": "0",
    "is_adult": "0",
    "is_reversed": "0",
    "language": "en",
    "language_probability": "100.00",
    "category": "Education",
    "category_root": "Education",
    "classification_id": "0",
    "first_word": "test",
    "second_word": "",
    "num_words": "1",
    "num_hyphens": "0",
    "num_numbers": "0",
    "sld_length": "4",
    "com_taken": "1",
    "net_taken": "1",
    "org_taken": "1",
    "biz_taken": "1",
    "us_taken": "1",
    "info_taken": "1",
    "extensions_taken": "6",
    "sld_extensions_taken": null,
    "search_results_phrase": "100000000",
    "search_ads_phrase": "2",
    "search_results_sld": "100000000",
    "search_ads_sld": "2",
    "search_results_reverse_phrase": "-1",
    "serp_source_phrase": "2",
    "serp_source_sld": "-1",
    "keyword_locale": "1:1",
    "keyword_exact_keyword": "[test]",
    "keyword_exact_competition": "100",
    "keyword_exact_cpc": "0.69",
    "keyword_exact_global_search_volume": "9140000",
    "keyword_exact_local_search_volume": "2740000",
    "keyword_exact_month": "5",
    "keyword_broad_keyword": "test",
    "keyword_broad_competition": "100",
    "keyword_broad_cpc": "0.69",
    "keyword_broad_global_search_volume": "5000000",
    "keyword_broad_local_search_volume": "45500000",
    "keyword_broad_month": "4",
    "keyword_ng_locale": "1:1",
    "keyword_ng_exact_keyword": "[test]",
    "keyword_ng_exact_competition": "1",
    "keyword_ng_exact_cpc": "2.51",
    "keyword_ng_exact_global_search_volume": "4090000",
    "keyword_ng_exact_local_search_volume": "550000",
    "keyword_ng_exact_month": "5",
    "keyword_ng_broad_keyword": "test",
    "keyword_ng_broad_competition": "20",
    "keyword_ng_broad_cpc": "1.97",
    "keyword_ng_broad_global_search_volume": "226000000",
    "keyword_ng_broad_local_search_volume": "45500000",
    "keyword_ng_broad_month": "3",
    "sld_ng_exact_keyword": "[test]",
    "sld_ng_exact_competition": "1",
    "sld_ng_exact_cpc": "2.51",
    "sld_ng_exact_global_search_volume": "4090000",
    "sld_ng_exact_local_search_volume": "550000",
    "sld_ng_broad_keyword": "test",
    "sld_ng_broad_competition": "20",
    "sld_ng_broad_cpc": "1.97",
    "sld_ng_broad_global_search_volume": "226000000",
    "sld_ng_broad_local_search_volume": "45500000",
    "tld_ng_exact_keyword": "[test.com]",
    "tld_ng_exact_competition": "35",
    "tld_ng_exact_cpc": "1.44",
    "tld_ng_exact_global_search_volume": "3600",
    "tld_ng_exact_local_search_volume": "1000",
    "keyword_root_ntld_ng_keyword": null,
    "keyword_root_ntld_ng_competition": "-1",
    "keyword_root_ntld_ng_cpc": "-1.00",
    "keyword_root_ntld_ng_global_search_volume": "-1",
    "keyword_root_ntld_ng_local_search_volume": "-1",
    "keyword_root_ntld_og_keyword": null,
    "keyword_root_ntld_og_competition": "-1",
    "keyword_root_ntld_og_cpc": "-1.00",
    "keyword_root_ntld_og_global_search_volume": "-1",
    "keyword_root_ntld_og_local_search_volume": "-1",
    "keyword_exact_ntld_ng_keyword": null,
    "keyword_exact_ntld_ng_competition": "-1",
    "keyword_exact_ntld_ng_cpc": "-1.00",
    "keyword_exact_ntld_ng_global_search_volume": "-1",
    "keyword_exact_ntld_ng_local_search_volume": "-1",
    "whois_create_date": "1997-06-18",
    "whois_expire_date": "2019-06-17",
    "whois_update_date": "2017-04-18",
    "whois_registrar": "NETWORK SOLUTIONS",
    "whois_registrar_iana": "2",
    "whois_reg_name": "PRIVATE",
    "whois_reg_org": "PRIVATE",
    "whois_reg_email": "DV5TV6EE2Z7@NETWORKSOLUTIONSPRIVATEREGISTRATION.COM",
    "whois_reg_email_count": null,
    "whois_is_private": "1",
    "alexa_traffic_rank": "42796",
    "alexa_reach_3m": "-1",
    "alexa_link_popularity": "0",
    "overture_tld": "1093",
    "overture_sld": "126711",
    "overture_term": "126711",
    "word_tracker_sld": "7315",
    "word_tracker_term": "7315",
    "pagerank": "0",
    "pagerank_real": "-1",
    "has_letters": "1",
    "has_numbers": "0",
    "has_only_letters": "1",
    "has_only_numbers": "0",
    "has_trademark": "0",
    "has_udrp_wipo": "0",
    "has_bad_ending": "0",
    "trademark_type": "",
    "trademark_term": "",
    "trademark_company": "",
    "trademark_risk": "0",
    "trademark_probability": "0",
    "trademark_domains": "0",
    "trademark_penalized": "0",
    "monetization_hits": "-1",
    "monetization_clicks": "-1",
    "monetization_searches": "-1",
    "monetization_revenue": "-1.000",
    "monetization_days_parked": "-1",
    "monetization_daily_avg_hits": "-1.000",
    "monetization_daily_avg_revenue": "-1.000",
    "web_pages": "1",
    "web_links_internal": "0",
    "web_links_external": "0",
    "web_links_external_unique": "0",
    "site_language": "en",
    "site_status": "1",
    "backlinks": "-1",
    "backlinks_unique": "-1",
    "backlink_ips": "0",
    "backlink_ips_unique": "-1",
    "backlink_subnets": "0",
    "backlink_subnets_unique": "-1",
    "altavista_link_popularity": "29400",
    "google_link_popularity": "530",
    "yahoo_link_popularity": "71420",
    "altavista_link_saturation": "-1",
    "google_link_saturation": "-1",
    "yahoo_link_saturation": "-1",
    "traffic_estimate": "0",
    "traffic_estimate_source": "metrics",
    "parking_revenue_estimate": "0.00",
    "development_revenue_estimate": "0.00",
    "end_user_buyers": "-1",
    "wayback_age": "0",
    "wayback_records": "0",
    "word_choppyness": "0",
    "unique_characters": "3",
    "unique_letters": "3",
    "words_rhyme": "0",
    "words_flow": "0",
    "pronounceability_score": "0",
    "using_sld": "0",
    "dmoz_listed": "1",
    "has_dns": "1",
    "pending_reappraisal": "0",
    "add_date": "2017-02-07 03:20:59",
    "appraisal_date": "2017-10-02 21:19:12",
    "appraiser_version": "2.5.9"
  }],
  "cache": true,
  "bulk": false,
  "item_count": 1,
  "not_found": [],
  "result_count": null
}
```

The `appraise` call returns roughly 100 data points per domain. The fields
above are illustrative; treat any value of `-1` (or `-1.00`) as "not available."

</details>

---

## Rate Limits

The API endpoint applies per-client-IP rate limiting to protect the service.
Current limits are:

| Limit | Scope | Notes |
|-------|-------|-------|
| **3 requests / second** | per IP | Short bursts above this are absorbed up to a small allowance, then throttled. |
| **100 requests / minute** | per IP | Volume ceiling per client IP. |

When a limit is exceeded, the endpoint returns HTTP **429** with the standard
JSON error envelope (see [Errors](#errors)). Clients should watch for `429`
responses and back off / retry rather than continuing to hammer the endpoint.

In addition to these endpoint-level limits, **individual API calls have their
own restrictions** (maximum domains per request, daily download caps, single
in-flight query rules, etc.). Those are documented under each call in
[API Calls](#api-calls).

> Limits are subject to change. Build clients to handle `429` and other error
> responses gracefully rather than assuming a fixed ceiling.

---

## Errors

When a request fails, `success` is `false` and `message` describes the problem.
The envelope is still returned so clients can parse failures the same way as
successes. **Always check `success` before reading `results`.**

Common error responses:

| HTTP status | `message` | Cause |
|-------------|-----------|-------|
| `400` | `Invalid API key.` | The `k` parameter is missing or malformed. |
| `400` | `Invalid or missing tool (a).` | The `a` parameter is missing or is not a supported tool. |
| `429` | `Rate limit exceeded.` | Too many requests from your IP (see [Rate Limits](#rate-limits)). |

Example error response (invalid key):

```json
{"success":false,"message":"Invalid API key.","redirect_url":"","results":{"total":0,"count":0,"start":0,"end":0,"data":[]}}
```

Example error response (rate limit exceeded):

```json
{"success":false,"message":"Rate limit exceeded.","redirect_url":"","results":{"total":0,"count":0,"start":0,"end":0,"data":[]}}
```

> The origin may also return its own `success:false` messages for conditions
> beyond the above (e.g. account-level restrictions or per-call limits). Treat
> any response with `success:false` as a failure and read `message` for detail.

---

## Live vs. Cache Queries

Some calls (notably [`appraise`](#appraise--domain-appraisal)) let you choose
between a live lookup and a cached lookup via the `t` parameter.

- **`live`** — forces a refresh of every data point and performs a new
  appraisal. Thorough but slow: up to ~3 seconds per domain. A batch of 200
  domains can take ~5 minutes.
- **`cache`** — returns the most recent cached appraisal from the database,
  usually in under 150 ms. The same 200-domain batch can complete in ~2 seconds.
- **`auto`** (default) — uses cached data if it exists and is recent (less than
  30 days old), otherwise falls back to live. **Single-domain lookups only.**

**Recommended pattern for batches:** query everything in `cache` mode first,
then re-query any domains returned in `not_found` using `live` mode.

---

## API Calls

### appraise — Domain Appraisal

Performs a domain appraisal and returns ~100 data points per domain.

**Restrictions**

- Maximum **200 domains** per submission. Delimit batches with `>>` —
  e.g. `d=test.com>>great.com`.
- **Auto mode works only with single-domain lookups.** For batches, use
  `t=cache` or `t=live`.
- **One query at a time.** Wait for a query to complete before sending another.

**Examples**

Appraise `test.com` using `auto` mode (cached data if available and less than
30 days old, otherwise live):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=appraise&d=test.com
```

Appraise `test.com` using `cache` mode (returns cached appraisal; if the domain
is not cached it is not returned):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=appraise&t=cache&d=test.com
```

Appraise `test.com` using `live` mode (forces a full live appraisal; slower
because every data point is refreshed):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=appraise&t=live&d=test.com
```

Appraise `test.com` and `great.com` in `cache` mode:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=appraise&t=cache&d=test.com>>great.com
```

---

### lead — Lead Generator

Generates a set of end-user leads for a domain or keyword and returns the data
along with analytics.

**Restrictions**

- Maximum **1 domain or keyword set** per submission.
- **One query at a time.**
- **Be patient** — lead generation is resource intensive; allow up to 60
  seconds per query.

**Examples**

Generate leads for `estibot.com`:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=lead&d=estibot.com
```

Generate leads for the keyword `fast cars` and domain `fastcars.com` (both must
be specified):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=lead&keyword=fast+cars&d=fastcars.com
```

---

### drops — Drop Lists

Downloads drop lists programmatically. The response is always a JSON object
containing a URL to a compressed CSV file generated by the system.

**Restrictions**

- Maximum **20 API downloads per day.**
- List date cannot be before today or more than 10 days old.
- **One query at a time.**
- **Be patient** — CSV files are generated in real time and may take up to 10
  seconds to process.

**Parameters**

| Parameter | Description |
|-----------|-------------|
| `t` | List type — e.g. `raw_pendingdelete`, `appraisals`, `complete`. |
| `list` | Specific list — e.g. `available`, `namejet pre-release`, `pendingdelete`. |
| `date` | List date in `YYYY-MM-DD` format. |

**Examples**

Get the most recent `raw_pendingdelete` file:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=drops&t=raw_pendingdelete
```

Get the AVAILABLE domain list (appraisals only) from a specific date:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=drops&t=appraisals&list=available&date=2026-06-10
```

Get the AVAILABLE domain list (complete with analytics) from a specific date:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=drops&t=complete&list=available&date=2026-06-10
```

Get the NAMEJET PRE-RELEASE domain list (complete with analytics):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=drops&t=complete&list=namejet+pre-release&date=2026-06-10
```

Get the PENDINGDELETE domain list (complete with analytics):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=drops&t=complete&list=pendingdelete&date=2026-06-10
```

---

### zone_diff — Zone File Differences

Downloads zone file differences — newly added domains and newly expired domains
— programmatically. The response is always a JSON object containing a URL to a
compressed CSV file (ZIP format).

**Restrictions**

- Maximum **25 API downloads per day.**
- **One query at a time.**
- **Be patient** — CSV files are generated in real time and may take up to 5
  seconds to process.
- **Files expire** — on-demand temporary files expire after 1 hour.

**Parameters**

| Parameter | Description |
|-----------|-------------|
| `type` | `new` (newly registered domains) or `deleted` (newly expired domains). |
| `format` | Optional. Set to `direct` to be forwarded straight to the file instead of receiving the JSON wrapper. |

**Examples**

Get the most recent `new` zone change (newly registered domains):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=zone_diff&type=new
```

Get the most recent `deleted` zone change (newly expired domains):

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=zone_diff&type=deleted
```

Get the most recent `new` zone change and be forwarded directly to the file:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=zone_diff&type=new&format=direct
```

---

### bid_tool — Search Volume / CPC

Returns search volume and CPC data for keywords. The response is always a JSON
object.

**Restrictions**

- Maximum **3 concurrent threads.**
- You must provide one or more **keywords** (e.g. `car+insurance` for
  "car insurance"). This call does **not** parse domains into keywords.
- **No data republishing, reselling, or sharing.** Personal use only.

**Examples**

Get search volume and CPC for `car insurance`:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=bid_tool&d=car+insurance
```

Get search volume and CPC for `car insurance` and `fast cars`:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=bid_tool&d=car+insurance>>fast+cars
```

---

### site_info — Web Site Info

Returns a web site crawl summary: domain usage status (developed, parked,
redirected, etc.), plus title, emails, phone numbers, MX servers, and more.
Data comes from EstiBot's crawl feed, which continuously crawls all registered
domains, so responses are very fast. The response is always a JSON object.

**Restrictions**

- Maximum **3 concurrent threads.**
- Check up to **1,000 domains per second** (Expert-level accounts only).
- You must provide one or more **domains.**
- **No data republishing, reselling, or sharing.** Personal use only.

**Examples**

Get web site information for `estibot.com`:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=site_info&d=estibot.com
```

Get web site information for `estibot.com` and `great.com`:

```
https://public-api.estibot.com/api?k=YOUR_API_KEY&a=site_info&d=estibot.com>>great.com
```

---

## Related Services

Need more data — whois, word parsing, trademark analysis, or 100+ additional API
calls? See EstiBot's sister sites:

- **[DomainPlex](https://www.domainplex.com)** — enterprise API services.
- **[domainIQ.com](https://www.domainiq.com)** — whois API services.

---

*Code samples (PHP, Java, and more) will be added to this repository over time.*
