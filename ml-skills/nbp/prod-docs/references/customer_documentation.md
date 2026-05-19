# Phase 5: Customer Documentation

Create Confluence pages in the customer's folder with deployment documentation. Uses the **customer folder page ID** resolved in Phase 1 as the parent for all new pages.

## Confluence Setup

### Prerequisites

- **NBP sub-folder page ID**: Resolved in Phase 1 (requirements-gathering.md, Steps 1, 1b, and 1c)
- **CUST space ID**: `9797636`
- **Cloud ID**: `treasure-data.atlassian.net`

### Page Structure

Create documentation pages directly under the NBP sub-folder:

```
[Customer Folder]
└── FDE Solutions (or ML & Analytics Projects, etc.)
    └── Next Best Product (NBP)            ← existing or created in Phase 1
        ├── NBP Configuration Summary      ← child page
        ├── NBP Architecture & Output Schema  ← child page
        └── NBP Runbook & Maintenance      ← child page
```

---

## Step 1: Create the Deployment Summary Page

Create the main NBP deployment summary directly under the NBP sub-folder.

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <nbp_subfolder_page_id>
  title: "NBP Deployment Summary"
  contentFormat: markdown
  body: <see Executive Summary template below>
```

### Executive Summary (Parent Page Body)

Use this as the body content for the parent page. Fill in all `[placeholders]` with actual values from the deployment.

```markdown
# NBP Deployment Summary

We deployed a Next Best Product (NBP) recommendation workflow for [customer name] using the **[hive/automl]** engine.

## Key Facts

| | |
|---|---|
| **Source data** | `[in_db].[original_item_transactions_table]` |
| **Users** | [X] distinct users |
| **Items** | [Y] distinct items |
| **Total interactions** | [Z] rows |
| **Model type** | [hive / automl ([algorithm])] |
| **Recommendations per user** | [max_recommended_items] |
| **Output table** | `[sink_database].[user_master_attr_table]` |
| **Dashboard** | [yes/no] |
| **TD Site** | [site] |
| **Deployed** | [date] |

## Output Columns

The final table `[user_master_attr_table]` contains:

| Column | Description |
|--------|-------------|
| `[user_id_column]` | User identifier |
| `nbp_1` | Top-1 recommended product name |
| `nbp_1_sku` | Top-1 recommended product SKU |
| `nb_category_1` | Top-1 recommended product category |
| `nbp_2`, `nbp_2_sku`, `nb_category_2` | Top-2 recommendation |
| ... | up to `nbp_[N]` |
| `product_recommendations` | Full array of recommended product names |
| `product_skus` | Full array of recommended product SKUs |
| `product_categories` | Full array of recommended product categories |

## Pages

- **Configuration Summary** — Source tables, column mappings, model parameters, exclusion filters
- **Architecture & Output Schema** — Data flow, intermediate tables, output schema
- **Runbook & Maintenance** — How to run, modify, troubleshoot, and maintain
```

**Save the created page ID** — use it as `parentId` for the child pages below.

---

## Step 2: Create Configuration Summary Page

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <nbp_subfolder_page_id>
  title: "NBP Configuration Summary"
  contentFormat: markdown
  body: <see template below>
```

### Configuration Summary Template

