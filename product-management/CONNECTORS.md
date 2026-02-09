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
| Project tracker | `~~project tracker` | JSON/CSV Export aus Jira/Linear/Asana → DuckDB Import | json, fts |
| Knowledge base | `~~knowledge base` | Content-Export → DuckDB Import mit FTS + VSS | fts, vss |
| Product analytics | `~~product analytics` | Analytics-Export (CSV/JSON/Parquet) → DuckDB Import | json, Parquet, fts |
| User feedback | `~~user feedback` | Feedback-Export (CSV/JSON) → DuckDB Import mit FTS + VSS | fts, vss |
| Design | `~~design` | Design-Specs als JSON/Markdown → DuckDB Import | json |
| Meeting transcription | `~~meeting transcription` | Transkript-Export → DuckDB Import mit FTS | fts, vss |

### DuckDB als Primary Data Layer

DuckDB ersetzt direkte MCP-Verbindungen zu externen Systemen. Stattdessen:

1. **Tracker-Daten**: JSON/CSV Export aus Jira, Linear, Asana → DuckDB Import mit FTS-Index
2. **User Research**: Feedback und Interview-Daten → DuckDB mit VSS für Themen-Clustering
3. **Product Analytics**: Amplitude/Mixpanel Export → DuckDB als Parquet für Adoption-Metrics
4. **Knowledge Base**: Content-Export → FTS für Volltext-Suche, VSS für semantische Suche
5. **Competitor Research**: Webbed Extension für Web-Scraping von Competitor-Pages
6. **Roadmap-Daten**: YAML Extension für Roadmap-Configs, Pivot Tables für Prioritisierung

Siehe [DUCKDB.md](./DUCKDB.md) für detaillierte Extension-Configs und SQL-Beispiele.
