# DuckDB Configuration for Sales Plugin

## Overview

This plugin uses DuckDB for sales pipeline analytics, deal tracking, customer relationship analysis, and sales forecasting with graph-like relationship queries.

## Database Location

```
~/knowledge-work-data/sales/sales.duckdb
```

## Key Extensions

### VSS (Vector Similarity Search)
Find similar deals, prospects, and sales opportunities.

```sql
INSTALL vss;
LOAD vss;

CREATE TABLE opportunities (
    opp_id INT,
    account_name VARCHAR,
    deal_size DECIMAL,
    stage VARCHAR,
    close_date DATE,
    notes TEXT,
    embedding FLOAT[384]
);

CREATE INDEX opps_idx ON opportunities 
USING HNSW (embedding) WITH (metric = 'cosine');

-- Find similar deals for insights
SELECT opp_id, account_name, deal_size, stage,
       array_cosine_similarity(embedding, $current_deal::FLOAT[384]) as similarity
FROM opportunities
WHERE stage = 'closed_won'
ORDER BY similarity DESC LIMIT 10;
```

### JSON for Hierarchical Data
Store account hierarchies and relationships.

```sql
INSTALL json;
LOAD json;

CREATE TABLE accounts (
    account_id INT,
    account_name VARCHAR,
    hierarchy JSON  -- Store parent-child relationships
);

-- Query account hierarchies
SELECT 
    account_id,
    account_name,
    json_extract(hierarchy, '$.parent_id') as parent_account,
    json_extract(hierarchy, '$.level') as org_level
FROM accounts;
```

## Common Sales Queries

### Pipeline Analysis
```sql
-- Pipeline by stage and rep
SELECT 
    sales_rep,
    stage,
    COUNT(*) as deal_count,
    SUM(deal_value) as total_value,
    AVG(deal_value) as avg_deal_size,
    AVG(EXTRACT(DAY FROM (CURRENT_DATE - created_date))) as avg_age_days
FROM opportunities
WHERE stage != 'closed_lost' AND stage != 'closed_won'
GROUP BY sales_rep, stage
ORDER BY sales_rep, total_value DESC;
```

### Win Rate Analysis
```sql
-- Win rates by segment and size
SELECT 
    customer_segment,
    CASE 
        WHEN deal_value < 10000 THEN 'Small'
        WHEN deal_value < 50000 THEN 'Medium'
        ELSE 'Large'
    END as deal_size_category,
    COUNT(*) as total_opportunities,
    SUM(CASE WHEN stage = 'closed_won' THEN 1 ELSE 0 END) as wins,
    SUM(CASE WHEN stage = 'closed_won' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) as win_rate,
    AVG(CASE WHEN stage = 'closed_won' THEN deal_value END) as avg_win_value
FROM opportunities
WHERE stage IN ('closed_won', 'closed_lost')
GROUP BY customer_segment, deal_size_category;
```

### Sales Velocity
```sql
-- Average time in each stage
SELECT 
    stage,
    COUNT(*) as deals_in_stage,
    AVG(EXTRACT(DAY FROM (stage_exit_date - stage_entry_date))) as avg_days_in_stage,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY EXTRACT(DAY FROM (stage_exit_date - stage_entry_date))) as median_days
FROM stage_history
WHERE stage_exit_date IS NOT NULL
GROUP BY stage
ORDER BY 
    CASE stage
        WHEN 'prospecting' THEN 1
        WHEN 'qualification' THEN 2
        WHEN 'proposal' THEN 3
        WHEN 'negotiation' THEN 4
        WHEN 'closed_won' THEN 5
    END;
```

### Forecast Accuracy
```sql
-- Compare forecasts to actuals
WITH forecast_vs_actual AS (
    SELECT 
        forecast_month,
        SUM(forecasted_value) as forecasted,
        SUM(CASE WHEN closed_date IS NOT NULL THEN actual_value ELSE 0 END) as actual
    FROM sales_forecast
    GROUP BY forecast_month
)
SELECT 
    forecast_month,
    forecasted,
    actual,
    actual - forecasted as variance,
    (actual - forecasted) * 100.0 / forecasted as variance_pct
FROM forecast_vs_actual
ORDER BY forecast_month DESC;
```

## Resources

- [DuckDB Window Functions](https://duckdb.org/docs/sql/window_functions)
- [Date/Time Functions](https://duckdb.org/docs/sql/functions/date)
- [VSS Extension](https://duckdb.org/docs/extensions/vss)