```markdown
# NBP Configuration Summary

## Source Tables

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Database | `[in_db]` | Customer's [golden/staging/production] database |
| Transaction table | `[original_item_transactions_table]` | Contains item-level purchase history ([X] rows, [Y] users, [Z] items) |
| Item info table | `[original_item_info_table]` | [Same table as transactions / Separate catalog table] |

## Column Mappings

| Role | Column Name | Source Table |
|------|-------------|-------------|
| User ID | `[original_user_id_column_name]` | Transaction table |
| Item ID | `[original_item_id_column_name]` | Transaction table |
| Item Name | `[original_item_name_column_name]` | Item info table |
| Item Category | `[original_item_category_column_name]` | Item info table |
| Timestamp | `[original_timestamp_name]` | Transaction table |

## Model Configuration

| Parameter | Value | Notes |
|-----------|-------|-------|
| Model type | `[hive/automl]` | [Reason for choice] |
| Algorithm | `[als/similar_to_latest/popular]` | [Reason] (automl only) |
| Recommendations per user | `[max_recommended_items]` | |
| Orders lookback | `[orders_lookback]` | [All history / last N days] |
| ALS sample rate | `[als_sample_rate]` | (automl+ALS only) |
| ALS max users | `[als_max_users]` | (automl+ALS only) |
| Filter viewed | `[filter_viewed]` | (automl only) |

### Hive Engine Parameters (if model_type = hive)

| Parameter | Value |
|-----------|-------|
| `topk_similar_items` | [value] |
| `recent_items` | [value] |
| `max_recommended_items` | [value] |
| `min_cooccurence_filter` | [value] |
| `dimsum_similarity_threshold` | [value] |

## Exclusion Filters

| Filter | Method | Details |
|--------|--------|---------|
| Training exclusion | [regex/query] | [What was excluded and why] |
| Recommendation exclusion | [regex/query/none] | [What was excluded and why] |

[If using custom SQL, include the SQL logic here]

## Dashboard

| Parameter | Value |
|-----------|-------|
| Create dashboard | `[yes/no]` |
| API endpoint | `[api_endpoint]` |
| Workflow secret | `secret_key` is set (do not include the actual key) |

## Output Tables

| Table | Description |
|-------|-------------|
| `[sink_database].[user_master_attr_table]` | Final recommendations with per-rank columns |
| `[sink_database].[prefix]transaction` | Standardized transaction table |
| `[sink_database].[prefix]topk_recommended_items` | Raw recommendation arrays |
| `[sink_database].[prefix]dash_*` | Dashboard aggregation tables |
```

---

## Step 3: Create Architecture & Output Schema Page

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <nbp_subfolder_page_id>
  title: "NBP Architecture & Output Schema"
  contentFormat: markdown
  body: <see template below>
```

### Architecture Template

```markdown
# NBP Architecture & Output Schema

## Data Flow

1. **Source**: `[in_db].[transaction_table]`
   - Raw item-level transactions (user + item + timestamp)

2. **Preprocessing**: `[prefix]transaction`
   - Columns standardized to `userid`, `itemid`, `tstamp`
   - Exclusion filters applied: [describe what was filtered]
   - Time filter: [all history / last N days]

3. **Model Training** ([hive/automl]):
   [For hive]:
   - Co-occurrence matrix computed from user-item interactions
   - DIMSUM algorithm calculates approximate cosine similarity between items
   - Top [topk_similar_items] similar items retained per item

   [For automl]:
   - Data sent to ml-batch-api (`[ml_batch_api_endpoint]`)
   - [ALS/similar_to_latest/popular] algorithm trained and predictions generated

4. **Prediction**: `[prefix]topk_recommended_items`
   - [max_recommended_items] recommendations generated per user
   - Raw output: `userid` + `rec_items` array

5. **Enrichment**: `[user_master_attr_table]`
   - Item names and categories joined from `[original_item_info_table]`
   - Per-rank columns exploded: `nbp_1`, `nbp_1_sku`, `nb_category_1`, `nbp_2`, ...

6. **Dashboard**: `[prefix]dash_*` tables
   - Aggregation tables for TD Insights visualization
   - Test set evaluation results ([enabled/disabled])

## Intermediate Tables

All tables use prefix `[prefix]` in database `[sink_database]`.

| Table | Engine | Description |
|-------|--------|-------------|
| `[prefix]transaction` | Both | Standardized transactions |
| `[prefix]itemid_name_product_mapping` | Both | Item ID to name/category mapping |
| `[prefix]user_item` | Both | Aggregated user-item interactions |
| `[prefix]item_similarity` | Hive only | Item-item DIMSUM similarity scores |
| `[prefix]topk_similar_items` | Hive only | Top-K neighbors per item |
| `[prefix]recent_item_contacts` | Hive only | Each user's recent purchases |
| `[prefix]topk_recommended_items` | Both | Raw recommendations |
| `[user_master_attr_table]` | Both | Final output with per-rank columns |
| `[prefix]dash_*` | Both | Dashboard tables |
| `[prefix]testset` | Both | Test set results (if enabled) |

## Final Output Schema

