# DuckDB Configuration for Product Management Plugin

## Overview

This plugin uses DuckDB for product metrics tracking, KPI analysis, and product analytics. DuckDB enables fast queries on feature usage data, user feedback, and roadmap metrics.

## Database Location

```
~/knowledge-work-data/product-management/product-management.duckdb
```

## Key Extensions

### VSS (Vector Similarity Search)
Find similar feature requests, user feedback, and product requirements.

```sql
INSTALL vss;
LOAD vss;

-- Store feature request embeddings
CREATE TABLE feature_requests (
    request_id INT,
    title VARCHAR,
    description TEXT,
    embedding FLOAT[384]
);

CREATE INDEX requests_idx ON feature_requests 
USING HNSW (embedding) WITH (metric = 'cosine');

-- Find similar requests
SELECT request_id, title,
       array_cosine_similarity(embedding, $1::FLOAT[384]) as similarity
FROM feature_requests
ORDER BY similarity DESC LIMIT 10;
```

### Full-Text Search
Search through user feedback, feature descriptions, and product documentation.

```sql
INSTALL fts;
LOAD fts;

PRAGMA create_fts_index('feature_requests', 'request_id', 'title', 'description');

SELECT * FROM (
    SELECT *, fts_main_feature_requests.match_bm25(request_id, 'mobile authentication') AS score
    FROM feature_requests
) WHERE score IS NOT NULL ORDER BY score DESC;
```

### Parquet for Analytics Data
Efficiently store and query product usage metrics and event data.

```sql
-- Load usage events
CREATE TABLE usage_events AS 
SELECT * FROM 'analytics/events_*.parquet';

-- Aggregate by feature
SELECT 
    feature_name,
    DATE_TRUNC('week', event_timestamp) as week,
    COUNT(DISTINCT user_id) as active_users,
    COUNT(*) as event_count
FROM usage_events
GROUP BY feature_name, week
ORDER BY week DESC;
```

## Common Product Management Queries

### Feature Adoption Metrics
```sql
-- Track feature adoption over time
WITH feature_users AS (
    SELECT 
        feature_id,
        DATE_TRUNC('week', first_used_at) as adoption_week,
        COUNT(DISTINCT user_id) as new_users
    FROM feature_usage
    GROUP BY feature_id, adoption_week
)
SELECT 
    feature_id,
    adoption_week,
    new_users,
    SUM(new_users) OVER (PARTITION BY feature_id ORDER BY adoption_week) as cumulative_users
FROM feature_users
ORDER BY feature_id, adoption_week;
```

### User Retention Analysis
```sql
-- Cohort retention analysis
SELECT 
    DATE_TRUNC('month', signup_date) as cohort_month,
    COUNT(DISTINCT user_id) as cohort_size,
    COUNT(DISTINCT CASE WHEN last_active_date >= signup_date + INTERVAL 30 DAY THEN user_id END) as retained_30d,
    COUNT(DISTINCT CASE WHEN last_active_date >= signup_date + INTERVAL 30 DAY THEN user_id END) * 100.0 / COUNT(DISTINCT user_id) as retention_rate
FROM users
GROUP BY cohort_month
ORDER BY cohort_month DESC;
```

### Roadmap Progress Tracking
```sql
-- Track feature delivery vs. plan
SELECT 
    sprint_name,
    COUNT(*) as total_features,
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) as completed,
    SUM(CASE WHEN status = 'completed' THEN story_points ELSE 0 END) as completed_points,
    SUM(story_points) as total_points,
    SUM(CASE WHEN status = 'completed' THEN story_points ELSE 0 END) * 100.0 / SUM(story_points) as completion_rate
FROM roadmap_items
GROUP BY sprint_name
ORDER BY sprint_name;
```

## Resources

- [DuckDB Documentation](https://duckdb.org/docs/)
- [Time Series Analysis](https://duckdb.org/docs/sql/functions/date)
- [Window Functions](https://duckdb.org/docs/sql/window_functions)
