# NBP — GitHub Instructions

## Repository

```
https://github.com/treasure-data-ps/next_best_product/tree/main/td_wf/nbp_prod
```

## Clone

```bash
git clone https://github.com/treasure-data-ps/next_best_product.git
cd next_best_product/td_wf/nbp_prod
```

## Update Existing Clone

```bash
cd next_best_product
git pull
cd td_wf/nbp_prod
```

## Directory Structure

```
nbp_prod/
├── config/
│   └── params.yml              ← generated configuration (Step 4)
├── queries/
│   ├── create_transaction_table_filters.sql   ← custom training exclusion (when exclusion_values: query)
│   └── filter_products_recs.sql               ← custom recommendation exclusion
├── nbp_launch.dig              ← main workflow entry point
└── ...                         ← workflow SQL and sub-workflows
```
