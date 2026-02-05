# DuckDB Configuration for Marketing Plugin

## Overview

This plugin uses DuckDB for campaign analytics, customer journey tracking, content performance analysis, and marketing attribution modeling.

## Database Location

```
~/knowledge-work-data/marketing/marketing.duckdb
```

## Key Extensions

### VSS (Vector Similarity Search)
Find similar campaigns, content themes, and audience segments.

```sql
INSTALL vss;
LOAD vss;

CREATE TABLE campaigns (
    campaign_id INT,
    campaign_name VARCHAR,
    channel VARCHAR,
    content TEXT,
    embedding FLOAT[384]
);

CREATE INDEX campaigns_idx ON campaigns 
USING HNSW (embedding) WITH (metric = 'cosine');

-- Find similar successful campaigns
SELECT c.campaign_id, c.campaign_name, c.channel,
       array_cosine_similarity(c.embedding, $new_campaign::FLOAT[384]) as similarity,
       p.conversion_rate
FROM campaigns c
JOIN campaign_performance p ON c.campaign_id = p.campaign_id
WHERE p.conversion_rate > 0.05
ORDER BY similarity DESC LIMIT 10;
```

### Full-Text Search
Search content library and campaign materials.

```sql
INSTALL fts;
LOAD fts;

PRAGMA create_fts_index('content_library', 'content_id', 'title', 'body', 'tags');

SELECT * FROM (
    SELECT *, fts_main_content_library.match_bm25(content_id, 'product launch announcement') AS score
    FROM content_library
) WHERE score IS NOT NULL ORDER BY score DESC;
```

## Common Marketing Queries

### Campaign Performance
```sql
-- Multi-channel campaign ROI
SELECT 
    campaign_name,
    channel,
    SUM(spend) as total_spend,
    SUM(impressions) as total_impressions,
    SUM(clicks) as total_clicks,
    SUM(conversions) as total_conversions,
    SUM(revenue) as total_revenue,
    (SUM(revenue) - SUM(spend)) / SUM(spend) * 100 as roi_pct,
    SUM(revenue) / SUM(spend) as roas,
    SUM(spend) / SUM(conversions) as cost_per_conversion
FROM campaign_metrics
GROUP BY campaign_name, channel
ORDER BY roi_pct DESC;
```

### Attribution Modeling
```sql
-- Multi-touch attribution
WITH touch_sequences AS (
    SELECT 
        customer_id,
        conversion_id,
        touchpoint_channel,
        touchpoint_timestamp,
        ROW_NUMBER() OVER (PARTITION BY conversion_id ORDER BY touchpoint_timestamp) as touch_order,
        COUNT(*) OVER (PARTITION BY conversion_id) as total_touches
    FROM customer_touchpoints
    WHERE conversion_id IS NOT NULL
)
SELECT 
    touchpoint_channel,
    COUNT(DISTINCT conversion_id) as attributed_conversions,
    -- First touch attribution
    SUM(CASE WHEN touch_order = 1 THEN 1.0 ELSE 0 END) as first_touch_credit,
    -- Last touch attribution
    SUM(CASE WHEN touch_order = total_touches THEN 1.0 ELSE 0 END) as last_touch_credit,
    -- Linear attribution
    SUM(1.0 / total_touches) as linear_credit,
    -- U-shaped (40-20-40)
    SUM(CASE 
        WHEN touch_order = 1 THEN 0.4
        WHEN touch_order = total_touches THEN 0.4
        ELSE 0.2 / (total_touches - 2)
    END) as u_shaped_credit
FROM touch_sequences
GROUP BY touchpoint_channel
ORDER BY attributed_conversions DESC;
```

### Audience Segmentation
```sql
-- RFM segmentation
WITH customer_metrics AS (
    SELECT 
        customer_id,
        MAX(purchase_date) as last_purchase,
        COUNT(*) as frequency,
        SUM(purchase_amount) as monetary
    FROM purchases
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT 
        customer_id,
        NTILE(5) OVER (ORDER BY EXTRACT(DAY FROM (CURRENT_DATE - last_purchase))) as recency_score,
        NTILE(5) OVER (ORDER BY frequency) as frequency_score,
        NTILE(5) OVER (ORDER BY monetary) as monetary_score
    FROM customer_metrics
)
SELECT 
    CASE 
        WHEN recency_score >= 4 AND frequency_score >= 4 THEN 'Champions'
        WHEN recency_score >= 3 AND frequency_score >= 3 THEN 'Loyal'
        WHEN recency_score >= 4 AND frequency_score <= 2 THEN 'Promising'
        WHEN recency_score <= 2 AND frequency_score >= 3 THEN 'At Risk'
        WHEN recency_score <= 2 AND frequency_score <= 2 THEN 'Lost'
        ELSE 'Other'
    END as segment,
    COUNT(*) as customer_count,
    AVG(monetary_score) as avg_value_score
FROM rfm_scores
GROUP BY segment;
```

### Content Performance
```sql
-- Content engagement metrics
SELECT 
    content_type,
    content_title,
    SUM(views) as total_views,
    SUM(shares) as total_shares,
    SUM(conversions) as total_conversions,
    AVG(time_on_page) as avg_engagement_seconds,
    SUM(conversions) * 100.0 / SUM(views) as conversion_rate,
    SUM(shares) * 100.0 / SUM(views) as share_rate
FROM content_analytics
WHERE publish_date >= CURRENT_DATE - INTERVAL 90 DAY
GROUP BY content_type, content_title
ORDER BY total_conversions DESC;
```

## Resources

- [DuckDB Analytics](https://duckdb.org/docs/)
- [Window Functions for Cohorts](https://duckdb.org/docs/sql/window_functions)
- [VSS Extension](https://duckdb.org/docs/extensions/vss)
