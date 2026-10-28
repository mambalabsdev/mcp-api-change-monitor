# API Change Monitor MCP Server

[![Smithery](https://smithery.ai/badge/mambabuilt/mcp-api-change-monitor)](https://smithery.ai/servers/mambabuilt/mcp-api-change-monitor) [![Glama score](https://glama.ai/mcp/servers/mambalabsdev/mcp-api-change-monitor/badges/score.svg)](https://glama.ai/mcp/servers/mambalabsdev/mcp-api-change-monitor) [![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%3Fsearch%3Dcom.mambabuilt%252Fmcp-api-change-monitor%26limit%3D1&query=%24.servers%5B0%5D._meta%5B%22io.modelcontextprotocol.registry%2Fofficial%22%5D.status&label=mcp%20registry&color=blue)](https://registry.modelcontextprotocol.io/v0/servers?search=com.mambabuilt/mcp-api-change-monitor&limit=1) [![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-api-change-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-api-change-monitor) [![npm downloads](https://img.shields.io/npm/dm/@mambalabsdev/mcp-api-change-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-api-change-monitor) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-api-change-monitor)](https://github.com/mambalabsdev/mcp-api-change-monitor/blob/main/LICENSE) [![mcpservers.org](https://img.shields.io/badge/mcpservers.org-listed-blue)](https://mcpservers.org/servers/mambalabsdev/mcp-api-change-monitor)

An MCP server for the Mamba Labs API Change Monitor actor on Apify. Give it API vendors or URLs. Each run compares every source with the state saved by the last run and returns only what changed.

## What's Inside

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Prerequisites](#prerequisites)
- [Example prompts](#example-prompts)
- [Tool](#tool)
- [Inputs](#inputs)
- [Pricing](#pricing)
- [Your keys](#your-keys)
- [Full actor documentation](#full-actor-documentation)
- [License](#license)

## What it does

- **Changelogs**: new entries from the vendor's changelog feed or page, tagged `deprecation_notice` or `breaking_change` when the wording says so, with the first `sunset_date` named.
- **Status**: new incidents and status changes from the vendor's status history feed.
- **OpenAPI**: operations added, removed, or deprecated, and the API version, from a published OpenAPI JSON document.
- **Headers**: `Sunset` and `Deprecation` response headers on API endpoints you list.
- **One row per source per run**: `source_baseline` on the first check under a `state_key`, then `source_check` with `changes_found`, or a free `no_change` row when nothing moved.

All of the work runs on Apify. This package is a thin client that starts the actor run, waits for it, and hands back the dataset rows.

## Quick start

You need Node.js 18 or newer and an Apify account with an API token.

Add this to your Claude Desktop config (`claude_desktop_config.json`), Cursor, or any MCP client:

```json
{
  "mcpServers": {
    "mamba-api-change-monitor": {
      "command": "npx",
      "args": [
        "-y",
        "@mambalabsdev/mcp-api-change-monitor"
      ],
      "env": {
        "APIFY_TOKEN": "your-apify-token"
      }
    }
  }
}
```

Restart the client. The `monitor_api_changes` tool appears.

## Prerequisites

- Node.js 18 or newer.
- An Apify API token from [console.apify.com/account/integrations](https://console.apify.com/account/integrations), set as `APIFY_TOKEN`. The server lists its tool without a token; a call needs one.

## Example prompts

- "Did GitHub or OpenAI announce any deprecations or breaking changes since last week?"
- "Watch the Stripe and GitHub OpenAPI documents and tell me which operations were removed or deprecated."
- "Has any incident been opened on the Cloudflare or Vercel status pages since yesterday?"

## Tool

`monitor_api_changes`. One tool. Example arguments:

```json
{
  "vendors": [
    "github",
    "openai"
  ],
  "state_key": "api-watch-daily"
}
```

## Inputs

Every buyer facing input of the actor, unchanged. Omitted inputs take the actor's own defaults. Numbers and booleans are sent as strings.

| Input | Type | Description |
|---|---|---|
| `vendors` | array | API vendors to watch, one per line. Each expands to the public sources read for it: changelog, deprecation page, status history, and OpenAPI document where one is published. Available: github, stripe, shopify, hubspot, openai, anthropic, google-ads, twilio, slack, notion, atlassian, cloudflare, vercel, plaid |
| `sources` | array | Any other public URLs to watch, one per line: a changelog or release notes page, an RSS or Atom feed, a deprecation or sunset dates page, a Statuspage site or its history.rss feed, or an OpenAPI or Swagger JSON document. The kind is detected from what the URL returns |
| `endpoint_urls` | array | API endpoints to call without credentials, one per line. The response headers are read for Sunset (RFC 8594), Deprecation (RFC 9745), and Link relations deprecation, sunset, and successor-version. Only GET requests are sent |
| `watch` | array | For vendor presets, any of: changelog, deprecations, status, headers, openapi. Empty means changelog, deprecations, status, and headers. openapi reads the vendor's OpenAPI document (up to 14 MB) and reports operations added, removed, or deprecated. Without changelog, changelog sources return only deprecations and breaking changes. URLs you list in Source URLs are always read |
| `keywords` | array | Return only change rows whose title, summary, categories, or operation contains one of these words. Every change still updates the saved state |
| `only_deprecations_and_breaking` | string | true to return only change rows tagged as a deprecation or breaking change, OpenAPI operations removed or deprecated, and header changes. Incidents and plain changelog entries are left out Default false |
| `max_changes_per_source` | string | Change rows returned per source per run, 1 to 1000. The rest are counted in changes_found Default 100 |
| `include_snapshot` | string | true to also return the current entries, incidents, or operations of a source on its first check, up to Max change rows per source. Each snapshot row is billed as a snapshot row Default false |
| `state_key` | string | Names the saved state, so separate schedules keep separate baselines. Runs with the same state key and source compare against each other |
| `reset_state` | string | true to ignore the saved state and record a new baseline for every source Default false |
| `max_charge_usd` | string | Optional. The run stops before it would charge more than this |
| `proxy_mode` | string | How sources are fetched Default auto |

## Pricing

Pay per event on Apify, billed to your Apify account. Prices in USD.

| Event | Free plan | Bronze | Silver | Gold |
|---|---|---|---|---|
| `change-row`, one change on a watched source since the last run: a new changelog entry, deprecation notice, or breaking change, a new or updated status incident, an OpenAPI operation added, removed, or deprecated, an API version change, or a Sunset or Deprecation header change | $0.0035 | $0.003325 | $0.00315 | $0.002975 |
| `source-check`, one source read and compared with its saved state: the first check of a source, which records the baseline, or a later check that finds at least one change. A source where nothing moved returns a free no_change row, and a source that cannot be read is free | $0.003 | $0.00285 | $0.0027 | $0.00255 |
| `snapshot-row`, one current entry, incident, or operation on the first check of a source, returned only when include_snapshot is on | $0.001 | $0.00095 | $0.0009 | $0.00085 |
| `apify-actor-start`, per run start | $0.001 | $0.001 | $0.001 | $0.001 |

A source where nothing moved returns a free `no_change` row, sources that cannot be read are free, and so is the one row a refused input returns.

## Your keys

The actor needs no key. It reads public changelogs, feeds, status history, and OpenAPI documents, and calls the endpoints you list without credentials. It holds no key of any kind. Proxy traffic adds nothing to your price: Mamba Labs pays the platform costs of the run.

## Full actor documentation

[apify.com/mambalabs/api-change-monitor](https://apify.com/mambalabs/api-change-monitor)

## License

MIT. See [LICENSE](LICENSE).
