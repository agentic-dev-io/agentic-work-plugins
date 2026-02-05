# DuckDB Configuration for Bio-Research Plugin

## Overview

This plugin uses DuckDB as its primary analytical database for biomedical research data processing, genomics analysis, and scientific computing. DuckDB's support for large datasets and Parquet files makes it ideal for handling biological data.

## Database Location

The DuckDB database file is stored at:
```
~/knowledge-work-data/bio-research/bio-research.duckdb
```

## Key Extensions for Bio-Research

### 1. Parquet Extension (Built-in)
Essential for handling large genomics datasets, experimental results, and clinical trial data.

**Usage:**
```sql
-- Load genomics data from Parquet
CREATE TABLE gene_expression AS 
SELECT * FROM 'expression_data.parquet';

-- Load clinical trial results
CREATE TABLE trial_results AS
SELECT * FROM 's3://research-data/clinical-trials/*.parquet';

-- Export analysis results
COPY (
    SELECT gene_id, sample_id, expression_level, p_value
    FROM differential_expression
    WHERE p_value < 0.05
) TO 'significant_genes.parquet' (FORMAT PARQUET);
```

### 2. VSS (Vector Similarity Search)
Perform similarity searches on protein sequences, chemical structures, and research papers.

**Installation:**
```sql
INSTALL vss;
LOAD vss;
```

**Usage:**
```sql
-- Store research paper embeddings
CREATE TABLE research_papers (
    pmid INT,
    title VARCHAR,
    abstract TEXT,
    embedding FLOAT[768]  -- BioBERT or SciBERT embeddings
);

-- Create HNSW index for fast similarity search
CREATE INDEX papers_embedding_idx ON research_papers 
USING HNSW (embedding) WITH (metric = 'cosine');

-- Find similar papers
SELECT pmid, title, 
       array_cosine_similarity(embedding, $query_embedding::FLOAT[768]) as similarity
FROM research_papers
ORDER BY similarity DESC
LIMIT 20;

-- Chemical compound similarity
CREATE TABLE compounds (
    compound_id VARCHAR,
    smiles VARCHAR,
    molecular_fingerprint FLOAT[2048]  -- Morgan fingerprint
);

CREATE INDEX compounds_fp_idx ON compounds
USING HNSW (molecular_fingerprint) WITH (metric = 'cosine');
```

### 3. JSON Extension
Handle complex biological data structures from APIs and experimental metadata.

**Installation:**
```sql
INSTALL json;
LOAD json;
```

**Usage:**
```sql
-- Parse metadata from sequencing runs
SELECT 
    sample_id,
    json_extract(metadata, '$.sequencing_platform') as platform,
    json_extract(metadata, '$.read_length') as read_length,
    json_extract(metadata, '$.coverage') as coverage
FROM sequencing_runs;

-- Store nested experimental conditions
CREATE TABLE experiments (
    experiment_id INT,
    conditions JSON
);

INSERT INTO experiments VALUES 
(1, '{"temperature": 37, "pH": 7.4, "treatment": {"drug": "compound_x", "concentration": "10uM"}}');
```

### 4. FTS (Full-Text Search)
Search through research literature, experimental notes, and protocols.

**Installation:**
```sql
INSTALL fts;
LOAD fts;
```

**Usage:**
```sql
-- Index research abstracts
PRAGMA create_fts_index('research_papers', 'pmid', 'title', 'abstract');

-- Search for relevant papers
SELECT pmid, title, score
FROM (
    SELECT *, fts_main_research_papers.match_bm25(pmid, 'CRISPR gene editing') AS score
    FROM research_papers
) WHERE score IS NOT NULL
ORDER BY score DESC;
```

### 5. HTTPFS Extension
Access genomics data from cloud storage and public repositories.

**Installation:**
```sql
INSTALL httpfs;
LOAD httpfs;

-- Configure for AWS S3 (for accessing public genomics data)
SET s3_region='us-east-1';
```

**Usage:**
```sql
-- Query public genomics data
SELECT * FROM 's3://1000genomes/phase3/data/*.parquet' LIMIT 100;

-- Load ChEMBL data
CREATE TABLE chembl_compounds AS
SELECT * FROM 'https://data.chembl.org/exports/compounds.parquet';
```

## Common Bio-Research Use Cases

### 1. Differential Gene Expression Analysis
```sql
-- Calculate fold changes and p-values
WITH expression_stats AS (
    SELECT 
        gene_id,
        AVG(CASE WHEN condition = 'treatment' THEN expression END) as treatment_mean,
        AVG(CASE WHEN condition = 'control' THEN expression END) as control_mean,
        STDDEV(CASE WHEN condition = 'treatment' THEN expression END) as treatment_sd,
        STDDEV(CASE WHEN condition = 'control' THEN expression END) as control_sd,
        COUNT(CASE WHEN condition = 'treatment' THEN 1 END) as treatment_n,
        COUNT(CASE WHEN condition = 'control' THEN 1 END) as control_n
    FROM gene_expression
    GROUP BY gene_id
)
SELECT 
    gene_id,
    treatment_mean,
    control_mean,
    LOG2(treatment_mean / control_mean) as log2_fold_change,
    -- T-test calculation
    (treatment_mean - control_mean) / 
        SQRT((treatment_sd^2 / treatment_n) + (control_sd^2 / control_n)) as t_statistic
FROM expression_stats
WHERE control_mean > 0 AND treatment_mean > 0;
```

