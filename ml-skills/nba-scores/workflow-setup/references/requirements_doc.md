# Phase 1: Requirements Gathering

Collect all information needed to configure the NBA Engagement Scores workflow before touching any configuration files. This document is built on the canonical NBA requirements template at:

`https://treasure-data.atlassian.net/wiki/spaces/PS/pages/2695593985/Next+Best+Action+-+Engagement+Scores+-+Requirements+Gathering+Template`

with additional questions covering scoring strategy, business-rule windows, dashboarding, and timezone alignment.

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

**Search for an existing sub-folder** with:

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

### Step 1c: Locate or Create the NBA Sub-Folder

Within the ML/FDE sub-folder, find or create a folder specific to the NBA project.

**Search for an existing NBA folder**:

```
getConfluencePageDescendants:
  cloudId: treasure-data.atlassian.net
  pageId: <ml_fde_folder_page_id>
  depth: 1
  limit: 50
```

Scan the results for a page whose title matches any of these patterns (case-insensitive):
- `NBA Engagement Scores`
- `Next Best Action`
- `NBA`
- `Engagement Scores`

If a match is found, use that page's ID.

**If no match is found**, create the sub-folder:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <ml_fde_folder_page_id>
  title: "NBA Engagement Scores"
  contentFormat: markdown
  body: "Next Best Action engagement scores workflow documentation for this customer."
```

### Store the Folder IDs

Save both:
- **ML/FDE sub-folder page ID**
- **NBA sub-folder page ID** — you'll use this as the `parentId` when creating documentation pages later in the prod-docs phase

The final page hierarchy will be:
```
[Customer Folder]
└── FDE Solutions (or ML & Analytics Projects, etc.)
    └── NBA Engagement Scores          ← parentId for prod-docs
        ├── NBA Configuration Summary
        ├── NBA Architecture & Output Schema
        └── NBA Runbook & Maintenance
```

## Step 2: Check for Existing Requirements Doc

Ask the user:

> Do you have an existing filled-out NBA requirements gathering doc? If yes, paste the Confluence link.

If provided, read the page content using `getConfluencePage` and extract whatever configuration details are already documented (database, tables, columns, scoring strategy, etc.). Use the extracted values to pre-fill later steps, but still validate everything through auto-discovery against the actual data.

The standard NBA requirements template lives at:
`https://treasure-data.atlassian.net/wiki/spaces/PS/pages/2695593985/Next+Best+Action+-+Engagement+Scores+-+Requirements+Gathering+Template`

## Step 3: Initial Questions

Walk through the questions below before exploring any data. Together they determine the scope, scoring approach, and business-rule windows for the NBA configuration.

### 3a: Data Readiness

Ask: **Has the customer's data gone through ID Unification? Do they have a `gldn` database with enriched/unified tables?**

- If **yes**: `unique_user_id` will likely be `canonical_id` and source tables will be in a `gldn_*` database.
- If **no**: ask **Is there a unique identifier that can be used as the main `customer_id` across all tables?** (e.g., `cdp_profile_id`, `user_id`, `email_hash`). That becomes `unique_user_id` in the config.

### 3b: Parent Segment & Activation

Ask: **What is the name of the Parent Segment that will be used for Audience building and Activation, where the NBA recommendations need to be added as attributes?**

This is critical for the deployment phase — the `nba_combined_metrics_final` table will be added as an attribute of this Parent Segment so the scores become filterable in Audience Studio.

### 3c: Use-Case Scope

Ask: **Please describe the high-level use cases this model will be used for and who the end users will be.**

Examples:
- Marketing ops: pick the daypart with highest engagement to schedule sends.
- Channel strategy: shift budget toward each user's top-affinity channel.
- Retargeting: trigger cart-abandon journeys; suppress new-visitor audiences from aggressive ads.

The answer here directly informs which NBA metrics the customer cares about most — and therefore which `regexp_columns` to keep when joining temp tables into the final output.

### 3d: NBA Use-Case Type

Ask: **Is this a Next Best Channel / Next Best Time / Next Best Offer use case (engagement-score-based), or a Next Best Product recommendation use case?**

- **Engagement scores (Channel / Time / Offer)** → this skill applies. Continue with the questions below.
- **Next Best Product** → use the separate `ml-skills:nbp-agent-skills` skill instead. Stop here and route to that skill.

