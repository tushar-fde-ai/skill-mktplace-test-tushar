# NBA Engagement Scores — Table Configuration Guide

This guide explains how to configure each type of source table inside `aggregate_metrics_tables`. Every table you add becomes one source feeding the unioned activity table that NBA scores against.

## Per-Source Column Reference

NBA needs every source to expose a consistent set of columns even if some don't apply to that source — fill the unused ones with `CAST(NULL AS VARCHAR)` or a hardcoded label.

| Column | Purpose | Web | Email | Sales | Orders |
|--------|---------|-----|-------|-------|--------|
| `unixtime_col` | Event timestamp | `time` | `time` | `time` | `time` |
| `join_key` | Profile ID | `${unique_user_id}` | `${unique_user_id}` | `${unique_user_id}` | `${unique_user_id}` |
| `url_col` | Page URL | `td_url` | `CAST(NULL AS VARCHAR)` | `CAST(NULL AS VARCHAR)` | `CAST(NULL AS VARCHAR)` |
| `referral_col` | Traffic referrer | `td_referrer` | `"''email''"` | `CAST(NULL AS VARCHAR)` | `CAST(NULL AS VARCHAR)` |
| `medium_col` | Marketing medium | UTM extraction | `"''email''"` | `"''sales_rep_interactions''"` | `CAST(NULL AS VARCHAR)` |
| `source_col` | Marketing source | UTM extraction | `"''sfmc''"` | column or hardcoded | `CAST(NULL AS VARCHAR)` |
| `campaign_col` | Campaign name | UTM extraction | normalized campaign column | topic column | `CAST(NULL AS VARCHAR)` |
| `context_col` | Touchpoint context | `td_os` or `td_title` | email name | topic | `order_type` |
| `event_type` | Event label expression | `"''web_activity''"` | `CONCAT('email_', event_type)` | `CONCAT('sales_rep_interactions_', source)` | `CONCAT(lower(order_type), '_order')` |
| `conversion_flag` | Is this row a conversion? | URL pattern → 1.0 / 0.0 | `0.0` | `0.0` | `1.0` |
| `item_price` | Revenue value | `0.0` | `0.0` | `0.0` | revenue column |

> NBA does NOT use `utm_mailing_col` — that's an MTA-specific field. Don't add it here.

---

## Source Type: Pageviews / Web Activity

### Purpose
The primary source of channel and time signals for NBA scoring. UTM parameters from the URL drive the channel affinity score; event timestamps drive the time-of-day score; URL-pattern matching (e.g., `/thank_you`) defines conversion events.

### Required Columns
- **Timestamp**: `time` (UNIX). If only datetime, wrap with `TD_TIME_PARSE(...)`.
- **Profile ID**: `canonical_id` (or whatever `unique_user_id` resolves to)
- **URL**: `td_url` or `page_url` — needed for UTM extraction and conversion-pattern matching

### Discovery Steps

```sql
SHOW TABLES IN <customer_database> LIKE '%pageview%';
DESCRIBE <customer_database>.enriched_pageviews;
SELECT * FROM <customer_database>.enriched_pageviews LIMIT 10;

-- Check UTM parameter availability
SELECT url_extract_parameter(td_url, 'utm_source') AS source,
       url_extract_parameter(td_url, 'utm_medium') AS medium,
       COUNT(*) AS cnt
FROM <customer_database>.enriched_pageviews
WHERE td_url IS NOT NULL
GROUP BY 1, 2 ORDER BY 3 DESC LIMIT 20;

-- Check for conversion URL patterns
SELECT td_path, COUNT(*) FROM <customer_database>.enriched_pageviews
WHERE REGEXP_LIKE(lower(td_path), 'thank|download|confirm|success|order-received')
GROUP BY 1 ORDER BY 2 DESC LIMIT 20;
```

### Configuration Template

