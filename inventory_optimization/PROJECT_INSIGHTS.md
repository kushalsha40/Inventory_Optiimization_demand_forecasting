# Inventory Project Insights

## Purpose

This file stores the key findings from `notebooks/eda.ipynb` in business language so they can be reused for modeling, documentation, and dashboard design.

## Project Goal

Build an inventory optimization workflow that can answer:

- which SKUs need replenishment first
- which SKUs carry stockout risk or overstock risk
- how much safety stock is needed by SKU
- how promotions, seasonality, competition, and supplier lead time affect inventory decisions

## Dataset Snapshot

### Raw files

- `data/raw/sales_fact.csv`
- `data/raw/inventory_snapshot.csv`
- `data/raw/products_master.csv`
- `data/raw/suppliers_master.csv`

### Summary from EDA

- Sales data covers `2024-01-01` to `2024-12-31`
- `sales_fact.csv` contains `14,640` daily SKU records
- The project contains `40` SKUs and `10` suppliers
- Product master and inventory snapshot align fully with sales SKUs
- No missing category values were found after joining sales with products
- Product mix spans `6` top-level categories

## Insights From `eda.ipynb`

### 1. Demand history is complete and stable enough for forecasting

**Observation:**  
The dataset contains one full year of daily sales for `40` SKUs, with average demand of `29.53` units and median demand of `28` units per SKU-day.

**Why this is important:**  
This gives enough history to build daily or weekly forecasts and to study patterns like seasonality, promotion effects, and supplier-driven inventory risk.

### 2. Electronics Accessories is the highest-demand category

**Observation:**  
Total category demand is led by `Electronics Accessories` with `134,038` units, followed by `Home & Kitchen` with `89,931` and `Fitness & Wellness` with `66,021`.

**Why this is important:**  
These categories should receive tighter service-level targets, more monitoring, and stronger replenishment rules because stockouts here will have the biggest operational impact.

### 3. Demand is concentrated in a small group of top SKUs

**Observation:**  
The top-selling SKUs are `SKU0034`, `SKU0018`, `SKU0013`, `SKU0038`, and `SKU0005`, all with more than `12,700` annual units sold.

**Why this is important:**  
These products are strong candidates for `A` items in ABC analysis, which means they should be replenished more carefully and tracked more frequently.

### 4. Promotions have a strong positive impact on demand

**Observation:**  
Average demand is `38.28` units when `promo_flag = 1` versus `26.60` when `promo_flag = 0`. The promotion t-test produced a near-zero p-value, which supports a statistically significant difference.

**Why this is important:**  
Promotion periods should not use normal reorder rules. Forecasting and replenishment logic must include promotion effects to avoid preventable stockouts.

### 5. Discounts are positively related to demand

**Observation:**  
The correlation between `discount_pct` and `units_sold` is `0.63`, which is a strong positive relationship in this dataset.

**Why this is important:**  
Discounting appears to be a meaningful demand driver. Price-led demand uplifts should be included as a feature in demand forecasting and scenario planning.

### 6. Festival season produces clearly higher demand

**Observation:**  
Average demand rises from `28.34` units in non-festival periods to `33.05` in festival periods. The festival t-test also returned a near-zero p-value.

**Why this is important:**  
Festival periods need higher reorder points and possibly higher safety stock because normal-period assumptions will likely understate true demand.

### 7. Seasonality matters, with festive months standing out most

**Observation:**  
Average demand by season is highest in `festive` at `33.05`, while `summer`, `monsoon`, and `winter` all stay near `28.3` to `28.4`. ANOVA also shows a statistically significant seasonal difference.

**Why this is important:**  
Seasonality should be modeled explicitly instead of using one flat annual demand average across the entire year.

### 8. Competitor stockouts create extra demand for our products

**Observation:**  
Average demand increases from `29.16` to `32.37` units when `competitor_stockout_flag = 1`, which is roughly an `11.01%` uplift.

**Why this is important:**  
When competitors run out of stock, demand shifts to us. This can be turned into an external demand signal for forecast adjustment and risk monitoring.

### 9. Campaign intensity is a stronger signal than traffic or ratings

**Observation:**  
`campaign_intensity` has correlation `0.509` with demand, while `traffic_index` is `0.25`, `rating_score` is `0.052`, and `review_volume` is `0.031`.

**Why this is important:**  
Marketing effort appears to influence demand more directly than ratings or review count in this data. It should be prioritized as a forecasting feature.

### 10. Day-of-week and selling price have weaker direct relationships

**Observation:**  
`day_of_week` has low correlation with demand at `0.056`, and `selling_price` has a mild negative correlation at `-0.138`.

**Why this is important:**  
These variables may still matter in combination with other factors, but they are weaker standalone predictors than promotions, discounts, campaigns, and festival periods.

### 11. Most SKUs show moderate demand variability

**Observation:**  
Using coefficient of variation, `30` SKUs fall in `Y (Moderate)` and `10` fall in `X (stable)`. No SKU currently falls into a highly volatile `Z` class using the notebook’s thresholds.

**Why this is important:**  
The inventory problem is more about moderate, manageable variability than extreme unpredictability. This supports using ABC-XYZ segmentation for differentiated policies.

### 12. Supplier lead times vary enough to matter for replenishment

**Observation:**  
Supplier average lead time ranges from `3` to `13` days, with average lead time of `6.8` days. Lead time variability averages `3.2`.

**Why this is important:**  
Lead time uncertainty should be built into safety stock calculations, especially for SKUs tied to slower or more variable suppliers.

### 13. Existing stock policies already differ materially across SKUs

**Observation:**  
Current stock averages `611.38`, safety stock averages `228.60`, reorder point averages `474.90`, and lead time averages `7.43` days.

**Why this is important:**  
The business already uses inventory rules, so the project should compare current policy versus optimized policy instead of starting from zero.

## Notebook Cleanup Notes

### What is already good

- The notebook covers demand trend, category analysis, SKU performance, promotions, seasonality, competitor effects, volatility, and hypothesis testing
- The analysis direction is strong for an inventory optimization project

### What should be improved next

- Replace hard-coded absolute file paths with project-relative paths
- Add markdown section headers before each analysis block
- Remove duplicate exploratory cells and empty cells
- Convert repeated logic into reusable functions later in `src/`
- Keep comments aligned with computed results because some current comments contradict the numbers

## Recommended Business Questions

- Which SKUs should become `A-X`, `A-Y`, and `B-Y` priority items?
- Which SKUs need higher safety stock before festive months?
- Which supplier-linked SKUs face the highest stockout exposure?
- How much does demand increase under promotion or competitor stockout conditions?
- Which categories deserve the highest service-level targets?

## Insight Log Template

### Insight Title

**Observation:**  
Write the pattern found in the data.

**Evidence:**  
Mention the metric, table, or chart.

**Business impact:**  
Explain how it affects stock planning, forecasting, service level, or working capital.

**Action for project:**  
Explain what this means for feature engineering, forecasting, classification, or inventory logic.

## Build Priority After EDA

### Phase 1: Data foundation

- build loaders for all raw files
- validate schema, joins, and date parsing
- create one merged SKU-date modeling table

### Phase 2: Segmentation

- implement ABC classification using annual demand value
- implement XYZ classification using demand variability
- combine them into an inventory priority matrix

### Phase 3: Inventory logic

- build baseline demand forecast
- calculate safety stock using service level and uncertainty
- calculate reorder point using demand during lead time
- flag stockout and overstock risk

### Phase 4: Dashboard

- show category and SKU demand summary
- show top replenishment-priority SKUs
- compare current inventory policy against recommended policy
