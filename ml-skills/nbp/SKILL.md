---
name: fde-nbp
description: |
  Next Best Product (NBP) recommendation workflow for Treasure Data. Configures workflows that generate personalized product recommendations using collaborative filtering (Hivemall) or PrecisionML (ml-batch-api). Trigger on: NBP, Next Best Product, product recommendations, recommend products, ALS recommendations, similar_to_latest, popular recommendations, item recommendations, collaborative filtering, Hivemall, DIMSUM similarity, PrecisionML, automl recommendations, requirements gathering for NBP, NBP workflow setup, NBP configuration, NBP runbook.
---

# NBP — Next Best Product

Next Best Product workflow that generates personalized product recommendations for every user using collaborative filtering (Hivemall) or PrecisionML (ml-batch-api with ALS, similar_to_latest, or popular algorithms). Output is shaped for direct ingestion into Audience Studio as Parent Segment attribute columns.

## Pick the right reference for the task

| If the user wants to... | Read |
|---|---|
| Gather requirements before configuring (Confluence folder, transaction table, item info, exclusion rules) | `workflow-setup/references/requirements_doc.md` |
| Configure the workflow / generate `params.yml` end-to-end | `workflow-setup/references/workflow_setup_guide.md` |
| Look up YAML parameter reference | `workflow-setup/references/yaml_structure.md` |
| Configure source tables (transactions / item info) | `workflow-setup/references/table_configuration.md` |
| See a working example config | `workflow-setup/references/input_params_template.yml` |
| Decide between `hive` and `automl` engines | `workflow-setup/references/gap_analysis.md` |
| Clone the repo / push to GitHub / set secrets | `workflow-setup/references/github_instructions.md` |
| Production runbook — architecture, output tables, monitoring, failure modes, dashboard | `prod-docs/references/runbook.md` |
| Customer documentation template | `prod-docs/references/customer_documentation.md` |
| Visualization / dashboard guidance | `prod-docs/references/visualizations.md` |
| Workflow deployment specifics | `prod-docs/references/workflow_deployment.md` |

## Companion agent skills

NBP does not currently ship a companion Foundry agent or Treasure Work analysis skill. Scaffold only — when one is built, it'll be registered as a separate skill (e.g. `fde-nbp-foundry-skill` or `fde-nbp-insights-agent`).

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/next_best_product/tree/main/td_wf/nbp_prod`
- **Workflow path**: `next_best_product/td_wf/nbp_prod/`
- **Config file**: `nbp_prod/config/params.yml`
- **Engines**: `hive` (Hivemall DIMSUM collaborative filtering) or `automl` (ml-batch-api: ALS / similar_to_latest / popular)
- **Key output**: Per-user product recommendations (`nbp_1`, `nbp_2`, ... columns) for Parent Segment ingestion

## How It Works

1. **Ingest**: Standardize transaction table (user + item + timestamp) with exclusion filters
2. **Map**: Build item-to-name/category mapping from item info table
3. **Train**: Run collaborative filtering (hive: DIMSUM similarity) or PrecisionML (automl: ALS / similar_to_latest / popular)
4. **Predict**: Generate top-K recommendations per user
5. **Enrich**: Join item names/categories, explode into per-rank columns (`nbp_1`, `nbp_1_sku`, `nb_category_1`, ...)
6. **Dashboard**: Create TI datamodel with model stats, test set evaluation, and recommendation distribution

## Engine Decision

| Consideration | `hive` | `automl` |
|---|---|---|
| API dependency | None — runs on Hive | Requires ml-batch-api |
| Algorithm control | Full tuning (similarity threshold, K neighbors) | Algorithm selection only |
| Cold-start handling | No built-in fallback | `popular` algorithm covers all users |
| Already-purchased filtering | Built into SQL (`NOT EXISTS`) | `filter_viewed` toggle |
| Compute | Hive cluster | ml-batch-api service |

For deeper trade-offs see `workflow-setup/references/gap_analysis.md`.

## Standard Workflow

When a user asks to set up NBP from scratch, follow this sequence:
1. Walk through `workflow-setup/references/requirements_doc.md` to gather inputs (transaction table, item info, exclusion rules, model engine choice) and locate the customer's Confluence folder
2. Use `tdx-skills:tdx-basic` to explore the customer's TD database, verify item-level granularity, and confirm the item-info join works
3. Generate `params.yml` using `workflow-setup/references/workflow_setup_guide.md`
4. Present the YAML to the user, get confirmation, set the `secret_key` workflow secret, then push via `tdx wf upload next_best_product`
5. After deploy, document the configuration on Confluence under the customer's NBP sub-folder