```yaml
- src_table: <customer_database>.enriched_pageviews
  name: 'pageviews'
  unixtime_col: time
  join_key: ${unique_user_id}
  market_col: "''none''"
  brand_col: "''none''"
  url_col: td_url
  referral_col: td_referrer
  medium_col: "url_extract_parameter(LOWER(td_url),''utm_medium'')"
  source_col: "url_extract_parameter(LOWER(td_url),''utm_source'')"
  campaign_col: "url_extract_parameter(LOWER(td_url),''utm_campaign'')"
  context_col: td_os
  event_type: "''web_activity''"
  custom_filter: "${unique_user_id} IS NOT NULL AND REGEXP_LIKE(lower(td_language), ''en'')"
  conversion_flag: "IF(REGEXP_LIKE(lower(td_path), ''thank|download''), 1.0, 0.0)"
  item_price: 0.0
  apply_time_filter: false
  query_type:
```

### Key Decisions
- **`conversion_flag`**: Customize the regex to match the customer's conversion pages. Common patterns: `thank|download` (lead gen), `order-received|checkout/success` (e-commerce), `application/submit` (financial services).
- **`custom_filter`**: Add language filter, bot exclusion (`NOT REGEXP_LIKE(td_user_agent, 'bot|crawler')`), or geo restriction as needed.
- **`context_col`**: For time-of-day analysis, `td_os` (operating system) gives device-aware splits. Use `td_title` if you want page-title context instead.

### URL Column Variations

| Source pattern | URL column | Referrer column | Path column |
|----------------|------------|-----------------|-------------|
| `enriched_pageviews` (TD JS SDK default) | `td_url` | `td_referrer` | `td_path` |
| `web_events` | `page_url` | `referrer_url` | `page_path` |
| `page_views` | `url` | `referrer` | `path` |
| Adobe Analytics export | `pagename` or `page_url` | `referrer` | derived from `pagename` |

---

## Source Type: Email Events

### Purpose
Email channel touchpoints — opens, clicks, unsubscribes. **Never conversions** — `conversion_flag: 0.0`. Excluding `'send'` events is critical to avoid inflating engagement counts with messages the user never saw.

### Required Columns
- **Timestamp**: `time`
- **Profile ID**: `canonical_id`
- **Event type**: needed to filter out `'send'`
- **Campaign name** + **email name** for context

### Discovery Steps

```sql
SHOW TABLES IN <customer_database> LIKE '%email%';
DESCRIBE <customer_database>.enriched_email_events;

SELECT DISTINCT event_type, COUNT(*) FROM <customer_database>.enriched_email_events
GROUP BY 1 ORDER BY 2 DESC;

SELECT DISTINCT campaign_name, COUNT(*) FROM <customer_database>.enriched_email_events
GROUP BY 1 ORDER BY 2 DESC LIMIT 20;
```

### Configuration Template

```yaml
- src_table: <customer_database>.enriched_email_events
  name: 'email_events'
  unixtime_col: time
  join_key: ${unique_user_id}
  market_col: "''none''"
  brand_col: "''none''"
  url_col: CAST(NULL AS VARCHAR)
  referral_col: "''email''"
  medium_col: "''email''"
  source_col: "''sfmc''"
  campaign_col: "REGEXP_REPLACE(lower(campaign_name), ''[- ]'', ''_'')"
  context_col: "REGEXP_REPLACE(lower(email_name), ''[- ]'', ''_'')"
  event_type: "CONCAT(''email_'', event_type)"
  custom_filter: "${unique_user_id} IS NOT NULL AND NOT REGEXP_LIKE(lower(event_type), ''send'')"
  conversion_flag: 0.0
  item_price: 0.0
  apply_time_filter: false
  query_type:
```

### Key Decisions
- **`source_col`**: Hardcode to the customer's ESP — `'sfmc'`, `'marketo'`, `'braze'`, `'hubspot'`, `'mailchimp'`.
- **`custom_filter`**: Always exclude `'send'`. Add `'bounce'` and `'spam'` exclusions if those events are present.

### Email Event Types Reference

