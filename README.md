# Genie & Compute Cost Dashboard

An AI/BI (Lakeview) dashboard, packaged as a Databricks Asset Bundle, that shows what Databricks Genie costs. It splits the cost into two parts:

- **LLM tokens (billed):** actual pay-as-you-go Genie token cost from `system.billing.usage` (`billing_origin_product = 'GENIE'`), broken out by surface (Genie Agents, Genie Code, Genie One) and channel.
- **Compute (SQL warehouse, estimated):** each SQL warehouse's daily cost, allocated to Genie users by their share of query task time in `system.query.history`.

## Pages

| Page | What it shows |
|------|---------------|
| Cost Overview | KPIs and trends by cost component, surface, Genie space, warehouse, and user |
| Compute vs LLM Tokens | Side-by-side comparison of the two cost components |
| User Genie Cost | Per-user cost split by Genie Agents / Genie Code / Genie One, with totals |

All queries cover a rolling 30-day window and use list prices from `system.billing.list_prices`, with no discounts applied.

## Prerequisites

- Unity Catalog with system tables enabled. The person who runs the dashboard needs `SELECT` on:
  - `system.billing.usage`
  - `system.billing.list_prices`
  - `system.query.history`
- A SQL warehouse (serverless recommended)
- Databricks CLI v0.250+ (`databricks bundle` commands)

No custom tables, catalogs, or schemas are needed.

## Deploy

```bash
git clone https://github.com/susan-farjami-db/genie-compute-cost-dashboard.git
cd genie-compute-cost-dashboard

databricks bundle validate --var="warehouse_id=<your-warehouse-id>" --profile <your-profile>
databricks bundle deploy   --var="warehouse_id=<your-warehouse-id>" --profile <your-profile>
```

To deploy a shared copy instead of a per-user dev copy, add `-t prod`. You can also set `warehouse_id` in a `targets:` override in `databricks.yml`, or with the `BUNDLE_VAR_warehouse_id` environment variable.

The dashboard runs with viewer credentials (`embed_credentials: false`), so each viewer only sees the system-table data they have access to.

## Notes and limitations

- **Compute cost is an estimate.** Warehouse cost is allocated by task-time share, counting only users who have Genie billing activity. Non-Genie queries from those same users on the same warehouse can inflate the estimate.
- **Token billing started 2026-07-06.** Until then the LLM token half shows $0, and in any workspace without Genie token usage it still reads $0.
- Costs use list price. To reflect a negotiated discount, multiply the `cost_usd` expressions in the dataset SQL by your discount factor.

## Repo layout

```
databricks.yml                              # bundle + targets + warehouse_id variable
resources/genie_compute_cost.dashboard.yml  # dashboard resource
src/genie_compute_cost.lvdash.json          # dashboard definition
```
