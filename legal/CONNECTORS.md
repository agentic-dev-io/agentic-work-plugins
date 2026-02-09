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
| Cloud storage | `~~cloud storage` | Dokumente als Dateien → DuckDB Import (Metadata + Fulltext) | fts, json |
| CLM | `~~CLM` | Contract-Export (CSV/JSON) → DuckDB Import | json, fts, vss |
| CRM | `~~CRM` | CRM-Export → DuckDB Import (Vendor/Client-Daten) | json |
| Office suite | `~~office suite` | Dokumente als Text/JSON → DuckDB Import | fts, vss |
| Project tracker | `~~project tracker` | JSON/CSV Export aus Jira/Linear → DuckDB Import | json, fts |

### DuckDB als Primary Data Layer

DuckDB ersetzt direkte MCP-Verbindungen zu externen Systemen. Stattdessen:

1. **Verträge/Dokumente**: Text/JSON Export → DuckDB Import mit FTS-Index für Klausel-Suche
2. **Semantic Search**: VSS Extension für ähnliche Verträge, NDAs und Klauseln
3. **Fuzzy Matching**: Rapidfuzz Extension für ähnliche NDA-Klauseln und Vertragsbestimmungen
4. **Strukturierte Metadaten**: JSON Extension für Contract-Metadata und Klausel-Hierarchien
5. **Compliance**: YAML Extension für Compliance-Checklisten und Review-Workflows

Siehe [DUCKDB.md](./DUCKDB.md) für detaillierte Extension-Configs und SQL-Beispiele.
