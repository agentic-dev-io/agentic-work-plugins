# DuckDB Configuration for Customer Support Plugin

## Overview

This plugin uses DuckDB for support ticket analysis, customer interaction tracking, and knowledge base management with full-text and semantic search capabilities.

## Database Location

```
~/knowledge-work-data/customer-support/customer-support.duckdb
```

## Key Extensions

### VSS (Vector Similarity Search)
Find similar tickets, suggest solutions, and match customer issues to knowledge base articles.

```sql
INSTALL vss;
LOAD vss;

-- Store ticket embeddings
CREATE TABLE support_tickets (
    ticket_id INT,
    customer_id INT,
    subject VARCHAR,
    description TEXT,
    status VARCHAR,
    priority VARCHAR,
    embedding FLOAT[384]
);

CREATE INDEX tickets_idx ON support_tickets 
USING HNSW (embedding) WITH (metric = 'cosine');

-- Find similar tickets
SELECT ticket_id, subject, status,
       array_cosine_similarity(embedding, $query_embedding::FLOAT[384]) as similarity
FROM support_tickets
WHERE status = 'resolved'
ORDER BY similarity DESC LIMIT 5;
```

### Full-Text Search
Search through tickets, knowledge base, and customer interactions.

```sql
INSTALL fts;
LOAD fts;

-- Index knowledge base articles
PRAGMA create_fts_index('kb_articles', 'article_id', 'title', 'content');

-- Search for solutions
SELECT article_id, title, score
FROM (
    SELECT *, fts_main_kb_articles.match_bm25(article_id, 'password reset email') AS score
    FROM kb_articles
) WHERE score IS NOT NULL ORDER BY score DESC;
```

## Common Support Queries

### Ticket Volume and Response Time Analysis
```sql
-- Daily ticket metrics
SELECT 
    DATE_TRUNC('day', created_at) as date,
    COUNT(*) as tickets_created,
    AVG(EXTRACT(EPOCH FROM (first_response_at - created_at)) / 3600) as avg_response_hours,
    AVG(EXTRACT(EPOCH FROM (resolved_at - created_at)) / 3600) as avg_resolution_hours,
    SUM(CASE WHEN priority = 'high' THEN 1 ELSE 0 END) as high_priority
FROM support_tickets
WHERE created_at >= CURRENT_DATE - INTERVAL 30 DAY
GROUP BY date
ORDER BY date DESC;
```

### Customer Support History
```sql
-- Customer interaction timeline
SELECT 
    customer_id,
    COUNT(*) as total_tickets,
    SUM(CASE WHEN status = 'resolved' THEN 1 ELSE 0 END) as resolved_tickets,
    AVG(satisfaction_score) as avg_satisfaction,
    MAX(created_at) as last_ticket_date,
    STRING_AGG(DISTINCT category, ', ') as common_issues
FROM support_tickets
GROUP BY customer_id
HAVING total_tickets >= 3
ORDER BY total_tickets DESC;
```

### Escalation Pattern Analysis
```sql
-- Track escalations
SELECT 
    category,
    COUNT(*) as total_tickets,
    SUM(CASE WHEN escalated = TRUE THEN 1 ELSE 0 END) as escalated,
    SUM(CASE WHEN escalated = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(*) as escalation_rate,
    AVG(CASE WHEN escalated = TRUE THEN resolution_hours END) as avg_escalated_resolution
FROM support_tickets
GROUP BY category
ORDER BY escalation_rate DESC;
```

## Resources

- [DuckDB Documentation](https://duckdb.org/docs/)
- [Full-Text Search](https://duckdb.org/docs/extensions/full_text_search)
- [VSS Extension](https://duckdb.org/docs/extensions/vss)
