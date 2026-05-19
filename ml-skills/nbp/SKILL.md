---
name: nbp
description: |
  Next Best Product (NBP) recommendation workflow for Treasure Data. Configures workflows that generate personalized product recommendations using collaborative filtering (Hivemall) or PrecisionML (ml-batch-api). Trigger on: NBP, Next Best Product, product recommendations, recommend products, ALS recommendations, similar_to_latest, popular recommendations, item recommendations, collaborative filtering, Hivemall, DIMSUM similarity, PrecisionML, automl recommendations.
---

# NBP — Next Best Product

Next Best Product workflow that generates personalized product recommendations for every user using collaborative filtering (Hivemall) or PrecisionML (ml-batch-api with ALS, similar_to_latest, or popular algorithms).

## Sub-Folder Routing

| Task | Action |
|------|--------|
| Configure the NBP workflow (`config/params.yml`) | Read `workflow-setup/SKILL.md` |
| Production docs, output tables, operational runbook | Read `prod-docs/SKILL.md` |
| Build a companion LLM agent for NBP analysis | Read `agent-skills/SKILL.md` |

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/next_best_product/tree/main/td_wf/nbp_prod`
- **Workflow path**: `next_best_product/td_wf/nbp_prod/`
- **Config file**: `nbp_prod/config/params.yml`
- **Key output**: Per-user product recommendations (nbp_1, nbp_2, ... columns) for Parent Segment ingestion

## How It Works

1. **Ingest**: Standardize transaction table (user + item + timestamp) with exclusion filters
2. **Map**: Build item-to-name/category mapping from item info table
3. **Train**: Run collaborative filtering (hive: DIMSUM similarity) or PrecisionML (automl: ALS/similar_to_latest/popular)
4. **Predict**: Generate top-K recommendations per user
5. **Enrich**: Join item names/categories, explode into per-rank columns (nbp_1, nbp_1_sku, nb_category_1, ...)
6. **Dashboard**: Create TI datamodel with model stats, test set evaluation, and recommendation distribution