### 3e: Cart Abandon & New Visitor Definitions

Ask:

> Do you have a specific definition for **New Visitor** or **Cart Abandon** in terms of time frames and data context?

Pull these specific answers:

| Question | YAML field |
|----------|------------|
| How many days back should we look for an add-to-cart-without-purchase signal? | `next_best_campaign.event_lookback` (default: `90`) |
| Is the customer's add-to-cart event named anything besides "add cart"? | `next_best_campaign.abandon_regexp` |
| What's the max age of first visit (days) for someone to count as "new"? | `next_best_campaign.new_customers_days` (default: `45`) |
| What's the max page-visit count for "new"? | `next_best_campaign.max_number_visits` (default: `15`) |
| Are there channels/sources that should disqualify someone from being "new" (i.e., paid-traffic channels)? | `next_best_campaign.ad_engagement_logic` |

### 3f: Time Window for Scoring

Ask:

> Should we use any time-period filters on the behavioral data when scoring engagement, or should we use all historical behavior available in CDP?

Pull these answers:

| Question | YAML field |
|----------|------------|
| `range` (fixed start/end) or `interval` (rolling lookback)? | `time_filter_type` |
| If `range`: what start/end dates? | `time_range_start_date` / `time_range_end_date` |
| If `interval`: how far back? | `lookback_period` (e.g. `-180d`, `-1y`) |

### 3g: Scoring Strategy

Ask:

> Three scoring strategies are available — which fits the customer's data best?
>
> - **percentile** — score = percentile rank within the audience. Default for evenly distributed activity.
> - **quartile** — bucket users into 4 evenly-sized groups. Easier to communicate to marketers ("top quartile only").
> - **minmax** — Hivemall min-max scaling. Best when activity distribution is heavily skewed by power users.

Quick rule of thumb: if the customer has a lot of low-frequency users and a small group of very active ones, recommend `minmax`. If they want simple "top 25% only" segmentation, recommend `quartile`. Otherwise default to `percentile`.

This sets `scoring_logic`.

### 3h: Time-of-Day Granularity

Ask:

> The default time-of-day breakdown is four 6-hour periods (morning / afternoon / evening / overnight). Does the customer need finer granularity (e.g., 8 three-hour periods) or shifted boundaries (e.g., morning starts at 5 AM for early-rising audiences)?

This sets `next_best_time.day_breakdown`. Most customers stick with the default — only override if they have a specific operational reason.

### 3i: Timezone Alignment

Ask:

> Source timestamps are in UTC. What timezone does the customer's marketing operations team plan campaigns in?

This sets `time_zone`, `timeshift_change`, and `timeshift_hours`. Common values:

| Customer timezone | `timeshift_change` | `timeshift_hours` |
|-------------------|-------------------|-------------------|
| UTC (no shift) | `+` | `0` |
| EST (UTC-5) | `-` | `5` |
| PST (UTC-8) | `-` | `8` |
| JST (UTC+9) | `+` | `9` |
| KST (UTC+9) | `+` | `9` |

### 3j: Touchpoint Sources

Ask: **Beyond pageviews, which sources of customer behavior should feed the engagement score?**

The four standard sources the workflow expects are:

| Source | Description | Important columns |
|--------|-------------|-------------------|
| `customers` / `user_master` | Distinct profile identifiers (e.g. `td_canonical_id`) | unique identifier used as the join key for the Parent Segment |
| `pageviews` | Web activity from JS SDK or customer's source data (Adobe Analytics, GA export) | `UNIXTIME` timestamp; UTM availability in URL; conversion-page patterns |
| `orders` | Transaction table with order type, status, date, revenue | `UNIXTIME` timestamp; `purchase_price`; `order_status` filter to avoid counting cancelled/returned |
| `email` | KPIs such as opens, sends, clicks | `UNIXTIME` timestamp; event-type column for filtering out 'send' |
| _any other activity_ | Sales-rep notes, in-store events, support tickets, webinar attendance, form fills, ad impressions | `UNIXTIME` timestamp; profile ID column |

For each source, capture: full table name, primary timestamp column, profile ID column, and any business-rule filter (e.g. order_status whitelist, language filter).

### 3k: Conversion Definition

Ask:

> What is the exact conversion event we should use for cart-abandon detection and the conversion flag column?

