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
| Local database | `~~database` | DuckDB (primary) | Parquet, json, vss, fts, httpfs |
| Data warehouse | `~~data warehouse` | DuckDB Warehouse-Extensions (Snowflake, BigQuery, Databricks) | snowflake, bigquery, databricks |
| ERP / Accounting | `~~erp` | ERP-Export (CSV/JSON/Parquet) → DuckDB Import | json, Parquet |
| Analytics / BI | `~~analytics` | Dashboard-Export → DuckDB Import | Parquet, json |

### DuckDB als Primary Data Layer

DuckDB ersetzt direkte MCP-Verbindungen zu externen Systemen. Stattdessen:

1. **Warehouse-Extensions** (P0): Snowflake, BigQuery, Databricks — direkte SQL-Federation via DuckDB Extensions
2. **ERP-Daten**: CSV/JSON/Parquet Export aus NetSuite, SAP, QuickBooks, Xero → DuckDB Import
3. **Cloud-Storage**: HTTPFS Extension für direkten Zugriff auf S3/GCS/Azure Parquet-Files
4. **Finanz-Analyse**: ANOFOX Statistics + Forecast Extensions für Variance Analysis und Projections
5. **Volltextsuche**: FTS Extension für Suche in Beschreibungen und Notizen
6. **Semantic Search**: VSS Extension für ähnliche Transaktionen und Dokumente

Siehe [DUCKDB.md](./DUCKDB.md) für detaillierte Extension-Configs und SQL-Beispiele.
