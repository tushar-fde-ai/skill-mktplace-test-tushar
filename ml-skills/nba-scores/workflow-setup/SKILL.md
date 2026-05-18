---
name: nba-scores-workflow-setup
description: |
  NBA (Next Best Action) Engagement Scores workflow configuration for Treasure Data. Use this skill to configure NBA engagement-scoring workflows that produce per-profile Next Best Channel, Next Best Time, Next Best Campaign affinity scores plus cart-abandon and new-visitor flags. Trigger when users mention NBA, Next Best Action, engagement scores, channel affinity, time of day affinity, cart abandon, new visitor, or activating engagement scores into Audience Studio.
---

# NBA Engagement Scores — Workflow Setup

This skill walks you through configuring and deploying the NBA Engagement Scores workflow in Treasure Data. The workflow unions customer behavioral data from multiple sources, derives per-profile engagement scores, and writes a single output table that can be joined to a Parent Segment in Audience Studio.

## What is NBA Engagement Scoring?

The NBA workflow produces, for each unique profile in the customer's data:

- **Next Best Channel score(s)** — affinity per channel (social, email, search, display, etc.). Built from UTM parsing + channel regex rules.
- **Next Best Time score(s)** — affinity per daypart (morning / afternoon / evening / overnight by default).
- **Cart Abandon flag** — boolean: did this profile add to cart in the last N days without a conversion?
- **New Visitor flag** — boolean: is this profile recently first-seen, with low page-visit count, no paid-traffic referral, and no purchase?

All scores combine into `nba_combined_metrics_final`. Scoring strategy is configurable: `percentile`, `quartile`, or `minmax` (Hivemall).

**Key outputs:**
- `nba_combined_metrics_final` — final per-profile NBA scores joinable to Parent Segment
- `nba_dash_stats_summary` / `nba_dash_model_metrics` / `nba_dash_source_tables` — dashboard tables (consumed by both the TI dashboard and the NBA Insights agent)
- Optional TI dashboard: `nba_engagement_scores_automated`

## Use Cases

- Picking the daypart with highest engagement to schedule sends
- Shifting marketing budget toward each user's top-affinity channel
- Triggering cart-abandon journeys
- Suppressing new-visitor audiences from aggressive ads / retargeting
- Building "high-engagement on social" or "evening-active responders" segments for activation

## NBA Configuration Workflow

### Step 1: Gather Requirements

**Reference**: Read `references/requirements_doc.md`. It walks you through the Confluence-folder location flow, then 12 NBA-specific questions covering ID strategy, parent segment, use case scope, cart-abandon / new-visitor windows, scoring strategy, time-of-day granularity, timezone, touchpoint sources, conversion definition, and dashboard build.

The questions you MUST answer before generating YAML:

1. What is the customer's TD database for source tables and where should outputs be written? (`sink_database`, `unique_user_id`)
2. What Parent Segment will the scores be activated into?
3. Engagement-score use case (Channel / Time / Offer) or Next Best Product? — if NBP, route to the NBP skill.
4. Cart-abandon and new-visitor windows.
5. Time filter: `range` or `interval`, what window?
6. Scoring strategy: `percentile`, `quartile`, or `minmax`?
7. Time-of-day granularity (default 4 buckets, or custom)?
8. Customer's marketing-operations timezone?
9. Which touchpoint sources to union (pageviews, email, sales, orders, custom)?
10. Conversion definition?
11. Build the TI dashboard (yes/no)?

### Step 2: Explore the Customer's Database

Use **Trino SQL** via **tdx-skills** to discover the source tables.

NBA expects these touchpoint types — look for tables matching each pattern:

#### Web Activity (pageviews, site visits)
- Tables like: `enriched_pageviews`, `web_events`, `page_views`
- Needed columns: `time`, `unique_user_id`, URL, referrer, UTM params (source/medium/campaign)
- **Purpose**: Channel signals (UTM-driven) + time signals + conversion-pattern matching

#### Email Events
- Tables like: `enriched_email_events`, `email_activity`
- Needed columns: `time`, `unique_user_id`, `event_type`, `campaign_name`, `email_name`
- **Purpose**: Email-channel touchpoints. Critical: filter out 'send' events.

#### Sales Rep / CRM Interactions
- Tables like: `sales_rep_interactions`, `crm_activity`
- Needed columns: `time`, `unique_user_id`, source, topic
- **Purpose**: Offline / human-touch channel touchpoints

