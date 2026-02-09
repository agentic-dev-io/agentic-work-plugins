# Connectors

## Architektur: DuckDB-First

Dieses Plugin nutzt DuckDB als **primäre Datenschicht**. `~~category` Platzhalter
in Commands/Skills verweisen auf Daten die in DuckDB importiert wurden.

**Primary:** DuckDB + Extensions (siehe [DUCKDB.md](./DUCKDB.md))
**Import:** CLI + Skills (CSV/JSON/Parquet Export → DuckDB Import)

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool the user connects in that category. Plugins are **tool-agnostic** — they describe workflows in terms of categories rather than specific products. All data flows through DuckDB as the central analytical layer.

## Connectors for this plugin

| Kategorie | Placeholder | Import-Methode | Extensions |
|-----------|-------------|----------------|------------|
| Data warehouse | `~~data warehouse` | DuckDB Warehouse-Extensions (Snowflake, BigQuery, Databricks) | snowflake, bigquery, databricks |
| Parquet / CSV | `~~data source` | Lokale Dateien oder HTTPFS von S3/GCS/Azure | Parquet (built-in), httpfs |
| Notebook | `~~notebook` | Export als CSV/Parquet → DuckDB Import | json |
| Product analytics | `~~product analytics` | CSV/JSON Export aus Amplitude/Mixpanel → DuckDB Import | json, fts |
| Project tracker | `~~project tracker` | JSON/CSV Export aus Jira/Linear → DuckDB Import | json, fts |

### DuckDB als Primary Data Layer

DuckDB ersetzt direkte MCP-Verbindungen zu externen Systemen. Stattdessen:

1. **Warehouse-Extensions** (P0): Snowflake, BigQuery, Databricks — direkte SQL-Federation via DuckDB Extensions
2. **Datei-Import**: CSV/JSON/Parquet Exporte aus Tools → `CREATE TABLE ... AS SELECT * FROM 'file.parquet'`
3. **Cloud-Storage**: HTTPFS Extension für direkten Zugriff auf S3/GCS/Azure Parquet-Files
4. **Vektor-Suche**: VSS + FAISS Extensions für Embedding-basierte Analytics
5. **NLP in-DB**: Quackformers Extension für Embeddings und Sentiment direkt in DuckDB

Siehe [DUCKDB.md](./DUCKDB.md) für detaillierte Extension-Configs und SQL-Beispiele.
