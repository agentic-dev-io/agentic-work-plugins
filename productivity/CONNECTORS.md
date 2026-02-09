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
| Project tracker | `~~project tracker` | JSON/CSV Export aus Asana/Linear/Jira → DuckDB Import | json, fts |
| Knowledge base | `~~knowledge base` | Notes/Docs als JSON/Markdown → DuckDB Import mit FTS | fts, vss |
| Calendar | `~~calendar` | ICS/JSON Export → DuckDB Import | json |
| Email | `~~email` | — (Output: Copy to email client) | — |
| Office suite | `~~office suite` | Dokumente als JSON/Text → DuckDB Import | json, fts |

### DuckDB als Primary Data Layer

DuckDB ersetzt direkte MCP-Verbindungen zu externen Systemen. Stattdessen:

1. **Tasks**: JSON/CSV Export aus Asana, Linear, Jira → DuckDB Import mit FTS-Index
2. **Kalender**: ICS/JSON Export → DuckDB Import für Meeting-Analytics und Focus-Time-Tracking
3. **Notizen**: Content-Export aus Notion/Confluence → FTS für Volltext-Suche, VSS für Related Notes
4. **Workflow-Configs**: YAML Extension für Workflow-Automation Konfigurationen
5. **Multi-Tool Aggregation**: JSONata Extension für Normalisierung aus verschiedenen Tools

Siehe [DUCKDB.md](./DUCKDB.md) für detaillierte Extension-Configs und SQL-Beispiele.