#### Orders / Conversions
- Tables like: `enriched_orders`, `purchases`, `transactions`
- Needed columns: `time`, `unique_user_id`, `order_type`, `order_status`, revenue column
- **Purpose**: Conversion events (`conversion_flag: 1.0`) — defines the goal cart-abandon and new-visitor rules check against.

For each table, run:
```sql
DESCRIBE <database>.<table>;
SELECT * FROM <database>.<table> LIMIT 10;
```

For pageviews, also check UTM coverage:
```sql
SELECT
  COUNT(*) AS total_rows,
  COUNT(CASE WHEN url_extract_parameter(td_url, 'utm_source') IS NOT NULL THEN 1 END) AS has_utm_source,
  COUNT(CASE WHEN url_extract_parameter(td_url, 'utm_medium') IS NOT NULL THEN 1 END) AS has_utm_medium,
  COUNT(CASE WHEN url_extract_parameter(td_url, 'utm_campaign') IS NOT NULL THEN 1 END) AS has_utm_campaign
FROM <database>.enriched_pageviews
WHERE TD_INTERVAL(time, '-90d');
```

If UTM coverage is below ~30%, flag this to the user — channel scoring quality will be limited.

### Step 3: Clone the NBA Workflow Repository

```bash
git clone https://github.com/treasure-data-ps/nba_eng_scores.git
cd nba_eng_scores/td_wf
```

For full repo structure and deployment commands, read `references/github_instructions.md`.

### Step 4: Generate `input_params.yml`

The critical file to generate is `td_wf/config/input_params.yml`.

**References**:
- `references/yaml_structure.md` — full parameter reference (every section, every field, every type)
- `references/input_params_template.yml` — working filled-in example to copy from
- `references/table_configuration.md` — per-source-type configuration (pageviews / email / sales / orders / custom)

#### Global Parameters

```yaml
sink_database: <customer_database>
unique_user_id: canonical_id
api_endpoint: 'api.treasuredata.com'
model_config_table: 'datamodel_build_history'
create_dashboard: 'yes'
cleanup_temp_tables: 'yes'
prefix: 'nba'
```

Set `api_endpoint` to `api.eu01.treasuredata.com` for EU accounts, `api.ap02.treasuredata.com` for AP02, etc.

#### Filter Parameters

```yaml
time_filter_type: interval                      # or 'range'
time_range_start_date: 2022-01-01
time_range_end_date: 2222-22-22                 # 2222-22-22 = use latest available
lookback_period: -180d                          # only used with 'interval'
session_length: 3600
backfill_partition_col: ${unique_user_id}
top_k_channel_perc: 0.001
top_k_conv_perc: 0.001
time_zone: 'UTC'
timeshift_change: '+'
timeshift_hours: 0
scoring_logic: 'quartile'                       # 'percentile' | 'quartile' | 'minmax'
topk_values: 10
```

#### Output Tables

```yaml
src_union_table: nba_combined_user_events
final_nba_metrics_table: nba_combined_metrics_final
```

Rename only if the customer has a naming-convention requirement.

### Step 5: Configure Source Tables (`aggregate_metrics_tables`)

Each table in `aggregate_metrics_tables` has a consistent column shape regardless of source type — fill the unused ones with `CAST(NULL AS VARCHAR)` or hardcoded labels.

**Reference**: `references/table_configuration.md` has the full per-source-type guidance with discovery SQL, configuration templates, and key decisions.

Quick summary:

#### Pageviews (web touchpoints + conversion patterns)

```yaml
- src_table: <customer_database>.enriched_pageviews
  name: 'pageviews'
  unixtime_col: time
  join_key: ${unique_user_id}
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
```

Key decisions: customize `conversion_flag` regex to match the customer's conversion URL patterns; add bot/language exclusions to `custom_filter`.

#### Email Events (channel touchpoints, never conversions)

```yaml
- src_table: <customer_database>.enriched_email_events
  name: 'email_events'
  unixtime_col: time
  join_key: ${unique_user_id}
  url_col: CAST(NULL AS VARCHAR)
  referral_col: "''email''"
  medium_col: "''email''"
  source_col: "''<esp_name>''"                 # 'sfmc' | 'marketo' | 'braze' | 'hubspot'
  campaign_col: "REGEXP_REPLACE(lower(campaign_name), ''[- ]'', ''_'')"
  context_col: "REGEXP_REPLACE(lower(email_name), ''[- ]'', ''_'')"
  event_type: "CONCAT(''email_'', event_type)"
  custom_filter: "${unique_user_id} IS NOT NULL AND NOT REGEXP_LIKE(lower(event_type), ''send'')"
  conversion_flag: 0.0
  item_price: 0.0
  apply_time_filter: false
```

