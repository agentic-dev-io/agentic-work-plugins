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
| CRM | `~~CRM` | CRM-Export (CSV/JSON/Parquet) → DuckDB Import | json, Parquet, vss |
| Data enrichment | `~~data enrichment` | Enrichment-Export (CSV/JSON) → DuckDB Import | json, fts |
| Email | `~~email` | — (Output: Copy to email client) | — |
| Calendar | `~~calendar` | ICS/JSON Export → DuckDB Import | json |
| Knowledge base | `~~knowledge base` | Content-Export → DuckDB Import mit FTS | fts, vss |
| Meeting transcription | `~~conversation intelligence` | Transkript-Export → DuckDB Import mit FTS + VSS | fts, vss |
| Project tracker | `~~project tracker` | JSON/CSV Export → DuckDB Import | json, fts |

### DuckDB als Primary Data Layer

DuckDB ersetzt direkte MCP-Verbindungen zu externen Systemen. Stattdessen:

1. **CRM-Daten**: CSV/JSON/Parquet Export aus HubSpot, Salesforce → DuckDB Import
2. **Pipeline-Analytics**: VSS Extension für ähnliche Deals, Parquet für historische Analyse
3. **Transkripte**: Call-Recordings/Transkripte → DuckDB mit FTS + VSS für Insights
4. **Lead-Enrichment**: CSV/JSON Export aus Clay, ZoomInfo → DuckDB Import
5. **Fuzzy Matching**: Fuzzycomplete Extension für Lead-Deduplikation
6. **Prospecting**: Webbed + Netquack Extensions für Company Research

Siehe [DUCKDB.md](./DUCKDB.md) für detaillierte Extension-Configs und SQL-Beispiele.