### 2. Clinical Trial Patient Cohort Analysis
```sql
-- Analyze patient demographics and outcomes
SELECT 
    treatment_arm,
    COUNT(*) as patient_count,
    AVG(age) as avg_age,
    SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) as pct_female,
    AVG(primary_endpoint_score) as avg_outcome,
    SUM(CASE WHEN adverse_event = TRUE THEN 1 ELSE 0 END) as adverse_events
FROM clinical_trial_patients
GROUP BY treatment_arm;
```

### 3. Protein-Protein Interaction Network
```sql
-- Build interaction network from experimental data
CREATE TABLE protein_interactions AS
SELECT 
    p1.protein_id as protein_a,
    p2.protein_id as protein_b,
    i.interaction_type,
    i.confidence_score
FROM proteins p1
JOIN interactions i ON p1.protein_id = i.protein_a_id
JOIN proteins p2 ON p2.protein_id = i.protein_b_id
WHERE i.confidence_score > 0.7;

-- Find highly connected proteins (hubs)
SELECT 
    protein_id,
    COUNT(*) as interaction_count,
    AVG(confidence_score) as avg_confidence
FROM (
    SELECT protein_a as protein_id, confidence_score FROM protein_interactions
    UNION ALL
    SELECT protein_b as protein_id, confidence_score FROM protein_interactions
)
GROUP BY protein_id
ORDER BY interaction_count DESC
LIMIT 50;
```

### 4. Sequence Alignment Results Analysis
```sql
-- Analyze alignment quality metrics
SELECT 
    query_sequence_id,
    COUNT(*) as num_alignments,
    MAX(alignment_score) as best_score,
    AVG(percent_identity) as avg_identity,
    AVG(alignment_length) as avg_length,
    MIN(e_value) as best_e_value
FROM blast_results
WHERE e_value < 1e-5
GROUP BY query_sequence_id;
```

### 5. Drug Screening Results
```sql
-- Identify hit compounds from screening
WITH screening_stats AS (
    SELECT 
        compound_id,
        AVG(activity) as mean_activity,
        STDDEV(activity) as sd_activity,
        COUNT(*) as replicate_count
    FROM screening_results
    GROUP BY compound_id
)
SELECT 
    s.*,
    c.compound_name,
    c.molecular_weight,
    c.log_p,
    -- Z-score calculation
    (s.mean_activity - (SELECT AVG(mean_activity) FROM screening_stats)) / 
        (SELECT STDDEV(mean_activity) FROM screening_stats) as z_score
FROM screening_stats s
JOIN compounds c ON s.compound_id = c.compound_id
WHERE s.mean_activity > (SELECT AVG(mean_activity) + 3*STDDEV(mean_activity) FROM screening_stats)
ORDER BY z_score DESC;
```

## Performance Tips for Large Biological Datasets

1. **Use Parquet for genomics data**: 10-100x compression vs. CSV
2. **Partition by chromosome or experiment**: Faster queries on specific regions
3. **Index on gene IDs and positions**: Essential for range queries
4. **Use columnar projections**: Only read columns you need
5. **Batch operations**: Process samples in groups for better cache usage

## Integration with Bioinformatics Tools

DuckDB can complement existing tools:
- **Query results from BLAST, HMMER**: Load TSV/CSV outputs
- **Analyze VCF files**: Variant call analysis
- **Process GFF/GTF annotations**: Gene feature queries
- **Aggregate multi-omics data**: Join RNA-seq, proteomics, metabolomics
- **Clinical data integration**: Combine with patient records

## Data Formats Supported

- **Parquet**: Recommended for large datasets
- **CSV/TSV**: Common in bioinformatics pipelines
- **JSON/JSONL**: API responses, metadata
- **Arrow**: Zero-copy data exchange with Python/R
- **Direct cloud access**: S3, GCS for public datasets

## Example: Complete Analysis Pipeline

```sql
-- 1. Load gene expression data
CREATE TABLE expression AS 
SELECT * FROM 'experiment_001/counts.parquet';

-- 2. Load sample metadata
CREATE TABLE samples AS
SELECT * FROM 'experiment_001/metadata.json';

-- 3. Calculate differential expression
CREATE TABLE diff_exp AS
SELECT 
    e.gene_id,
    AVG(CASE WHEN s.condition = 'treated' THEN e.count END) as treated,
    AVG(CASE WHEN s.condition = 'control' THEN e.count END) as control,
    LOG2(AVG(CASE WHEN s.condition = 'treated' THEN e.count END) / 
         AVG(CASE WHEN s.condition = 'control' THEN e.count END)) as log2fc
FROM expression e
JOIN samples s ON e.sample_id = s.sample_id
GROUP BY e.gene_id
HAVING control > 0 AND treated > 0;

-- 4. Find significant genes
CREATE TABLE significant_genes AS
SELECT * FROM diff_exp 
WHERE ABS(log2fc) > 1.5;

-- 5. Export results
COPY significant_genes TO 'results/significant_genes.parquet' (FORMAT PARQUET);
```

## Resources

- [DuckDB Documentation](https://duckdb.org/docs/)
- [VSS Extension](https://duckdb.org/docs/extensions/vss)
- [Parquet for Genomics](https://duckdb.org/docs/data/parquet/overview)
- [JSON Handling](https://duckdb.org/docs/extensions/json)