Critical: always exclude `'send'` events. Keep `conversion_flag: 0.0`.

#### Sales / CRM Interactions

```yaml
- src_table: <customer_database>.sales_rep_interactions
  name: 'sales_rep_interactions'
  unixtime_col: time
  join_key: ${unique_user_id}
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
```

#### Orders (conversion events)

```yaml
- src_table: <customer_database>.enriched_orders
  name: 'order_events'
  unixtime_col: time
  join_key: ${unique_user_id}
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
```

Critical: `conversion_flag: 1.0`, always filter by valid `order_status`, watch for line-item duplication inflating counts.

### Step 6: Configure NBA-Specific Sections

These four sections are NOT in MTA or RFM templates — they're NBA-specific:

#### `next_best_time` (default daypart breakdown)

```yaml
next_best_time:
  outputs:
    database: ${sink_database}
    union_table: nba_combined_user_events_final
    activity_period_column: activity_period
    day_breakdown:
      - period: morning
        logic: time_hour >= 6 AND time_hour <= 11
      - period: afternoon
        logic: time_hour >= 12 AND time_hour <= 17
      - period: evening
        logic: time_hour >= 18 AND time_hour <= 23
      - period: overnight
        logic: time_hour >= 0 AND time_hour <= 5
```

Override `day_breakdown` if the customer wants finer granularity.

#### `utm_parcing` (channel parsing)

```yaml
utm_parcing:
  date_field: time
  channel_column: channel
  source_column: source
  exclude_channel_regexp: '%|!|@|#|\(|\)'
  min_percent: 0.4
  outputs:
    union_table: nba_combined_user_events_final
```

Tune `min_percent` if the customer has many low-frequency channels you want to keep.

#### `next_best_campaign` (cart-abandon + new-visitor + ad-engagement rules)

```yaml
next_best_campaign:
  database: ${sink_database}
  src_table: nba_combined_user_events_final
  join_key: ${unique_user_id}
  date_field: time
  abandon_regexp: REGEXP_LIKE(lower(event_type), '(?=.*add)(?=.*cart)')
  abandon_regexp_string: REGEXP_LIKE(lower(event_type), ''(?=.*add)(?=.*cart)'')
  conversion_logic: conversion_flag > 0
  ad_engagement_logic: channel IN ('paid search', 'tiktok', 'web ads', 'dsp_rtb', 'maddict', 'call center', 'others', 'display')
  ad_engagement_logic_string: channel IN (''paid'', ''search'', ''tiktok'', ''web ads'', ''dsp_rtb'', ''maddict'', ''call center'', ''others'', ''display'')
  event_lookback: 90                            # cart-abandon lookback days
  new_customers_days: 45                        # new-visitor max age
  max_number_visits: 15                         # new-visitor max page visits
  custom_flags_table: nba_custom_campaign_flags
```

**`abandon_regexp` and `abandon_regexp_string`** must stay in sync. Update both if the customer's add-to-cart event is named differently.

**`event_lookback`, `new_customers_days`, `max_number_visits`** — these are the values you collected in requirements gathering 3e.

#### `temporary_tables`

```yaml
temporary_tables:
  next_best_channel: nba_next_best_channel
  next_best_time: nba_next_best_time
  next_best_campaign: nba_next_best_campaign
  cart_abandon: nba_cart_abandon
  new_visitors: nba_new_visitor_no_ads
  regexp_columns: 'visitor|aban|perc|minmax'

  table_list:
    - nba_next_best_time
    - nba_next_best_channel
    - nba_next_best_campaign
    - nba_cart_abandon
    - nba_new_visitor_no_ads
    - schema
```

Most customers can leave this section at template defaults.

### Step 7: Validate the Configuration

Before finalizing, verify:

1. All table names exist in the database (run `DESCRIBE` on each).
2. Column names are correct for each table.
3. `unique_user_id` is consistent across every source table.
4. `conversion_flag` correctly identifies conversion events (run a `COUNT(*)` against the regex/filter).
5. `custom_filter` excludes invalid data (bots, language, sends, cancelled orders).
6. UTM coverage on pageviews is non-trivial (>30% rows have at least one UTM param).
7. `sink_database` exists and the user has write permissions.
8. `abandon_regexp` matches actual cart-abandon events in the data.
9. `scoring_logic` matches the customer's data distribution and use-case preference.

