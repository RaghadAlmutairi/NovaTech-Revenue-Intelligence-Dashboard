# Revenue Intelligence Dashboard for NovaTech Solutions

A multi-page, interactive business intelligence dashboard built in **Amazon QuickSight**, developed as a Udacity Data Analytics capstone project. The project simulates a real-world engagement for a fictional VP of Revenue Operations at NovaTech Solutions, unifying sales, marketing, and support data into a single decision-making tool.

## Project Overview

NovaTech Solutions needed a way to connect three previously siloed data sources — sales pipeline, marketing campaigns, and customer support — into one view that reveals how deals, leads, and tickets relate to the same accounts. This project builds that unified model and layers a three-sheet interactive dashboard plus AI-powered natural language querying (QuickSight Q) on top of it.

## Data Sources

| Dataset | Rows | Description |
|---|---|---|
| `novatech_crm_deals.csv` | 499 | Closed sales opportunities (won/lost), 85 unique accounts |
| `novatech_marketing_campaigns.csv` | 2,240 | Marketing leads and campaign interactions |
| `novatech_support_tickets.csv` | 3,000 | Customer support tickets |
| `novatech_unified.csv` | 63,420 | Unified dataset joining all three sources on `account_id` |

All datasets are shared to **SPICE** for fast, in-memory querying.

## Data Preparation

Each source dataset was cleaned and enriched before joining:

- **Data type corrections** — fixed a date-format parsing bug (`yyyy-MM-dd HH:mm` vs. `yyyy-MM-dd`) that was silently nulling date fields; corrected several fields mis-typed as strings (e.g., `annual_income`, `household_size`).
- **Calculated fields** — added business-ready metrics instead of leaving raw math to the dashboard viewer:
  - `days_to_close` — deal creation to close date
  - `is_won` — binary flag from `deal_stage`
  - `campaign_roi` — `(revenue_attributed − campaign_spend) / campaign_spend`, with a zero-spend guard
  - `total_product_spend` — sum of product spend tiers
  - `resolution_time_hours` — ticket creation to resolution
  - `is_unresolved` — binary flag from a null resolution date

## Join Strategy

The unified dataset uses `novatech_crm_deals.csv` as the **anchor table**, joined via **left joins** on `account_id`:

1. **Join 1:** CRM (left) + Marketing (right)
2. **Join 2:** Join 1 result (left) + Support (right)

CRM stays the anchor throughout, so no deal records are dropped even where an account has no marketing or support activity. Because one CRM account can match multiple marketing leads and multiple support tickets, the join fans out row-wise (499 + 2,240 + 3,000 → 63,420 rows). Any aggregation on the unified dataset therefore uses **Count Distinct** on entity IDs (`lead_id`, `ticket_id`, `opportunity_id`) or level-aware calculations with `distinctCountOver` / `sumOver` and `PRE_AGG` partitioning, rather than plain `SUM`/`COUNT`, to avoid inflated totals.

## Dashboard

Three interactive sheets, each with filter controls and cross-sheet navigation actions:

- **Marketing Funnel** — leads by channel, funnel stage progression, campaign performance (spend vs. revenue, ROI)
- **Sales Pipeline** — win rate, deal outcomes, revenue by segment/product, win rate by region/rep/product, days-to-close trend
- **Customer Health** — ticket volume, resolution time by priority, sentiment distribution, account risk (ticket volume vs. deal value)

## AI-Powered Querying (QuickSight Q)

A custom **Topic** was configured over the unified dataset with business-friendly synonyms and calculated fields (e.g., Conversion Rate, Win Rate) to improve natural language query accuracy.

Structured exploration compared Q's answers against dashboard visuals to validate accuracy and document limitations — see `NovaTech Verification Log.pdf` and `NovaTech Q Exploration Log.pdf` for the full logs. Key finding: Q performs reliably on single-dataset lookups and simple aggregates, but **sums and totals on the unified dataset must be treated with suspicion** unless independently verified, since Q's on-the-fly aggregation does not consistently apply the same Count Distinct / PRE_AGG safeguards built into the dashboard's calculated fields.

## Repository Contents

| File | Description |
|---|---|
| `NovaTech_Executive_Report.pdf` | Executive summary of the project and key findings |
| `Dashboard executive summary text.pdf` | Written summary accompanying the dashboard |
| `NovaTech Revenue Intelligence Dashboard with annotations.pdf` | Full dashboard export with annotations explaining each visual |
| `NovaTech Verification Log.pdf` | Data verification log — Q answers checked against the data dictionary |
| `NovaTech Q Exploration Log.pdf` | Q exploration log — Q answers checked against dashboard visuals, including documented accuracy gaps |
| `Project_screenshots_Update.pdf` | Build evidence: SPICE imports, data type corrections, calculated fields, join configuration, dashboard sheets, and Topic setup |

## Tools

- Amazon QuickSight (SPICE, no-code joins, calculated fields, QuickSight Q / Topics, multi-sheet dashboards)
- Source data: CSV exports simulating NovaTech's CRM, marketing, and support systems

## Author

Raghad Almutairi
