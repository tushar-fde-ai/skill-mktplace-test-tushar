# Phase 1: Requirements Gathering

Collect all information needed to configure the MTA workflow before touching any configuration files.

## Step 1: Locate the Customer's Confluence Folder

The customer documentation lives in the **Customers** Confluence space (CUST), organized by region:

```
Customers (CUST, space ID: 9797636)
├── US/ROWs    (page ID: 44728439)    → customer folders alphabetically
├── Japan      (page ID: 643963288)   → customer folders
└── Korea      (page ID: 1824266587)  → customer folders
```

Ask the user to identify the customer's Confluence folder using **one of two methods**:

### Method A: Search by Customer Name (Preferred)

Ask: **What is the customer name?**

Then search for their folder in the CUST space:

```
searchConfluenceUsingCql:
  cloudId: treasure-data.atlassian.net
  cql: space = "CUST" AND type = page AND title ~ "<customer_name>"
  limit: 10
```

From the results, identify the correct page by checking:
1. The page title matches or closely matches the customer name
2. The page's `parentId` is one of the three region pages (`44728439`, `643963288`, `1824266587`) — confirming it's a top-level customer folder, not a deeply nested subpage

If multiple matches are found, show them to the user and ask which one is correct.
If no matches are found, ask the user to provide a direct link (Method B).

### Method B: User Provides a Page URL or ID

Ask: **Can you paste a link to any page in the customer's Confluence folder?**

Extract the page ID from the URL. Confluence URLs look like:
- `https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/<pageId>/Page+Title`
- `https://treasure-data.atlassian.net/wiki/x/<tinyId>` (tiny link — pass the `tinyId` to `getConfluencePage` as the `pageId`)

Once you have the page ID, read the page to get its `parentId`. If the page itself is the customer folder (i.e., its parent is a region page), use its ID. Otherwise, walk up the tree until you find the customer-level folder.

### Step 1b: Locate or Create the FDE Solutions Sub-Folder

Documentation pages should live under an **FDE Solutions** sub-folder within the customer folder — not directly under the customer root.

**Search for an existing sub-folder**:

Get the direct children of the customer folder and look for a match:

```
getConfluencePageDescendants:
  cloudId: treasure-data.atlassian.net
  pageId: <customer_folder_page_id>
  depth: 1
  limit: 50
```

Scan the results for a page whose title matches any of these patterns (case-insensitive):
- `FDE Solutions`
- `ML & Analytics Solutions`
- `ML & Analytics Projects`
- `ML Solutions`
- `ML Projects`
- `Analytics Solutions`
- `Analytics Projects`
- `FDE`

Also match titles that include the customer name as a suffix (e.g., `ML & Analytics Projects - SCI`).

If a match is found, use that page's ID.

**If no match is found**, create the sub-folder:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <customer_folder_page_id>
  title: "FDE Solutions"
  contentFormat: markdown
  body: "Landing page for Forward Deployed Engineering solutions deployed for this customer."
```

### Step 1c: Locate or Create the MTA Sub-Folder

Within the ML/FDE sub-folder, create (or find) a folder specific to the MTA project.

**Search for an existing MTA folder**:

Get the direct children of the ML/FDE sub-folder:

```
getConfluencePageDescendants:
  cloudId: treasure-data.atlassian.net
  pageId: <ml_fde_folder_page_id>
  depth: 1
  limit: 50
```

Scan the results for a page whose title matches any of these patterns (case-insensitive):
- `Journey Analysis`
- `MTA`
- `Multi-Touch Attribution`
- `MTA Journey Analytics`

If a match is found, use that page's ID.

**If no match is found**, create the sub-folder:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <ml_fde_folder_page_id>
  title: "MTA Journey Analytics"
  contentFormat: markdown
  body: "Multi-Touch Attribution journey analytics workflow documentation for this customer."
```

### Store the Folder IDs

Save both:
- **ML/FDE sub-folder page ID**
- **MTA sub-folder page ID** — you'll use this as the `parentId` when creating documentation pages in Phase 5

The final page hierarchy will be:
```
[Customer Folder]
└── FDE Solutions (or ML & Analytics Projects, etc.)
    └── MTA Journey Analytics          ← parentId for Phase 5
        ├── MTA Configuration Summary
        ├── MTA Architecture & Output Schema
        └── MTA Runbook & Maintenance
```

## Step 2: Check for Existing Requirements Doc

Ask the user:

> Do you have an existing filled-out requirements gathering doc? If yes, paste the Confluence link.

If provided, read the page content using `getConfluencePage` and extract whatever configuration details are available (database, tables, columns, conversion definition, etc.). Use the extracted values to pre-fill later steps, but still validate everything through auto-discovery.

The standard MTA requirements template lives at:
`https://treasure-data.atlassian.net/wiki/spaces/PS/pages/2684977527/MTA+Model+-+Requirements+Gathering+Template`

## Step 3: Initial Questions

Collect answers to these questions before exploring any data. These determine the scope and complexity of the MTA configuration.

### 3a: Data Readiness

Ask: **Has the customer's data gone through ID Unification? Do they have a `gldn` database with enriched/unified tables?**

- If **yes**: The `unique_user_id` will likely be `canonical_id` and tables will be in a `gldn_*` database
- If **no**: Ask — **Is there a unique identifier that can be used as the main `customer_id` across all tables?** (e.g., `cdp_profile_id`, `user_id`, `email`). This becomes `unique_user_id` in the config.

### 3b: Web Activity & UTM Data

Ask: **Is web activity (pageviews) data already in TD, and does it include `utm_params` in the URL column and a `referrer` column?**

This is critical — MTA needs to parse **channel, source, and campaign** from web touchpoints. If UTM parameters are missing or sparse, attribution quality will be limited.

Follow-up discovery (run after getting database name):
```sql
-- Check UTM availability in pageviews
SELECT
  COUNT(*) as total_rows,
  COUNT(CASE WHEN url_extract_parameter(td_url, 'utm_source') IS NOT NULL THEN 1 END) as has_utm_source,
  COUNT(CASE WHEN url_extract_parameter(td_url, 'utm_medium') IS NOT NULL THEN 1 END) as has_utm_medium,
  COUNT(CASE WHEN url_extract_parameter(td_url, 'utm_campaign') IS NOT NULL THEN 1 END) as has_utm_campaign
FROM database_name.enriched_pageviews
WHERE td_interval(time, '-90d')
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