### Step 8: Present to User for Approval

Show the user:

1. **Discovered tables** and their role (touchpoint vs conversion)
2. **Conversion definition** — what triggers `conversion_flag: 1.0`
3. **Scoring strategy** — `percentile` / `quartile` / `minmax` and why
4. **Business-rule windows** — `event_lookback`, `new_customers_days`, `max_number_visits`
5. **Time-of-day buckets** — default 4-bucket or custom
6. **The complete `input_params.yml`**

### Step 9: Present Final YAML and Confirm

Always present the complete `input_params.yml` to the user for review before deploying.

Show the full YAML and ask:

> Here is the final `input_params.yml` that will be deployed. Please review and confirm.

**Wait for explicit confirmation before proceeding.**

After confirmation, ask:

> Would you like to push this workflow with the default project name `nba_eng_prod`, or would you like to provide a custom project name?

If the user provides a custom name, use it as the workflow project name in the next step.

### Step 10: Deploy

Once both YAML and project name are confirmed:

1. Place `input_params.yml` in `nba_eng_scores/td_wf/config/input_params.yml`
2. From inside `nba_eng_scores/td_wf`:
   - **Default name**: `tdx wf push -y` (uses `nba_eng_prod` from `tdx.json`)
   - **Custom name**: `tdx wf upload <custom_project_name>`
3. Run the workflow: `tdx wf run nba_eng_prod.nba_eng_launch`
4. Monitor: `tdx wf sessions nba_eng_prod` and `tdx wf timeline nba_eng_prod.nba_eng_launch --follow`

For full deployment details (auth, scheduling, dashboard upload, common failure modes), read `references/github_instructions.md`.

## Critical Configuration Rules

### Conversion Flag Logic

Every source table needs a `conversion_flag`. Only conversion-event tables should use `1.0`:

| Source type | `conversion_flag` | Reasoning |
|-------------|-------------------|-----------|
| Pageviews | `IF(REGEXP_LIKE(lower(td_path), 'thank\|download\|order-received'), 1.0, 0.0)` | Only specific URL patterns count |
| Email | `0.0` | Touchpoints only |
| Sales | `0.0` | Touchpoints only |
| Orders | `1.0` | Orders ARE the conversion |

### `abandon_regexp` Sync

The two copies must stay identical (one for raw SQL, one for interpolated SQL with doubled quotes):

```yaml
abandon_regexp: REGEXP_LIKE(lower(event_type), '(?=.*add)(?=.*cart)')
abandon_regexp_string: REGEXP_LIKE(lower(event_type), ''(?=.*add)(?=.*cart)'')
```

### Common Mistakes to Avoid

**1. Missing channel columns for non-web tables**
```yaml
# Use CAST(NULL AS VARCHAR) when a channel field doesn't apply
url_col: CAST(NULL AS VARCHAR)
medium_col: "''email''"         # hardcode the channel name instead
```

**2. Not excluding email 'send' events**
```yaml
# BAD — sends inflate touchpoint counts and skew time-of-day scoring
custom_filter: "${unique_user_id} IS NOT NULL"
# GOOD
custom_filter: "${unique_user_id} IS NOT NULL AND NOT REGEXP_LIKE(lower(event_type), ''send'')"
```

**3. Including cancelled orders as conversions**
```yaml
# BAD
custom_filter: "${unique_user_id} IS NOT NULL"
# GOOD
custom_filter: ${unique_user_id} IS NOT NULL AND order_status IN (''COMPLETE'', ''PROCESSING'')
```

**4. Wrong `conversion_flag` on touchpoint tables**
```yaml
# BAD — email is NOT a conversion
conversion_flag: 1.0
# GOOD
conversion_flag: 0.0
```

**5. `abandon_regexp` doesn't match the customer's cart event**
If the customer logs add-to-cart as `basket_add` or `cart_item_added` instead of containing both 'add' and 'cart', update both regex copies.

## Progressive Disclosure

- **Complete YAML structure**: Read `references/yaml_structure.md`
- **Per-source-type configuration**: Read `references/table_configuration.md`
- **Working example**: Read `references/input_params_template.yml`
- **Requirements gathering**: Read `references/requirements_doc.md`
- **GitHub clone & deploy**: Read `references/github_instructions.md`

## GitHub Repository

The production NBA workflow code is at:
```
https://github.com/treasure-data-ps/nba_eng_scores
```

After generating `input_params.yml`, place it in `nba_eng_scores/td_wf/config/input_params.yml`.