| Event type | Include? | Why |
|------------|----------|-----|
| `sent` / `send` | ❌ | Not user-initiated engagement |
| `delivered` | ❌ | Technical event |
| `open` | ✅ | Engagement |
| `click` | ✅ | Strong engagement |
| `conversion` | ✅ | Strongest engagement |
| `reply` / `forward` | ✅ | Engagement |
| `unsubscribe` | ✅ | Negative-but-real engagement signal |
| `bounce` | ❌ | Technical failure |
| `spam` | ❌ | Not valid engagement |

---

## Source Type: Sales Rep / CRM Interactions

### Purpose
Offline / human-touch channel touchpoints (calls, in-person meetings, surveys, support tickets). Not conversions — `conversion_flag: 0.0`.

### Required Columns
- **Timestamp**: `time`
- **Profile ID**: `canonical_id`
- **Source / topic** columns for channel labeling

### Discovery Steps

```sql
SHOW TABLES IN <customer_database> LIKE '%sales%';
SHOW TABLES IN <customer_database> LIKE '%crm%';
DESCRIBE <customer_database>.sales_rep_interactions;

SELECT DISTINCT source, COUNT(*) FROM <customer_database>.sales_rep_interactions
GROUP BY 1 ORDER BY 2 DESC;
SELECT DISTINCT topic, COUNT(*) FROM <customer_database>.sales_rep_interactions
GROUP BY 1 ORDER BY 2 DESC LIMIT 20;
```

### Configuration Template

```yaml
- src_table: <customer_database>.sales_rep_interactions
  name: 'sales_rep_interactions'
  unixtime_col: time
  join_key: ${unique_user_id}
  market_col: "''none''"
  brand_col: "''none''"
  url_col: CAST(NULL AS VARCHAR)
  referral_col: CAST(NULL AS VARCHAR)
  medium_col: "''sales_rep_interactions''"
  source_col: source
  campaign_col: topic
  context_col: topic
  event_type: "CONCAT(''sales_rep_interactions_'', source)"
  custom_filter: ${unique_user_id} IS NOT NULL
  conversion_flag: 0.0
  item_price: 0.0
  apply_time_filter: false
  query_type:
```

### Key Decisions
- **`source_col`**: Either a column (e.g., `source`, `interaction_type`) when distinct values are meaningful (`call`, `email`, `meeting`, `survey`), or hardcoded to a single label.
- **`custom_filter`**: For SFDC-sourced data, you may want to exclude internal/test records: `AND NOT REGEXP_LIKE(lower(source), 'internal|test')`.

---

## Source Type: Orders / Conversion Events

### Purpose
The conversion source. `conversion_flag: 1.0`. Drives both the `converted_users` metric on the dashboard tables and the cart-abandon rule (cart-abandon = added to cart in lookback window WITHOUT a subsequent order).

### Required Columns
- **Timestamp**: `time`
- **Profile ID**: `canonical_id`
- **Order status**: to filter to valid orders only
- **Revenue column** (optional): `unit_price`, `total_amount`, etc.

### Discovery Steps

```sql
SHOW TABLES IN <customer_database> LIKE '%order%';
DESCRIBE <customer_database>.enriched_orders;

SELECT DISTINCT order_status, COUNT(*) FROM <customer_database>.enriched_orders
GROUP BY 1 ORDER BY 2 DESC;

SELECT MIN(unit_price), MAX(unit_price), AVG(unit_price)
FROM <customer_database>.enriched_orders
WHERE order_status IN ('COMPLETE', 'PROCESSING');

SELECT DISTINCT order_type, COUNT(*) FROM <customer_database>.enriched_orders
GROUP BY 1 ORDER BY 2 DESC;
```

### Configuration Template

```yaml
- src_table: <customer_database>.enriched_orders
  name: 'order_events'
  unixtime_col: time
  join_key: ${unique_user_id}
  market_col: "''none''"
  brand_col: "''none''"
  url_col: CAST(NULL AS VARCHAR)
  referral_col: CAST(NULL AS VARCHAR)
  medium_col: CAST(NULL AS VARCHAR)
  source_col: CAST(NULL AS VARCHAR)
  campaign_col: CAST(NULL AS VARCHAR)
  context_col: order_type
  event_type: "CONCAT(lower(order_type), ''_order'')"
  custom_filter: ${unique_user_id} IS NOT NULL AND order_status IN (''COMPLETE'', ''PROCESSING'')
  conversion_flag: 1.0
  item_price: unit_price
  apply_time_filter: false
  query_type:
```