Common patterns:

| Conversion type | Where it lives | `conversion_flag` |
|-----------------|----------------|-------------------|
| Online purchase | pageviews `td_path ~ '/order-received\|/thank-you'` | `IF(REGEXP_LIKE(...), 1.0, 0.0)` on pageviews |
| Lead form submit | pageviews `td_path ~ '/thank\|/download'` | `IF(REGEXP_LIKE(...), 1.0, 0.0)` on pageviews |
| Completed order | orders table with status filter | `1.0` on orders + `order_status IN (...)` filter |
| Subscription signup | subscriptions table | `1.0` on subscriptions |

### 3l: Dashboard Build

Ask: **Should we build the TI dashboard for this customer?**

- If **yes** → `create_dashboard: 'yes'`. Workflow will run `nba_datamodel_create.dig` + `nba_datamodel_build.dig` and the dashboard template `dashboard/NBA_Engagement_Scores.dash` will need to be uploaded to TI manually.
- If **no** → `create_dashboard: 'no'`. The `nba_dash_*` tables will still be populated (the NBA Insights agent depends on them), but the TI datamodel won't be created.

## Step 4: Data Source Inventory

After the initial questions, build a per-source inventory. For each touchpoint, capture:

| Field | Pageviews | Orders | Email | Sales/CRM | Other |
|-------|-----------|--------|-------|-----------|-------|
| **Table name** | `<db>.enriched_pageviews` | `<db>.enriched_orders` | `<db>.enriched_email_events` | `<db>.sales_rep_interactions` | `<db>.<table>` |
| **Description** | Web activity from JS SDK | Transaction history | Email engagement from ESP | Sales notes from SFDC | Custom |
| **`unixtime_col`** | `time` | `time` | `time` | `time` | ? |
| **Profile ID** | `canonical_id` | `canonical_id` | `canonical_id` | `canonical_id` | ? |
| **URL column** | `td_url` | N/A | N/A | N/A | ? |
| **Referrer column** | `td_referrer` | N/A | N/A | N/A | ? |
| **Channel/source/campaign** | UTM from URL | N/A | `campaign_name`, `email_name` | `source`, `topic` | ? |
| **`conversion_flag`** | URL pattern OR `0.0` | `1.0` (with status filter) | `0.0` | `0.0` | ? |
| **Revenue column** | `0.0` | `unit_price` | `0.0` | `0.0` | ? |
| **Filters needed** | Language, bot exclusion | Status filter | Exclude 'send' events | NULL profile filter | ? |

For each table, you MUST verify three things:
1. The `UNIXTIME` column exists (often `time`; if not, identify the conversion needed: `TD_TIME_PARSE(datetime_col)`)
2. The profile ID column matches `unique_user_id`
3. The conversion logic / channel-extraction columns are present

Use `DESCRIBE` and `SELECT * LIMIT 10` via tdx-skills to fill in unknowns. See `table_configuration.md` for per-source-type discovery queries.

## Step 5: Validate & Confirm

Before proceeding to YAML generation, present the full requirements summary to the user:

```
NBA Engagement Scores — Requirements Summary
=============================================

Customer: [name]
Database: [database_name]
Profile ID column: [unique_user_id]
Parent Segment for activation: [segment_name]
Confluence folder: [page URL]

Use-Case Scope:
  Type: Engagement scores (Channel / Time / Offer)
  Primary use case: [from question 3c]

Touchpoint Sources:
1. [table_name] — [description] — conversion: [yes/no]
2. [table_name] — [description] — conversion: [yes/no]
...

Conversion Definition: [what triggers conversion_flag: 1.0]

Time Window: [range YYYY-MM-DD to YYYY-MM-DD] OR [interval -180d]

Scoring Strategy: [percentile / quartile / minmax] — [reasoning]

Business Rule Windows:
  cart-abandon lookback (event_lookback): [N days]
  new-visitor max age (new_customers_days): [N days]
  new-visitor max page visits (max_number_visits): [N]

Time-of-Day Buckets: [default 4-bucket OR custom: ...]

Timezone: [UTC + N hours OR UTC - N hours]

Dashboard Build: [yes / no]

Please confirm this is correct before I generate the YAML.
```

Once confirmed, proceed to the YAML generation workflow in `workflow-setup/SKILL.md` Step 4.
