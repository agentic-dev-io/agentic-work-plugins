# MCP Server Referenz

On-Demand Verbindung via DuckDB `duckdb_mcp` Extension:

```sql
INSTALL duckdb_mcp FROM community;
LOAD duckdb_mcp;

-- Server verbinden
ATTACH 'mcp:https://url/mcp' AS servername;
```

## Bio-Research

| Server | URL |
|--------|-----|
| pubmed | `https://pubmed.mcp.claude.com/mcp` |
| biorender | `https://mcp.services.biorender.com/mcp` |
| biorxiv | `https://mcp.deepsense.ai/biorxiv/mcp` |
| c-trials | `https://mcp.deepsense.ai/clinical_trials/mcp` |
| chembl | `https://mcp.deepsense.ai/chembl/mcp` |
| synapse | `https://mcp.synapse.org/mcp` |
| wiley | `https://connector.scholargateway.ai/mcp` |
| owkin | `https://mcp.k.owkin.com/mcp` |
| ot | `https://mcp.platform.opentargets.org/mcp` |
| benchling | *(URL nicht konfiguriert)* |

## Customer Support

| Server | URL |
|--------|-----|
| slack | `https://mcp.slack.com/mcp` |
| intercom | `https://mcp.intercom.com/mcp` |
| hubspot | `https://mcp.hubspot.com/anthropic` |
| guru | `https://mcp.api.getguru.com/mcp` |
| atlassian | `https://mcp.atlassian.com/v1/mcp` |
| notion | `https://mcp.notion.com/mcp` |
| ms365 | `https://microsoft365.mcp.claude.com/mcp` |

## Enterprise Search

| Server | URL |
|--------|-----|
| slack | `https://mcp.slack.com/mcp` |
| notion | `https://mcp.notion.com/mcp` |
| guru | `https://mcp.api.getguru.com/mcp` |
| atlassian | `https://mcp.atlassian.com/v1/mcp` |
| asana | `https://mcp.asana.com/v2/mcp` |
| ms365 | `https://microsoft365.mcp.claude.com/mcp` |

## Alle Unique URLs

| Server | URL | Plugins |
|--------|-----|---------|
| slack | `https://mcp.slack.com/mcp` | customer-support, enterprise-search |
| notion | `https://mcp.notion.com/mcp` | customer-support, enterprise-search |
| atlassian | `https://mcp.atlassian.com/v1/mcp` | customer-support, enterprise-search |
| guru | `https://mcp.api.getguru.com/mcp` | customer-support, enterprise-search |
| ms365 | `https://microsoft365.mcp.claude.com/mcp` | customer-support, enterprise-search |
| hubspot | `https://mcp.hubspot.com/anthropic` | customer-support |
| intercom | `https://mcp.intercom.com/mcp` | customer-support |
| asana | `https://mcp.asana.com/v2/mcp` | enterprise-search |
| pubmed | `https://pubmed.mcp.claude.com/mcp` | bio-research |
| biorender | `https://mcp.services.biorender.com/mcp` | bio-research |
| biorxiv | `https://mcp.deepsense.ai/biorxiv/mcp` | bio-research |
| c-trials | `https://mcp.deepsense.ai/clinical_trials/mcp` | bio-research |
| chembl | `https://mcp.deepsense.ai/chembl/mcp` | bio-research |
| synapse | `https://mcp.synapse.org/mcp` | bio-research |
| wiley | `https://connector.scholargateway.ai/mcp` | bio-research |
| owkin | `https://mcp.k.owkin.com/mcp` | bio-research |
| ot | `https://mcp.platform.opentargets.org/mcp` | bio-research |
