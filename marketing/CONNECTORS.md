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
| Marketing automation | `~~marketing automation` | Campaign-Export (CSV/JSON) → DuckDB Import | json, Parquet, fts |
| Product analytics | `~~product analytics` | Analytics-Export (CSV/JSON) → DuckDB Import | json, Parquet |
| Knowledge base | `~~knowledge base` | Content-Export → DuckDB Import mit FTS | fts, vss |
| SEO | `~~SEO` | SEO-Tool Export (CSV) → DuckDB Import | Parquet, fts |
| Email marketing | `~~email marketing` | Email-Platform Export (CSV/JSON) → DuckDB Import | json, Parquet |
| Design | `~~design` | Asset-Metadata als JSON → DuckDB Import | json |

### DuckDB als Primary Data Layer

DuckDB ersetzt direkte MCP-Verbindungen zu externen Systemen. Stattdessen:

1. **Campaign-Daten**: CSV/JSON Export aus HubSpot, Marketo → DuckDB Import als Parquet
2. **SEO-Daten**: Ahrefs/Semrush CSV-Export → DuckDB Import mit FTS für Keyword-Analyse
3. **Content-Bibliothek**: FTS Extension für Volltextsuche in Marketing-Content
4. **Semantic Search**: VSS Extension für ähnliche Kampagnen und Content-Themes
5. **A/B-Tests**: ANOFOX Statistics für statistische Signifikanz-Tests
6. **Personalisierung**: MiniJinja Extension für Template-basierte Content-Generierung

Siehe [DUCKDB.md](./DUCKDB.md) für detaillierte Extension-Configs und SQL-Beispiele.