| Column | Type | Description |
|--------|------|-------------|
| `[user_id_column]` | VARCHAR | User identifier |
| `nbp_ranks` | ARRAY | Rank indices |
| `product_recommendations` | ARRAY | Ordered recommended product names |
| `product_categories` | ARRAY | Ordered product categories |
| `product_skus` | ARRAY | Ordered product SKUs |
| `nbp_1` | VARCHAR | Top-1 recommended product name |
| `nbp_1_sku` | VARCHAR | Top-1 recommended product SKU |
| `nb_category_1` | VARCHAR | Top-1 recommended category |
| ... | ... | Up to nbp_[max_recommended_items] |
```

---

## Step 4: Create Runbook & Maintenance Page

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <nbp_subfolder_page_id>
  title: "NBP Runbook & Maintenance"
  contentFormat: markdown
  body: <see template below>
```

### Runbook Template

````markdown
# NBP Runbook & Maintenance

## Running the Workflow

### Manual Run

```
tdx wf run next_best_product.nbp_launch
```

### Monitoring

```
# List recent sessions
tdx wf sessions next_best_product

# View logs for a specific attempt
tdx wf logs <attempt_id>
```

### Scheduling

Configure recurring runs in the TD console or via CLI:
```
tdx wf schedule --set nbp_launch --cron "0 6 * * 1"
```

## Modifying Parameters

1. Edit `config/params.yml` in the workflow project directory
2. Push changes: `tdx wf push -y`
3. Re-run the workflow

### Common Changes

| Change | Parameter to Edit |
|--------|-------------------|
| Add/remove exclusion filters | `exclusion_values` or custom SQL files |
| Change lookback period | `orders_lookback` |
| Change number of recommendations | `max_recommended_items` |
| Switch algorithm (automl) | `automl_model_type` |

### Updating Source Data

If the source table name or column names change:
1. Update the corresponding parameters in `config/params.yml`
2. Push and re-run

## Re-running After Failure

```
# Check what failed
tdx wf logs <attempt_id>

# Retry from the failed task
tdx wf retry <attempt_id>

# Or start a fresh run
tdx wf run next_best_product.nbp_launch
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Workflow fails at transaction table creation | Column names changed in source table | Verify with `DESCRIBE` and update `params.yml` |
| Automl API returns 401/403 | Expired or missing API key secret | `tdx wf secrets --set secret_key=<NEW_KEY>` |
| Dashboard not updating | API endpoint mismatch or stale secret | Verify `dash_params.api_endpoint` and re-set secret |
| User master table has fewer users than expected | Exclusion filters too aggressive | Review exclusion settings, check filter impact |
| Test set shows zero matches | Holdout period mismatch | Adjust `test_params.test_lookback` |
| Recommendations are all the same item | Too few items or threshold too high | Check item count, adjust `dimsum_similarity_threshold` (hive) |

## Maintenance

### When to Retrain

- **Weekly or bi-weekly** is typical for most e-commerce use cases
- More frequently if the product catalog changes rapidly
- Less frequently for stable catalogs (monthly)

### When to Update Configuration

- New product categories added that should be excluded
- Source table schema changes (column renames)
- Business requirements change (more/fewer recommendations per user)
- Switching from hive to automl or vice versa

### Monitoring

- Check the dashboard after each run for coverage and test set metrics
- Alert on workflow failures via the `_error` email notification
- Monitor `users_with_recs` trend — a sudden drop may indicate data issues

### API Key Rotation

If the TD API key is rotated:
```
tdx wf secrets --set secret_key=<NEW_API_KEY>
```

### Updating the Workflow Code

To pull the latest workflow code:
```
cd next_best_product
git pull
cd td_wf/nbp_prod
# Re-apply your config/params.yml
tdx wf push -y
```
````

---

## Delivery Checklist

Before handing off to the customer, confirm:

- [ ] All three Confluence pages created under customer folder
- [ ] All `[placeholders]` replaced with actual values
- [ ] Configuration summary matches the deployed `params.yml`
- [ ] Architecture overview matches actual data flow
- [ ] Runbook commands are tested and working
- [ ] Dashboard is accessible and shows correct data
- [ ] Troubleshooting guide covers the most common failure modes
- [ ] Maintenance schedule is discussed and agreed
- [ ] Customer team has access to the workflow project in TD
- [ ] Customer team knows how to re-run and modify parameters
- [ ] API key secret is set and documented (not the actual key)
