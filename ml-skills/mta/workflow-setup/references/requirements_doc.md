# Phase 1: Requirements Gathering

Collect all information needed to configure the MTA workflow before touching any configuration files.

## Step 1: Confluence Folder Setup

Read `../../../shared/confluence_folder_setup.md` for the full folder discovery and creation flow. Pass these MTA-specific values:

- **Solution folder name:** `MTA Journey Analytics`
- **Title variants for fuzzy matching:** `MTA`, `Journey Analysis`, `Multi-Touch Attribution`, `MTA Journey Analytics`
- **Page naming convention:**

> ⚠️ Every page created in the CUST space MUST suffix the title with the customer name. Never create pages without the suffix — those titles already exist for other customers and the create call will fail with a 400 error.

| Page type | Required title pattern |
|---|---|
| Customer root folder | `<Customer Name>` |
| FDE/ML solutions sub-folder | `FDE Solutions - <Customer Name>` |
| MTA solution sub-folder | `MTA Journey Analytics - <Customer Name>` |
| Requirements gathering page | `MTA Requirements Gathering - <Customer Name>` |
| Configuration summary page | `MTA Configuration Summary - <Customer Name>` |
| Architecture & schema page | `MTA Architecture & Output Schema - <Customer Name>` |
| Runbook page | `MTA Runbook & Maintenance - <Customer Name>` |

> 🚫 Do NOT include auto-generation disclaimers in published Confluence pages. The published page must read as if a human FDE author produced it.

## Step 2: Check for Existing Requirements Doc

Ask the user:

> Do you have an existing filled-out requirements gathering doc? If yes, paste the Confluence link.

If provided, read the page content using `getConfluencePage` and extract whatever configuration details are available (database, tables, columns, conversion definition, etc.). Use the extracted values to pre-fill later steps, but still validate everything through auto-discovery.

The standard MTA requirements template lives at:
`https://treasure-data.atlassian.net/wiki/spaces/PS/pages/2684977527/MTA+Model+-+Requirements+Gathering+Template`

## Step 3: Initial Questions

Collect answers to these questions before exploring any data. These determine the scope and complexity of the MTA configuration.

> **How to ask:** Never present these as a plain text list. Always use the `AskUserQuestion` tool so each question renders as an interactive selector. Group into batches of up to 4 questions per call (tool limit). For each question provide 2–4 pre-populated answer options — mark the recommended default with `(Recommended)` — plus the implicit "Other" option that lets the user type a custom answer. Ask all questions in Step 3 across `AskUserQuestion` calls before proceeding to Step 4.

### 3a: Data Readiness

Ask: **Has the customer's data gone through ID Unification? Do they have a `gldn` database with enriched/unified tables?**

- If **yes**: The `unique_user_id` will likely be `canonical_id` and tables will be in a `gldn_*` database
- If **no**: Ask — **Is there a unique identifier that can be used as the main `customer_id` across all tables?** (e.g., `cdp_profile_id`, `user_id`, `email`). This becomes `unique_user_id` in the config.

### 3b: Web Activity & UTM Data

Ask: **Is web activity (pageviews) data already in TD, and does it include a URL column with UTM params and a referrer column?**

This is critical — MTA needs to parse **channel, source, and campaign** from web touchpoints. If UTM parameters are missing or sparse, attribution quality will be limited.

Follow-up discovery — run after identifying the web table and its URL column name via `DESCRIBE`:

> ⚠️ **Do NOT assume UTM absence from visual inspection of sample URL values.** Always run the coverage query below — clean-looking path values may still have UTM params on a large fraction of rows.

```sql
-- Step 1: identify URL column name via DESCRIBE, then substitute <url_col> below

-- Step 2: check UTM coverage
SELECT
  COUNT(*) AS total_rows,
  COUNT(CASE WHEN url_extract_parameter(<url_col>, 'utm_source') IS NOT NULL THEN 1 END) AS has_utm_source,
  COUNT(CASE WHEN url_extract_parameter(<url_col>, 'utm_medium') IS NOT NULL THEN 1 END) AS has_utm_medium,
  COUNT(CASE WHEN url_extract_parameter(<url_col>, 'utm_campaign') IS NOT NULL THEN 1 END) AS has_utm_campaign
FROM database_name.web_table
WHERE td_interval(time, '-90d')

-- Step 3: if UTMs present, sample the values
SELECT
  url_extract_parameter(<url_col>, 'utm_source') AS utm_source,
  url_extract_parameter(<url_col>, 'utm_medium') AS utm_medium,
  url_extract_parameter(<url_col>, 'utm_campaign') AS utm_campaign,
  COUNT(*) AS cnt
FROM database_name.web_table
WHERE <url_col> LIKE '%utm_%'
  AND td_interval(time, '-90d')
GROUP BY 1, 2, 3 ORDER BY 4 DESC LIMIT 20;
```

### 3c: Touchpoint Sources

Ask: **Beyond pageviews, what other touchpoints/channels should be included in the MTA analysis?**