### Key Decisions
- **`conversion_flag: 1.0`** — every row in this table represents a conversion event.
- **`item_price`**: Use the column holding amount-paid per order. **Watch for duplicate rows per order** — if the table has one row per line-item, conversion counts will be inflated. Either pre-aggregate to one row per order in `sql/src_tables/order_events.sql` (set `query_type: 'custom'`) or accept that "conversions" are line-items.
- **`custom_filter`**: Always restrict to valid statuses. Common include patterns: `COMPLETE`, `PROCESSING`, `completed`, `shipped`, `delivered`, `paid`. Common exclude: `cancelled`, `returned`, `refunded`, `failed`, `abandoned`.
- **All channel columns NULL**: Orders are the conversion target, not a channel touchpoint.

### Order Status Filtering Cheat Sheet

| Status pattern | Action |
|----------------|--------|
| `COMPLETE`, `Complete`, `completed` | Include |
| `PROCESSING`, `processing`, `paid` | Include (already a real conversion) |
| `shipped`, `delivered` | Include |
| `cancelled`, `voided`, `returned`, `refunded` | Exclude |
| `pending`, `failed`, `abandoned` | Exclude |

---

## Adding a Custom Source

If a customer has a touchpoint source that doesn't fit the four standard types (e.g., webinar attendance, ad-impression logs, in-store transactions, IVR call logs):

1. Decide whether it's a **touchpoint** (`conversion_flag: 0.0`) or a **conversion** (`conversion_flag: 1.0`).
2. Pick a stable label for `medium_col` (hardcoded string) and use real columns for `source_col` / `campaign_col` if available.
3. Decide what `event_type` should look like — usually `CONCAT('<source_label>_', event_type)`.
4. If the underlying SQL needs joins or pre-aggregation, set `query_type: 'custom'` and write `td_wf/sql/src_tables/<name>.sql` — the workflow will read from there instead of the YAML.

### Custom Query Example (in `sql/src_tables/<name>.sql`)

```sql
-- This file is loaded when query_type: 'custom' is set on the source.
-- It must select the same canonical column set the workflow expects.
SELECT
  <unique_user_id>,
  time AS unixtime_col,
  ...
FROM <customer_database>.<custom_table>
WHERE ...
```

---

## Troubleshooting

### No UTM parameters in web data
If `url_extract_parameter` returns NULL for most rows, check whether UTM data lives in a different column or format:
```sql
SELECT td_url, COUNT(*)
FROM <customer_database>.enriched_pageviews
WHERE url_extract_parameter(td_url, 'utm_source') IS NOT NULL
GROUP BY 1 LIMIT 10;
```
Fallbacks: derive source from `td_referrer` (e.g. via custom SQL), or hardcode `medium_col: "''direct''"` for that source — at the cost of channel granularity.

### Too many channels collapse to "others"
Lower `top_k_channel_perc` (e.g. from `0.001` to `0.0001`) to keep more long-tail channels.

### No conversions detected
Check the `conversion_flag` logic against actual data:
```sql
SELECT COUNT(*) FROM <customer_database>.enriched_pageviews
WHERE REGEXP_LIKE(lower(td_path), 'thank|download');

SELECT COUNT(*) FROM <customer_database>.enriched_orders
WHERE order_status IN ('COMPLETE', 'PROCESSING');
```

### Cart-abandon flag is always 0
The `abandon_regexp` in `next_best_campaign` looks for `(?=.*add)(?=.*cart)` against `event_type`. If the customer's add-to-cart event is named differently (e.g., `basket_add`, `cart_item_added`), update both `abandon_regexp` and `abandon_regexp_string` to match.
