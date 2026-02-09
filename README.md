# Knowledge Work Plugins

Plugins that turn Claude into a specialist for your role, team, and company. Built for [Claude Cowork](https://claude.com/product/cowork), also compatible with [Claude Code](https://claude.com/product/claude-code).

## Architecture: DuckDB-First

All plugins use **DuckDB as the primary data layer** — no external MCP servers, no OAuth. Data flows through DuckDB as the central analytical layer.

- **DuckDB CLI** is installed system-wide for direct terminal access
- **Extensions**: VSS (vector search), FTS (full-text search), Parquet, JSON, HTTPFS, and domain-specific extensions per plugin
- **Import**: CSV/JSON/Parquet exports from your tools → `duckdb` CLI import
- **On-Demand MCP**: External MCP servers (Slack, Notion, etc.) can be connected via the `duckdb_mcp` community extension when needed — see [MCP-SERVERS.md](./MCP-SERVERS.md)

Each plugin's `DUCKDB.md` has SQL examples, extension configs, and common queries. Each plugin's `CONNECTORS.md` documents the import methods.

## Why Plugins

Cowork lets you set the goal and Claude delivers finished, professional work. Plugins let you go further: tell Claude how you like work done, which tools and data to pull from, how to handle critical workflows, and what slash commands to expose — so your team gets better and more consistent outcomes.

Each plugin bundles the skills, connectors, slash commands, and sub-agents for a specific job function. Out of the box, they give Claude a strong starting point for helping anyone in that role. The real power comes when you customize them for your company — your tools, your terminology, your processes — so Claude works like it was built for your team.

## Plugin Marketplace

We're open-sourcing 11 plugins built and inspired by our own work:

| Plugin | How it helps | Connectors |
|--------|-------------|------------|
| Plugin | How it helps | Data Layer |
|--------|-------------|------------|
| **[productivity](./productivity)** | Manage tasks, calendars, daily workflows, and personal context so you spend less time repeating yourself. | DuckDB (FTS, JSON) |
| **[sales](./sales)** | Research prospects, prep for calls, review your pipeline, draft outreach, and build competitive battlecards. | DuckDB (VSS, FTS, Parquet) |
| **[customer-support](./customer-support)** | Triage tickets, draft responses, package escalations, research customer context, and turn resolved issues into knowledge base articles. | DuckDB (FTS, JSON) |
| **[product-management](./product-management)** | Write specs, plan roadmaps, synthesize user research, keep stakeholders updated, and track the competitive landscape. | DuckDB (VSS, FTS, Parquet) |
| **[marketing](./marketing)** | Draft content, plan campaigns, enforce brand voice, brief on competitors, and report on performance across channels. | DuckDB (VSS, FTS, Parquet) |
| **[legal](./legal)** | Review contracts, triage NDAs, navigate compliance, assess risk, prep for meetings, and draft templated responses. | DuckDB (FTS, VSS) |
| **[finance](./finance)** | Prep journal entries, reconcile accounts, generate financial statements, analyze variances, manage close, and support audits. | DuckDB (Warehouse Extensions) |
| **[data](./data)** | Query, visualize, and interpret datasets — write SQL, run statistical analysis, build dashboards, and validate your work before sharing. | DuckDB (Snowflake, BigQuery, Databricks) |
| **[enterprise-search](./enterprise-search)** | Find anything across email, chat, docs, and wikis — one query across all your company's tools. | DuckDB (FTS) |
| **[bio-research](./bio-research)** | Connect to preclinical research tools and databases (literature search, genomics analysis, target prioritization) to accelerate early-stage life sciences R&D. | DuckDB (VSS, FTS) |
| **[cowork-plugin-management](./cowork-plugin-management)** | Create new plugins or customize existing ones for your organization's specific tools and workflows. | — |

Install these directly from Cowork, browse the full collection here on GitHub, or build your own.

## Getting Started

### Cowork

Install plugins from [claude.com/plugins](https://claude.com/plugins/).

### Claude Code

```bash
# Add the marketplace first
claude plugin marketplace add agentic-dev-io/knowledge-work-plugins

# Then install a specific plugin
claude plugin install sales@agentic-work-plugins
```

Once installed, plugins activate automatically. Skills fire when relevant, and slash commands are available in your session (e.g., `/sales:call-prep`, `/data:write-query`).

## How Plugins Work

Every plugin follows the same structure:

```
plugin-name/
├── .claude-plugin/plugin.json   # Manifest
├── .mcp.json                    # Tool connections
├── commands/                    # Slash commands you invoke explicitly
└── skills/                      # Domain knowledge Claude draws on automatically
```

- **Skills** encode the domain expertise, best practices, and step-by-step workflows Claude needs to give you useful help. Claude draws on them automatically when relevant.
- **Commands** are explicit actions you trigger (e.g., `/finance:reconciliation`, `/product-management:write-spec`).
- **Connectors** document the data sources and import methods for each plugin. External MCP servers can be connected on-demand via `duckdb_mcp` — see [MCP-SERVERS.md](./MCP-SERVERS.md).

Every component is file-based — markdown and JSON, no code, no infrastructure, no build steps.

## Making Them Yours

These plugins are generic starting points. They become much more useful when you customize them for how your company actually works:

- **Import your data** — Export CSV/JSON/Parquet from your tools and import into DuckDB via CLI.
- **Connect MCP servers** — Use `duckdb_mcp` extension to attach external MCP servers on-demand (see [MCP-SERVERS.md](./MCP-SERVERS.md)).
- **Add company context** — Drop your terminology, org structure, and processes into skill files so Claude understands your world.
- **Adjust workflows** — Modify skill instructions to match how your team actually does things, not how a textbook says to.
- **Build new plugins** — Use the `cowork-plugin-management` plugin or follow the structure above to create plugins for roles and workflows we haven't covered yet.

As your team builds and shares plugins, Claude becomes a cross-functional expert. The context you define gets baked into every relevant interaction, so leaders and admins can spend less time enforcing processes and more time improving them.

## Contributing

Plugins are just markdown files. Fork the repo, make your changes, and submit a PR.