Provide examples to prompt the user:
- Email activity (opens, clicks from SFMC/Marketo/Braze)
- Sales rep interactions (calls, meetings from SFDC)
- Store visits / in-person events
- Ad impressions (Facebook, Google Ads, display)
- Webinar attendance
- Customer support interactions
- Form fills / subscription events

Each additional source becomes an entry in `aggregate_metrics_tables`.

### 3d: Conversion Definition

Ask: **What counts as a conversion? Is it different from a purchase/subscription with monetary value?**

This is the most important business decision. Common patterns:

| Conversion Type | `conversion_flag` Logic | `item_price` |
|----------------|------------------------|-------------|
| Purchase on website | `IF(REGEXP_LIKE(lower(td_path), 'thank_you'), 1.0, 0.0)` on pageviews | `1.0` (count) or revenue column |
| Completed order | `1.0` on orders table with status filter | `unit_price` or `total_amount` |
| Form submission | `IF(REGEXP_LIKE(lower(td_path), 'form_submit\|application'), 1.0, 0.0)` | `1.0` (count-based, no monetary value) |
| Subscription signup | `1.0` on subscriptions table | subscription price or `1.0` |
| Lead qualification | `1.0` on CRM table with status = 'qualified' | `1.0` (count-based) |

**Key follow-up**: If the conversion is **not** a monetary transaction (e.g., mortgage application, form fill), the attribution model uses count-based metrics (`1.0`) instead of revenue. Confirm this with the user.

### 3e: Time Filters & Business Rules

Ask: **Should we apply any time filters or rule-based filters to the behavior tables?**

Examples:
- Only use last 180 days of data (`lookback_period: -180d`)
- Only include orders where `order_status = 'COMPLETE'`
- Filter by specific region or market

### 3f: Model Scope

Ask: **Does the scope include ML attribution models (Shapley and Markov), or only standard models (First Touch, Last Touch, U-Shaped, Linear)?**

- **Standard models** — simpler, faster, no ML dependency
- **ML models (Markov + Shapley)** — more accurate, require more data, longer compute time


### 3h: Customer Enrichment (Optional)

Ask: **Should we enrich the journey data with customer-level attributes (e.g., RFM segments)?**

If yes, this requires:
- An existing customer table (e.g., `rfm_output_table`)
- A list of columns to join (e.g., `recency, frequency, monetary_value, rfm_segment`)

This maps to the `add_customer_analysis` config section.

## Step 4: Data Source Inventory

After initial questions, build a source table inventory. For each touchpoint source, collect:

| Field | Pageviews | Orders | Email | Sales/CRM | Other |
|-------|-----------|--------|-------|-----------|-------|
| **Table name** | `db.enriched_pageviews` | `db.enriched_orders` | `db.enriched_email_events` | `db.sales_rep_interactions` | `db.table_name` |
| **Description** | Web activity from JS SDK | Transaction history | Email engagement from ESP | Sales notes from SFDC | Custom source |
| **UNIXTIME column** | `time` | `time` | `time` | `time` | ? |
| **User ID column** | `canonical_id` | `canonical_id` | `canonical_id` | `canonical_id` | ? |
| **URL column** | `td_url` | N/A | N/A | N/A | ? |
| **Referrer column** | `td_referrer` | N/A | N/A | N/A | ? |
| **Channel/source/campaign** | UTM from URL | N/A | `campaign_name`, `email_name` | `source`, `topic` | ? |
| **Conversion flag** | URL pattern or `0.0` | `1.0` (with status filter) | `0.0` | `0.0` | ? |
| **Revenue column** | `1.0` (count) | `unit_price` | `0.0` | `0.0` | ? |
| **Filters needed** | Language, bot exclusion | Status filter | Exclude 'send' events | NULL user filter | ? |

**IMPORTANT**: For each table, you MUST identify:
1. The `UNIXTIME` column (often `time`, but may need conversion from datetime)
2. How to extract/flag each touchpoint with `channel, source, campaign`
3. The conversion event definition and monetary metric

Use `DESCRIBE` and `SELECT * LIMIT 10` via td-skills to fill in unknowns. See `table_configuration.md` for per-table-type discovery queries.

## Step 5: Validate & Confirm

Before proceeding to YAML generation, present the full requirements summary to the user:

```
MTA Journey Analytics — Requirements Summary
=============================================

Customer: [name]
Database: [database_name]
User ID column: [unique_user_id]
Confluence folder: [page URL]

Touchpoint Sources:
1. [table_name] — [description] — conversion: [yes/no]
2. [table_name] — [description] — conversion: [yes/no]
...

Conversion Definition: [what triggers conversion_flag: 1.0]
Revenue Metric: [column name or count-based (1.0)]

Time Filter: [none / range: start-end / interval: lookback]
Business Rules: [order status filter, language filter, etc.]

Model Scope: [standard only / standard + ML (Markov, Shapley)]
Dashboard: [TI dashboard / external BI / none]
Customer Enrichment: [none / table + columns]

Please confirm this is correct before I proceed with configuration.
```

Once confirmed, proceed to the YAML generation workflow in `workflow-setup/SKILL.md` Step 4.
