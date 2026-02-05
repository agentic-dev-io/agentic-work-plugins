# DuckDB Configuration for Enterprise Search Plugin

## Overview

This plugin uses DuckDB with full-text search and vector similarity capabilities to enable semantic search across enterprise content, documents, and communications.

## Database Location

```
~/knowledge-work-data/enterprise-search/enterprise-search.duckdb
```

## Key Extensions

### VSS (Vector Similarity Search)
Semantic search across all enterprise content.

```sql
INSTALL vss;
LOAD vss;

CREATE TABLE documents (
    doc_id INT,
    title VARCHAR,
    content TEXT,
    source VARCHAR,
    author VARCHAR,
    created_at TIMESTAMP,
    embedding FLOAT[768]
);

CREATE INDEX docs_idx ON documents 
USING HNSW (embedding) WITH (metric = 'cosine');

-- Semantic search
SELECT doc_id, title, source,
       array_cosine_similarity(embedding, $query::FLOAT[768]) as relevance
FROM documents
ORDER BY relevance DESC LIMIT 20;
```

### Full-Text Search
Keyword-based search with ranking.

```sql
INSTALL fts;
LOAD fts;

PRAGMA create_fts_index('documents', 'doc_id', 'title', 'content');

-- Combined keyword + semantic search
WITH keyword_results AS (
    SELECT doc_id, fts_main_documents.match_bm25(doc_id, 'quarterly business review') AS bm25_score
    FROM documents
),
semantic_results AS (
    SELECT doc_id, array_cosine_similarity(embedding, $query_embedding::FLOAT[768]) as semantic_score
    FROM documents
)
SELECT 
    d.*,
    COALESCE(k.bm25_score, 0) * 0.4 + COALESCE(s.semantic_score, 0) * 0.6 as final_score
FROM documents d
LEFT JOIN keyword_results k ON d.doc_id = k.doc_id
LEFT JOIN semantic_results s ON d.doc_id = s.doc_id
WHERE final_score > 0.3
ORDER BY final_score DESC;
```

## Common Search Queries

### Cross-Source Search
```sql
-- Search across multiple sources
SELECT 
    source,
    COUNT(*) as result_count,
    AVG(relevance_score) as avg_relevance
FROM search_results
WHERE query = 'product roadmap Q4'
GROUP BY source
ORDER BY avg_relevance DESC;
```

### Usage Analytics
```sql
-- Track search patterns
SELECT 
    DATE_TRUNC('day', search_timestamp) as date,
    COUNT(DISTINCT user_id) as unique_searchers,
    COUNT(*) as total_searches,
    AVG(result_count) as avg_results,
    SUM(CASE WHEN clicked_result THEN 1 ELSE 0 END) * 100.0 / COUNT(*) as click_through_rate
FROM search_logs
GROUP BY date
ORDER BY date DESC;
```

## Resources

- [DuckDB Full-Text Search](https://duckdb.org/docs/extensions/full_text_search)
- [VSS Extension](https://duckdb.org/docs/extensions/vss)
