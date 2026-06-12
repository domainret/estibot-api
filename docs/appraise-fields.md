# Appraisal Tool — Field Reference

This page describes every field returned by the [`appraise`](../README.md#appraise--domain-appraisal) tool inside each entry of `results.data[]`. Use the main README for overall request/response shape and this page for per-field meaning.

The cache and live response shapes overlap but are not identical:

- **Cache mode** (`t=cache`, or `auto` when a cached row exists) returns a flat set of fields.
- **Live mode** (`t=live`, or `auto` with no cached row) returns most of the same data with several values folded into **nested objects** (`whois`, `web`, `trademark`, `keyword_stats`, …), plus extras that exist only in live mode (`stock`, `sales_history`, `trending`, `language_code`, …).

**Conventions used below:**

- **Sentinel values.** `-1` (or `-1.00`) in a numeric field means *not available*. `""` or `null` in a string field means *unknown*. Treat these as missing, not zero.
- **🟡 TBD** prefixes mark descriptions that are still being finalized — wording may change.
- Inline types in parentheses (e.g. `int unsigned`, `varchar(150)`, `decimal(10,2)`) come from the underlying schema and constrain what the API will return.

---

## Cache-mode fields


### Identifiers & domain parsing

| Field | Description |
|---|---|
| `id` | Internal numeric record ID for this cached appraisal row (int unsigned, auto-increment). |
| `domain` | The domain name that was appraised (varchar(150) ASCII). |
| `domain_cc` | Camel-case version of the domain — first letter of each parsed word capitalized (e.g. fastcars.com -> FastCars.com). Decorated at the API layer; not a stored column. |
| `sld` | Single level domain — the part before the dot (varchar(150) ASCII; e.g. fastcars.com -> fastcars). |
| `tld` | Top level domain — the part after the dot, a.k.a. domain extension (varchar(45) ASCII; e.g. fastcars.com -> com). |
| `words` | Words parsed from within the domain name, a.k.a. the domain term (varchar(150) utf8; e.g. fastcars.com -> "fast cars"). |
| `first_word` | First parsed word of the domain (varchar(20) utf8). Empty string if not applicable. |
| `second_word` | Second parsed word of the domain (varchar(20) utf8, nullable). Empty/null for single-word domains. |
| `num_words` | Count of words parsed from the SLD (tinyint unsigned). |
| `num_hyphens` | Count of hyphen characters in the SLD (tinyint unsigned). |
| `num_numbers` | Count of digit characters in the SLD (tinyint unsigned). |
| `sld_length` | Character length of the SLD (tinyint unsigned). |


### Language & category

| Field | Description |
|---|---|
| `language` | Language of the parsed words. CACHE MODE returns a 2-letter ISO code (varchar(2), default 'un' for unknown; e.g. 'en'). LIVE MODE returns the spelled-out name (e.g. 'english') and provides the 2-letter code separately in `language_code`. |
| `language_probability` | 0-100 confidence score for the language detection, expressed as decimal(5,2). |
| `category` | Category/vertical associated with this domain (varchar(75)). |
| `category_root` | Top-level (root) category that `category` rolls up to (varchar(45)). |
| `classification_id` | 🟡 **TBD** — Numeric ID of the classification (int unsigned). Mapping TBD. |


### Appraised values

| Field | Description |
|---|---|
| `appraised_value` | EstiBot retail appraisal — estimated value when sold to an end-user. USD integer (int unsigned). |
| `appraised_wholesale_value` | EstiBot wholesale appraisal — estimated value when sold to another domain investor. USD integer. |
| `appraised_monetization_value` | Appraisal based on monetization (parking) value. USD integer; 0 if not monetizable or no parking history. |
| `appraised_ignore_tm_value` | Appraisal calculated as if there were no trademark conflict. USD integer. Compare to `appraised_value` to see the trademark penalty. |
| `appraisal_data_source_id` | 🟡 **TBD** — Numeric ID identifying which data source/algorithm produced the appraisal (int unsigned). Mapping TBD. |
| `price_range_retail` | Bracket ID (tinyint unsigned, 1-24) that `appraised_value` falls into. See [price range brackets](#price_range_retail--price_range_no_tm-brackets) below. Default 0 means no bracket assigned. |
| `price_range_no_tm` | Bracket ID (tinyint unsigned, 1-24) that `appraised_ignore_tm_value` falls into. Same lookup as `price_range_retail`. See [price range brackets](#price_range_retail--price_range_no_tm-brackets) below. |


### Domain characteristics / flags

| Field | Description |
|---|---|
| `is_cctld` | Boolean (tinyint, 0/1). Whether the TLD is a country code TLD (e.g. .uk, .de). |
| `is_ntld` | Boolean (tinyint, 0/1). Whether the TLD is a new top-level domain (e.g. .guru, .link). |
| `is_adult` | Boolean (tinyint, 0/1). Whether the domain is adult in nature. |
| `is_reversed` | 🟡 **TBD** — Boolean (tinyint, 0/1). Whether the parsed words appear reversed in the SLD (e.g. carsfast vs fastcars). |
| `has_letters` | Boolean (int unsigned, 0/1). Whether the SLD contains any alphabetic characters. |
| `has_numbers` | Boolean (int unsigned, 0/1). Whether the SLD contains any digit characters. |
| `has_only_letters` | Boolean (int unsigned, 0/1). Whether the SLD consists entirely of alphabetic characters. |
| `has_only_numbers` | Boolean (int unsigned, 0/1). Whether the SLD consists entirely of digit characters. |
| `has_bad_ending` | 🟡 **TBD** — Boolean (int unsigned, 0/1). Whether the SLD ends with a character pattern that hurts value (specific rules TBD). |
| `has_dns` | Whether this exact domain has an entry in the zone file (i.e. is registered). Values: 1=yes, 0=no, -1=unknown. |
| `dmoz_listed` | Boolean (tinyint, 0/1, nullable). Whether the domain is listed in the DMOZ database. DMOZ shut down in 2017 so non-zero values reflect legacy data. |


### TLD availability matrix

| Field | Description |
|---|---|
| `com_taken` | Boolean. Whether the .com version of this SLD is registered (tinyint unsigned, 0/1). |
| `net_taken` | Boolean. Whether the .net version of this SLD is registered (tinyint unsigned, 0/1). |
| `org_taken` | Boolean. Whether the .org version of this SLD is registered (tinyint unsigned, 0/1). |
| `biz_taken` | Boolean. Whether the .biz version of this SLD is registered (tinyint unsigned, 0/1). |
| `us_taken` | Boolean. Whether the .us version of this SLD is registered (tinyint unsigned, 0/1). |
| `info_taken` | Boolean. Whether the .info version of this SLD is registered (tinyint unsigned, 0/1). |
| `extensions_taken` | Count of gTLD extensions registered for this SLD (tinyint unsigned). E.g. fastcars.com -> fastcars.{net,org,info,biz,us,com} -> 6. |
| `sld_extensions_taken` | Number of SLD extensions taken for all (smallint, nullable). |
| `tld_count` | 🟡 **TBD** — Total count of TLDs across which this SLD is registered (broader than `extensions_taken`, which is gTLD-only). Computed at the API layer; not a schema column. |
| `tld_count_dev` | 🟡 **TBD** — Count of those registrations that appear to host a developed site (not parked/empty). Computed at the API layer; not a schema column. |


### Google search results & ads

| Field | Description |
|---|---|
| `search_results_phrase` | Number of Google search results for the parsed words in quotes (e.g. "fast cars"). Also known as SERP (search engine results phrase). bigint. |
| `search_ads_phrase` | Number of Google ads for the parsed words in quotes (e.g. "fast cars"). Also known as ad SERP. int. |
| `search_results_sld` | Number of Google search results for the SLD portion of the domain in quotes (e.g. "fastcars"). No space. bigint. |
| `search_ads_sld` | Number of Google ads for the SLD portion of the domain in quotes (e.g. "fastcars"). No space. int. |
| `search_results_reverse_phrase` | 🟡 **TBD** — Number of Google search results when the parsed words are reversed in quotes (e.g. "cars fast"). bigint; -1 if not computed. |
| `serp_source_phrase` | 🟡 **TBD** — Code identifying which engine/source produced `search_results_phrase` (tinyint). Mapping TBD. |
| `serp_source_sld` | 🟡 **TBD** — Code identifying which engine/source produced `search_results_sld` (tinyint). Mapping TBD. |


### Keyword metrics — archive Google (2013/2014 dataset)

> Field family: keyword_exact_*  — exact-match lookup using parsed words, e.g. [fast cars] Field family: keyword_broad_*  — broad-match lookup using parsed words, e.g. fast cars

| Field | Description |
|---|---|
| `keyword_locale` | 🟡 **TBD** — Locale of the archive-Google ("og") dataset (varchar(8), default ''). Observed format like '1:1' suggests an internal country_id:language_id pair; verify mapping. |
| `keyword_exact_keyword` | The exact keyword analyzed — parsed words from the domain enclosed in brackets (e.g. [fast cars]). varchar(150), nullable. |
| `keyword_exact_competition` | Average ad competition for the exact keyword (archive Google data; int, expected 0-100 scale). |
| `keyword_exact_cpc` | Average cost-per-click for the exact keyword (archive Google data; decimal(10,2), USD). |
| `keyword_exact_global_search_volume` | Monthly global searches for the exact keyword (archive Google data including partner sites; bigint). |
| `keyword_exact_local_search_volume` | Monthly searches for the exact keyword restricted to the `keyword_locale` (archive Google data; bigint). |
| `keyword_exact_month` | Month of the year (1-12) with the peak search volume for the exact keyword (archive Google data). |
| `keyword_broad_keyword` | The broad keyword analyzed — parsed words from the domain, no brackets (varchar(150), nullable). |
| `keyword_broad_competition` | Average ad competition for the broad keyword (archive Google data; int, expected 0-100 scale). |
| `keyword_broad_cpc` | Average cost-per-click for the broad keyword (archive Google data; decimal(10,2), USD). |
| `keyword_broad_global_search_volume` | Monthly global searches for the broad keyword (archive Google data including partner sites; bigint). |
| `keyword_broad_local_search_volume` | Monthly searches for the broad keyword restricted to the `keyword_locale` (archive Google data; bigint). |
| `keyword_broad_month` | Month of the year (1-12) with the peak search volume for the broad keyword (archive Google data). |


### Keyword metrics — new Google (current dataset)

> Field family: keyword_ng_*  — same as keyword_* above but from the new Google search tool

| Field | Description |
|---|---|
| `keyword_ng_locale` | Locale of the new Google dataset (varchar(8), default ''). Schema comment lists country codes as examples (e.g. 'US', 'UK', 'DE'); observed values may instead use an internal ID-pair like '1:1' — verify which format the API emits. |
| `keyword_ng_exact_keyword` | The exact keyword analyzed against the new Google dataset — parsed words in brackets (varchar(150), nullable). |
| `keyword_ng_exact_competition` | Ad competition for the exact keyword (new Google data; int, expected 0-100 scale). |
| `keyword_ng_exact_cpc` | Average CPC for the exact keyword (new Google data; decimal(10,2), USD). |
| `keyword_ng_exact_global_search_volume` | Monthly global searches for the exact keyword (new Google data; bigint). |
| `keyword_ng_exact_local_search_volume` | Monthly searches for the exact keyword restricted to `keyword_ng_locale` (new Google data; bigint). |
| `keyword_ng_exact_month` | Month of the year (1-12) with the peak search volume for the new-Google exact keyword. |
| `keyword_ng_broad_keyword` | The broad keyword analyzed against the new Google dataset (varchar(150), nullable). |
| `keyword_ng_broad_competition` | Ad competition for the broad keyword (new Google data; int, expected 0-100 scale). |
| `keyword_ng_broad_cpc` | Average CPC for the broad keyword (new Google data; decimal(10,2), USD). |
| `keyword_ng_broad_global_search_volume` | Monthly global searches for the broad keyword (new Google data; bigint). |
| `keyword_ng_broad_local_search_volume` | Monthly searches for the broad keyword restricted to `keyword_ng_locale` (new Google data; bigint). |
| `keyword_ng_broad_month` | Month of the year (1-12) with the peak search volume for the new-Google broad keyword. |


### Keyword metrics — SLD-level (new Google, no space)

> Looks up the SLD as a single token (e.g. [fastcars] / fastcars)

| Field | Description |
|---|---|
| `sld_ng_exact_keyword` | The exact keyword analyzed using the SLD with no spaces, in brackets (new Google data). varchar(150) nullable. |
| `sld_ng_exact_competition` | Ad competition for the SLD-as-keyword exact lookup (new Google data; int, expected 0-100 scale). |
| `sld_ng_exact_cpc` | Average CPC for the SLD-as-keyword exact lookup (new Google data; decimal(10,2), USD). |
| `sld_ng_exact_global_search_volume` | Monthly global searches for the SLD-as-keyword exact lookup (new Google data; bigint). |
| `sld_ng_exact_local_search_volume` | Monthly local-locale searches for the SLD-as-keyword exact lookup (new Google data; bigint). |
| `sld_ng_broad_keyword` | The broad keyword analyzed using the SLD with no spaces (new Google data). varchar(150) nullable. |
| `sld_ng_broad_competition` | Ad competition for the SLD-as-keyword broad lookup (new Google data; int, expected 0-100 scale). |
| `sld_ng_broad_cpc` | Average CPC for the SLD-as-keyword broad lookup (new Google data; decimal(10,2), USD). |
| `sld_ng_broad_global_search_volume` | Monthly global searches for the SLD-as-keyword broad lookup (new Google data; bigint). |
| `sld_ng_broad_local_search_volume` | Monthly local-locale searches for the SLD-as-keyword broad lookup (new Google data; bigint). |


### Keyword metrics — TLD-level (new Google)

> Looks up the TLD itself as a keyword (rarely populated; mostly -1)

| Field | Description |
|---|---|
| `tld_ng_exact_keyword` | The exact keyword for the TLD itself (new Google data). varchar(150) nullable; typically null for legacy TLDs. |
| `tld_ng_exact_competition` | Ad competition for the TLD-only exact lookup (int); -1 if not populated. |
| `tld_ng_exact_cpc` | Average CPC for the TLD-only exact lookup (decimal(10,2)); -1 if not populated. |
| `tld_ng_exact_global_search_volume` | Monthly global searches for the TLD-only exact lookup (bigint); -1 if not populated. |
| `tld_ng_exact_local_search_volume` | Monthly local-locale searches for the TLD-only exact lookup (bigint); -1 if not populated. |


### Keyword metrics — nTLD root keyword (new & old Google)

> Applicable to nTLDs only. For nTLDs where words and extension combine (e.g. used.cars -> "used cars"), these fields isolate metrics for the root words without the extension. Schema confirms "og" = old Google.

| Field | Description |
|---|---|
| `keyword_root_ntld_ng_keyword` | Keyword for the root words only — no extension — new Google data. Applicable to nTLDs only (varchar(150), nullable). |
| `keyword_root_ntld_ng_competition` | Ad competition for the root-only nTLD lookup (new Google); nullable, -1 if not applicable. |
| `keyword_root_ntld_ng_cpc` | Average CPC for the root-only nTLD lookup (new Google; decimal(10,2)); nullable. |
| `keyword_root_ntld_ng_global_search_volume` | Monthly global searches for the root-only nTLD lookup (new Google); nullable. |
| `keyword_root_ntld_ng_local_search_volume` | Monthly local-locale searches for the root-only nTLD lookup (new Google); nullable. |
| `keyword_root_ntld_og_keyword` | Keyword for the root words only — no extension — old Google data. Applicable to nTLDs only (varchar(150), nullable). |
| `keyword_root_ntld_og_competition` | Ad competition for the root-only nTLD lookup (old Google); nullable, -1 if not applicable. |
| `keyword_root_ntld_og_cpc` | Average CPC for the root-only nTLD lookup (old Google; decimal(10,2)); nullable. |
| `keyword_root_ntld_og_global_search_volume` | Monthly global searches for the root-only nTLD lookup (old Google); nullable. |
| `keyword_root_ntld_og_local_search_volume` | Monthly local-locale searches for the root-only nTLD lookup (old Google); nullable. |


### Keyword metrics — nTLD full keyword incl. extension

> For nTLDs where the words + extension form a phrase, these fields contain metrics for the full combined phrase (e.g. used.cars -> [used cars] inclusive).

| Field | Description |
|---|---|
| `keyword_exact_ntld_ng_keyword` | Exact keyword combining words + extension (new Google data). varchar(150) nullable; applicable to nTLDs only. |
| `keyword_exact_ntld_ng_competition` | Ad competition for the combined-words-plus-extension lookup (new Google); nullable. |
| `keyword_exact_ntld_ng_cpc` | Average CPC for the combined-words-plus-extension lookup (new Google; decimal(10,2)); nullable. |
| `keyword_exact_ntld_ng_global_search_volume` | Monthly global searches for the combined-words-plus-extension lookup (new Google); nullable. |
| `keyword_exact_ntld_ng_local_search_volume` | Monthly local-locale searches for the combined-words-plus-extension lookup (new Google); nullable. |


### WHOIS

| Field | Description |
|---|---|
| `whois_create_date` | Whois creation date for the domain (date, YYYY-MM-DD; nullable). From the whois archive. |
| `whois_expire_date` | Whois expiration date for the domain (date, YYYY-MM-DD; nullable). From the whois archive. |
| `whois_update_date` | Whois last-update date for the domain (date, YYYY-MM-DD; nullable). From the whois archive. |
| `whois_registrar` | Whois registrar name for the domain (varchar(75), nullable). From the whois archive. |
| `whois_registrar_iana` | Whois registrar IANA number for the domain (varchar(25), nullable). Stored as a string, not necessarily numeric. |
| `whois_reg_name` | Whois registrant name for the domain (varchar(75), nullable). From the whois archive. |
| `whois_reg_org` | Whois registrant organization for the domain (varchar(75), nullable). From the whois archive. |
| `whois_reg_email` | Whois registrant email for the domain (varchar(150), nullable). From the whois archive. |
| `whois_reg_email_count` | Number of other domains EstiBot has on record under this registrant email (int, nullable) — portfolio-size signal. |
| `whois_is_private` | Boolean (tinyint, 0/1, nullable). Whether the registrant is protected by a privacy whois service. |


### Traffic & popularity

| Field | Description |
|---|---|
| `alexa_traffic_rank` | Alexa traffic rank — integer between 1 and ~50 million indicating popularity (1 = most popular). 0 indicates no popularity. (Alexa was retired in 2022; legacy data only.) |
| `alexa_reach_3m` | Alexa's 3-month reach estimate (double); -1 if not available. Legacy. |
| `alexa_link_popularity` | Alexa backlink count for the domain (int). Legacy. |
| `overture_tld` | Overture score for the domain name (int). Overture estimates organic (type-in) traffic. |
| `overture_sld` | Overture score for the SLD portion of the domain (int). |
| `overture_term` | Overture score for the words within the domain (int). |
| `word_tracker_sld` | WordTracker score for the SLD portion of the domain (int). |
| `word_tracker_term` | WordTracker score for the words within the domain (int). |
| `pagerank` | Google PageRank for the domain — 0-10 score (int). Google retired public PageRank in 2016; legacy data only. |
| `pagerank_real` | Real Google PageRank for the domain. Where `pagerank` was obtained from a redirected domain, this shows the real PageRank (int). |
| `traffic_estimate` | EstiBot's estimated monthly organic traffic for the domain (int). |
| `traffic_estimate_source` | Identifier of which method produced `traffic_estimate` (varchar(25); e.g. "metrics", "alexa"). |
| `parking_revenue_estimate` | Estimated monthly revenue if the domain were parked, in USD (decimal(10,2)). |
| `development_revenue_estimate` | Estimated monthly revenue if the domain were developed into a site, in USD (decimal(10,2)). |


### Link popularity & saturation (legacy)

| Field | Description |
|---|---|
| `altavista_link_popularity` | Backlink count per AltaVista, int (defunct). Typically -1. |
| `google_link_popularity` | Backlink count per Google, int. Typically -1 (Google no longer exposes this). |
| `yahoo_link_popularity` | Backlink count per Yahoo, int. Typically -1. |
| `altavista_link_saturation` | Indexed-page count per AltaVista, int (defunct). Typically -1. |
| `google_link_saturation` | Indexed-page count per Google, int. Typically -1. |
| `yahoo_link_saturation` | Indexed-page count per Yahoo, int. Typically -1. |


### Backlinks

| Field | Description |
|---|---|
| `backlinks` | Total backlink count from EstiBot's link graph (int); -1 if not computed. |
| `backlinks_unique` | Count of unique backlinks after deduping repeats from the same source page (int). |
| `backlink_ips` | Number of distinct source IPs that host backlinks to this domain (int). |
| `backlink_ips_unique` | Number of unique source IPs after deduping (int). |
| `backlink_subnets` | Number of distinct /24 subnets that host backlinks to this domain (int). |
| `backlink_subnets_unique` | Number of unique source /24 subnets after deduping (int). |


### Trademark

| Field | Description |
|---|---|
| `has_trademark` | Boolean (int unsigned, 0/1). Whether EstiBot has flagged any trademark match for this domain. |
| `has_udrp_wipo` | Boolean (int unsigned, 0/1). Whether the domain has a WIPO or UDRP case on record — legal cases that may affect value. |
| `trademark_type` | If the domain appears associated with a trademark, indicator of the class of trademark (varchar(15)). |
| `trademark_term` | If the domain appears associated with a trademark, the term that is trademarked (varchar(20)). |
| `trademark_company` | If the domain appears associated with a trademark, the company that owns the trademark (varchar(25)). |
| `trademark_risk` | 0-10 risk score (int) assigned by EstiBot based on the holder's enforcement history (e.g. Wikipedia typo would score low vs Microsoft or Verizon scoring high). Automated; not a legal opinion. |
| `trademark_probability` | 0-100 confidence score (int) in the trademark match. |
| `trademark_domains` | Count of other domains in EstiBot's database believed to relate to the same trademark term (int). |
| `trademark_penalized` | Boolean (tinyint, 0/1). Whether the appraised value was penalized due to the trademark match. Compare `appraised_value` vs `appraised_ignore_tm_value`. |


### Monetization (parking activity)

| Field | Description |
|---|---|
| `monetization_hits` | Total parking-page hits recorded for this domain (int); -1 if no monetization history. |
| `monetization_clicks` | Total parking-page ad clicks recorded (int); -1 if no history. |
| `monetization_searches` | Total parking-page search-box queries recorded (int); -1 if no history. |
| `monetization_revenue` | Total parking revenue in USD recorded (decimal(10,3), 3 decimal places); -1 if no history. |
| `monetization_days_parked` | Number of days the domain has been parked in EstiBot's tracking (int); -1 if not parked. |
| `monetization_daily_avg_hits` | Daily average parking-page hits over the parked period (decimal(10,3)); -1 if no history. |
| `monetization_daily_avg_revenue` | Daily average parking revenue in USD over the parked period (decimal(10,3)); -1 if no history. |


### Web & site

| Field | Description |
|---|---|
| `web_pages` | Number of pages found on the domain by EstiBot's crawler (int). |
| `web_links_internal` | Number of internal links observed across crawled pages (int); -1 if not crawled. |
| `web_links_external` | Number of external (outgoing) links observed (int); -1 if not crawled. |
| `web_links_external_unique` | Number of unique external link targets (int); -1 if not crawled. |
| `site_language` | Detected language of the live site (varchar(15)). May differ from `language` which is parsed from the SLD. |
| `site_status` | Numeric code describing site usage status (smallint, nullable). See [`site_status` enum](#site_status-partial) below. Note: values beyond the documented enum have been observed (e.g. 7); meaning unconfirmed. |


### Word & phonetic characteristics

| Field | Description |
|---|---|
| `words_rhyme` | Boolean (tinyint, 0/1). Whether the words within the domain rhyme. For nTLDs, also checks whether the extension rhymes with the words (e.g. blink.link -> 1). |
| `words_flow` | Boolean (tinyint, 0/1). Whether the words within the domain flow well in the English language. |
| `pronounceability_score` | 0-10 score indicating how easily the domain name can be pronounced (smallint, nullable). |
| `word_choppyness` | Score reflecting how disjointed the syllables/words sound when said together — higher = choppier (smallint unsigned). |
| `unique_characters` | Count of distinct characters used in the SLD (smallint unsigned). |
| `unique_letters` | Count of distinct alphabetic characters used in the SLD (smallint unsigned). |
| `using_sld` | Boolean (tinyint, 0/1, nullable). Whether the SLD (not the parsed words) was used to look up vital search metrics — used when the SLD has higher/better search data than its parsed words. |


### Wayback / archive

| Field | Description |
|---|---|
| `wayback_age` | Age in years of the oldest Wayback Machine snapshot for this domain (int); -1 if no snapshots. |
| `wayback_records` | Total number of Wayback Machine snapshots for this domain (int); -1 if no snapshots. |


### Lead & end-user data

| Field | Description |
|---|---|
| `end_user_buyers` | EstiBot approximation of how many potential end-user buyers exist for this domain (smallint); -1 if not computed. (See also `lead_score` returned by the `lead` tool.) |


### Metadata

| Field | Description |
|---|---|
| `appraiser_version` | Version string of the appraisal engine that generated this row (varchar(10)). |
| `cached` | Boolean (0/1). Whether this row was served from cache. Decorated at the API layer; mirrors the top-level `cache` envelope flag. |


---

## Live-mode-only fields

> These appear only when `t=live`. Many are nested objects that contain the same data as the flat fields above; some carry extra info not exposed in cache. These are NOT in the `appraise` SQL schema — they are computed/composed at request time.

| Field | Description |
|---|---|
| `sld_ntld` | The SLD with the extension concatenated (no space), only meaningful for nTLDs whose SLD reads as words when combined with the extension (e.g. buffet.reviews -> "buffetreviews"). For non-nTLDs, mirrors `sld`. |
| `language_code` | 2-letter ISO language code corresponding to the spelled-out `language` returned in live mode (e.g. "en" for English). |
| `udrp` | Boolean (0/1). Whether a UDRP case was detected for this domain. Live-mode equivalent of (and finer-grained than) `has_udrp_wipo`, which also covers WIPO cases. |
| `rhyme_extension` | Boolean (0/1). Whether the SLD rhymes with the extension (e.g. hive.live -> 1). Most useful for nTLDs and other word-like extensions. |
| `rhyme_pattern` | 🟡 **TBD** — String describing the rhyme structure used by EstiBot's phonetic analysis (format TBD). |
| `google_site_index` | 🟡 **TBD** — Number of pages indexed by Google for this domain (current data; complements the legacy `google_link_saturation`). |
| `search_results_tld` | 🟡 **TBD** — Number of Google search results for the TLD itself in quotes (rarely useful; mostly -1 for legacy TLDs). |
| `search_suggestion_sld` | 🟡 **TBD** — Google search-suggestion data for the SLD-as-keyword (format TBD). |
| `using_previous_sales_data` | 🟡 **TBD** — Boolean (0/1). Whether the appraisal incorporated prior sales-history data. |
| `char_pattern` | 🟡 **TBD** — Compact string encoding the character-class pattern of the SLD (e.g. CVCVC for consonant-vowel-consonant-vowel-consonant). |
| `char_pattern_basic` | 🟡 **TBD** — Simpler char-class pattern (e.g. LLNN for letters/numbers). |
| `whois_age` | 🟡 **TBD** — Age of the domain in years derived from `whois_create_date`. |
| `whois_name_count` | 🟡 **TBD** — Count of other domains we have on record under this registrant name (portfolio-size signal). |
| `trademark_sld_count` | 🟡 **TBD** — Count of domains EstiBot has flagged for the same trademark term as this SLD. |
| `keyword_broad_is_ng` | 🟡 **TBD** — Boolean (0/1). Whether `keyword_broad_*` values were sourced from the new Google ("ng") dataset rather than the archive. |
| `keyword_exact_is_ng` | 🟡 **TBD** — Boolean (0/1). Whether `keyword_exact_*` values were sourced from the new Google dataset. |
| `keyword_ng_broad_is_ng` | 🟡 **TBD** — Boolean (0/1). Tautological flag (always 1 in this family) indicating new-Google sourcing. |
| `keyword_ng_exact_is_ng` | 🟡 **TBD** — Boolean (0/1). Tautological flag (always 1 in this family) indicating new-Google sourcing. |
| `keyword_broad_type` | 🟡 **TBD** — Code indicating how the broad keyword was matched/selected (mapping TBD). |
| `keyword_exact_type` | 🟡 **TBD** — Code indicating how the exact keyword was matched/selected (mapping TBD). |
| `keyword_ng_broad_type` | 🟡 **TBD** — Code indicating how the new-Google broad keyword was matched/selected (mapping TBD). |
| `keyword_ng_exact_type` | 🟡 **TBD** — Code indicating how the new-Google exact keyword was matched/selected (mapping TBD). |
| `whois` | Nested object containing the full whois detail; the flat `whois_*` fields above are the convenience accessors. |
| `web` | Nested object containing the full crawl/web data; the flat `web_*` fields above are the convenience accessors. |
| `trademark` | Nested object containing the full trademark detail; the flat `trademark_*` fields above are the convenience accessors. |
| `keyword_stats` | Nested object containing archive-Google keyword data; the flat `keyword_exact_*`/`keyword_broad_*` fields are convenience accessors. |
| `keyword_stats_ng` | Nested object containing new-Google keyword data; the flat `keyword_ng_*`/`sld_ng_*`/`tld_ng_*` fields are convenience accessors. |
| `keyword_stats_root_ng` | Nested object containing nTLD root-only keyword data; the flat `keyword_root_ntld_*` fields are convenience accessors. |
| `keyword_stats_exact_ntld_ng` | Nested object containing nTLD-combined keyword data; the flat `keyword_exact_ntld_ng_*` fields are convenience accessors. |
| `sales_history` | 🟡 **TBD** — Array of prior public sales for this exact domain (date, price, venue). Not present in cache mode. |
| `similar_sales` | 🟡 **TBD** — Array of comparable sales used as appraisal references. Not present in cache mode. |
| `stock` | Stock-market lookup tied to the SLD. Nested object with `stock_ticker`, `name`, `market_cap`. Only checked when the SLD is <= 8 characters and contains only letters (no digits or hyphens); otherwise (and when no matching ticker exists) all sub-fields are returned at their empty/zero defaults (`""` / `"0.0"`). |
| `avg_sales_price` | 🟡 **TBD** — Average sale price of comparable domains used in the appraisal model. |
| `trending` | 🟡 **TBD** — Boolean or score indicating whether the domain's keywords are currently trending. |
| `trending_message` | 🟡 **TBD** — Human-readable explanation accompanying `trending` when set (e.g. news context). |


---

## Enums

### `price_range_retail` / `price_range_no_tm` brackets

Each bracket ID maps to a USD range. Bracket `0` (the schema default) means *no bracket assigned*.

| Bracket ID | Min (USD) | Max (USD) |
|---:|---:|---:|
| 1 | 0 | 0 |
| 2 | 1 | 10 |
| 3 | 11 | 25 |
| 4 | 26 | 50 |
| 5 | 51 | 100 |
| 6 | 101 | 250 |
| 7 | 251 | 500 |
| 8 | 501 | 750 |
| 9 | 751 | 1,000 |
| 10 | 1,001 | 2,000 |
| 11 | 2,001 | 3,000 |
| 12 | 3,001 | 4,000 |
| 13 | 4,001 | 5,000 |
| 14 | 5,001 | 7,000 |
| 15 | 7,001 | 10,000 |
| 16 | 10,001 | 15,000 |
| 17 | 15,001 | 25,000 |
| 18 | 25,001 | 50,000 |
| 19 | 50,001 | 99,999 |
| 20 | 100,000 | 249,999 |
| 21 | 250,000 | 499,999 |
| 22 | 500,000 | 749,999 |
| 23 | 750,000 | 999,999 |
| 24 | 1,000,000 | 50,000,000 |

### `site_status` (partial)

Documented values from the schema. Values outside this range have been observed in cache responses (e.g. `7`); the full mapping is still being confirmed.

| Value | Meaning |
|---:|---|
| 0 | not resolving |
| 1 | developed |
| 2 | developed external frame |
| 3 | parked |
| 4 | placeholder |
| 5 | error |
