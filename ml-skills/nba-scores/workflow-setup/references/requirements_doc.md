# NBA Engagement Scores — Requirements Gathering

NBA-specific questions and data inventory for configuring the workflow. 

**Confluence folder setup** (locating the customer folder, creating FDE Solutions and NBA sub-folders) is handled by `../../../shared/confluence_folder_setup.md`. Pass `solution-folder-name: NBA Engagement Scores` and title variants: `NBA Engagement Scores`, `Next Best Action`, `NBA`, `Engagement Scores`.

**Requirements page creation + handoff mechanics** (creating the Confluence page, sharing with the customer, session-end pattern) are handled by `../../../shared/requirements_doc_pattern.md`. Page title: `NBA Requirements Gathering - <Customer>`.

The canonical NBA requirements template lives at:
`https://treasure-data.atlassian.net/wiki/spaces/PS/pages/2695593985/Next+Best+Action+-+Engagement+Scores+-+Requirements+Gathering+Template`

---

## Step 1: Initial Questions

Walk through the questions below before exploring any data. Together they determine the scope, scoring approach, and business-rule windows for the NBA configuration.

> **How to ask:** Never present these as a plain text list. Always use the `AskUserQuestion` tool so each question renders as an interactive selector. Group into batches of up to 4 questions per call (tool limit). For each question provide 2–4 pre-populated answer options — mark the recommended default with `(Recommended)` — plus the implicit "Other" option that lets the user type a custom answer. Ask all questions in Step 1 across two `AskUserQuestion` calls before proceeding to Step 2.

### 1a: Data Readiness

Ask: **Has the customer's data gone through ID Unification? What is the name of the database with the final enriched/unified tables?**

- If **yes**: `unique_user_id` will likely be `canonical_id` , but you should verify by exploring the data in the enriched/unified tables in DB provided by user.
- If **no**: ask **Is there a unique identifier that can be used as the main `customer_id` across all tables?** (e.g., `cdp_profile_id`, `user_id`, `email_hash`). That becomes `unique_user_id` in the config.

### 1b: Parent Segment & Activation

Ask: **What is the name of the Parent Segment that will be used for Audience building and Activation, where the NBA recommendations need to be added as attributes?**

This is critical for the deployment phase — the `nba_combined_metrics_final` table will be added as an attribute of this Parent Segment so the scores become filterable in Audience Studio.

### 1c: Use-Case Scope

Ask: **Please describe the high-level use cases this model will be used for and who the end users will be.**

Examples:
- Marketing ops: pick the daypart with highest engagement to schedule sends.
- Channel strategy: shift budget toward each user's top-affinity channel.
- Retargeting: trigger cart-abandon journeys; suppress new-visitor audiences from aggressive ads.

The answer here directly informs which NBA metrics the customer cares about most — and therefore which `regexp_columns` to keep when joining temp tables into the final output.

### 1d: NBA Use-Case Type

Ask: **Is this a Next Best Channel / Next Best Time / Next Best Offer use case (engagement-score-based), or a Next Best Product recommendation use case?**

- **Engagement scores (Channel / Time / Offer)** → this skill applies. Continue with the questions below.
- **Next Best Product** → use the separate `ml-skills:fde-nbp` skill instead. Stop here and route to that skill.

### 1e: Cart Abandon & New Visitor Definitions

Ask:

> Do you have a specific definition for **New Visitor** or **Cart Abandon** in terms of time frames and data context?

| Question | YAML field |
|----------|------------|
| How many days back should we look for an add-to-cart-without-purchase signal? | `next_best_campaign.event_lookback` (default: `90`) |
| Is the customer's add-to-cart event named anything besides "add cart"? | `next_best_campaign.abandon_regexp` |
| What's the max age of first visit (days) for someone to count as "new"? | `next_best_campaign.new_customers_days` (default: `45`) |
| What's the max page-visit count for "new"? | `next_best_campaign.max_number_visits` (default: `15`) |
| Are there channels/sources that should disqualify someone from being "new" (i.e., paid-traffic channels)? | `next_best_campaign.ad_engagement_logic` |

### 1f: Time Window for Scoring

Ask:

> Should we use any time-period filters on the behavioral data when scoring engagement, or should we use all historical behavior available in CDP?

| Question | YAML field |
|----------|------------|
| `range` (fixed start/end) or `interval` (rolling lookback)? | `time_filter_type` |
| If `range`: what start/end dates? | `time_range_start_date` / `time_range_end_date` |
| If `interval`: how far back? | `lookback_period` (e.g. `-180d`, `-1y`) |
| Apply this time window to source tables, or use all historical data? | `apply_time_filter` on **every** entry in `aggregate_metrics_tables` |

> **Rule:** If the customer selects any time window (range or interval), set `apply_time_filter: true` on every source table in `aggregate_metrics_tables`. If the customer says "use all historical data / no filter", leave `apply_time_filter: false` (template default).

### 1g: Scoring Strategy

Ask:

> Three scoring strategies are available — which fits the customer's data best?
>
> - **percentile** — score = percentile rank within the audience. Default for evenly distributed activity.
> - **quartile** — bucket users into 4 evenly-sized groups. Easier to communicate to marketers ("top quartile only").
> - **minmax** — Hivemall min-max scaling. Best when activity distribution is heavily skewed by power users.

Quick rule of thumb: if the customer has a lot of low-frequency users and a small group of very active ones, recommend `minmax`. If they want simple "top 25% only" segmentation, recommend `quartile`. Otherwise default to `percentile`.

This sets `scoring_logic`.

### 1h: Time-of-Day Granularity

Ask:

> The default time-of-day breakdown is four 6-hour periods (morning / afternoon / evening / overnight). Does the customer need finer granularity (e.g., 8 three-hour periods) or shifted boundaries (e.g., morning starts at 5 AM for early-rising audiences)?

This sets `next_best_time.day_breakdown`. Most customers stick with the default — only override if they have a specific operational reason.

### 1i: Timezone Alignment

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

### 1j: Touchpoint Sources

Ask: **Beyond pageviews, which sources of customer behavior should feed the engagement score?**

| Source | Description | Important columns |
|--------|-------------|-------------------|
| `customers` / `user_master` | Distinct profile identifiers | unique identifier used as the join key for the Parent Segment |
| `pageviews` | Web activity from JS SDK or customer's source data | `UNIXTIME` timestamp; UTM availability in URL; conversion-page patterns |
| `orders` | Transaction table | `UNIXTIME` timestamp; `purchase_price`; `order_status` filter |
| `email` | Opens, sends, clicks | `UNIXTIME` timestamp; event-type column for filtering out 'send' |
| _other_ | Sales-rep notes, in-store events, support tickets, etc. | `UNIXTIME` timestamp; profile ID column |

For each source, capture: full table name, primary timestamp column, profile ID column, and any business-rule filter.

### 1k: Conversion Definition

Ask:

> What is the exact conversion event we should use for cart-abandon detection and the conversion flag column?

| Conversion type | Where it lives | `conversion_flag` |
|-----------------|----------------|-------------------|
| Online purchase | pageviews `td_path ~ '/order-received\|/thank-you'` | `IF(REGEXP_LIKE(...), 1.0, 0.0)` on pageviews |
| Lead form submit | pageviews `td_path ~ '/thank\|/download'` | `IF(REGEXP_LIKE(...), 1.0, 0.0)` on pageviews |
| Completed order | orders table with status filter | `1.0` on orders + `order_status IN (...)` filter |
| Subscription signup | subscriptions table | `1.0` on subscriptions |

### 1l: Dashboard Build

Ask: **Should we build the TI dashboard for this customer?**

- If **yes** → `create_dashboard: 'yes'`. Workflow will run `nba_datamodel_create.dig` + `nba_datamodel_build.dig`.
- If **no** → `create_dashboard: 'no'`. The `nba_dash_*` tables will still be populated (the NBA Insights agent depends on them), but the TI datamodel won't be created.

---

## Step 2: Data Source Inventory

After the initial questions, build a per-source inventory. For each touchpoint, capture:

| Field | Pageviews | Orders | Email | Sales/CRM | Other |
|-------|-----------|--------|-------|-----------|-------|
| **Table name** | `<db>.enriched_pageviews` | `<db>.enriched_orders` | `<db>.enriched_email_events` | `<db>.sales_rep_interactions` | `<db>.<table>` |
| **`unixtime_col`** | `time` | `time` | `time` | `time` | ? |
| **Profile ID** | `canonical_id` | `canonical_id` | `canonical_id` | `canonical_id` | ? |
| **URL column** | `td_url` | N/A | N/A | N/A | ? |
| **Referrer column** | `td_referrer` | N/A | N/A | N/A | ? |
| **Channel/source/campaign** | UTM from URL | N/A | `campaign_name`, `email_name` | `source`, `topic` | ? |
| **`conversion_flag`** | URL pattern OR `0.0` | `1.0` (with status filter) | `0.0` | `0.0` | ? |
| **Revenue column** | `0.0` | `unit_price` | `0.0` | `0.0` | ? |
| **Filters needed** | Language, bot exclusion | Status filter | Exclude 'send' events | NULL profile filter | ? |

For each table, verify three things using `tdx-skills:tdx-basic`:
1. The `UNIXTIME` column exists (often `time`; if not, identify the conversion needed: `TD_TIME_PARSE(datetime_col)`)
2. The profile ID column matches `unique_user_id`
3. The conversion logic / channel-extraction columns are present

See `table_configuration.md` for per-source-type discovery queries.

---

## Step 3: Confirm Before YAML Generation

Present the full requirements summary to the user before proceeding to YAML generation:

```
NBA Engagement Scores — Requirements Summary
=============================================

Customer: [name]
Input Tables Database: [input_database_name]
Output Tables Database: [output_database_name]
Profile ID column: [unique_user_id]
Parent Segment for activation: [segment_name]
Confluence folder: [page URL]

Use-Case Scope:
  Type: Engagement scores (Channel / Time / Offer)
  Primary use case: [from question 1c]

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

Once confirmed, proceed to YAML generation in `workflow_setup_guide.md`.
