# Discover Data 360 Metadata via Salesforce Catalog APIs

> **Dreamforce 2026 · Session 3625** — companion resources for the talk.

The **Salesforce Catalog APIs** are a set of read-only, progressive APIs (under
`/ssot/catalog-metadata`) that let you introspect the metadata of your org — both
**core platform** metadata (objects, fields, Apex, Flows, permission sets…) and
**Data 360** metadata (data lake objects, data model objects, calculated insights,
data streams…) — through one unified catalog.

Because everything in Salesforce is metadata-driven, the platform can describe
itself. These APIs expose that self-description so your code and your agents can
discover, inspect, and trace the metadata graph.

---

## The four moves

The APIs compose — each can feed the next — but every one stands alone.

| Verb | What it answers | Endpoints |
|------|-----------------|-----------|
| **Describe** | What *kinds* of metadata exist, and what properties they carry? | `/types` · `/types/{typeName}` |
| **Search** | Find assets by keyword / intent (hybrid search) | `/search?query=…` |
| **Inspect** | Read the full profile of one asset, or list its children | `/?metadataType=…&name=…` · `/{identifier}` · `/{identifier}/children` |
| **Relationships** | How is an asset connected — upstream (lineage) or downstream (impact)? | `/{identifier}/relationships` · `/relationships?metadataType=…&name=…` |

Full parameter reference: **[docs/api-cheatsheet.md](docs/api-cheatsheet.md)**.

---

## Two front doors

The same catalog is reachable two ways:

- **MCP** — for your **agents**. The hosted **Data 360 MCP server** exposes the catalog
  through three facade tools (`search` → `payload_examples` → `execute`).
- **REST** — for your **code**. Call the endpoints directly under `/ssot/catalog-metadata`.

Built into your app, your agent, or your CI pipeline.

---

## Contents

- **[docs/api-cheatsheet.md](docs/api-cheatsheet.md)** — every endpoint with its required and optional parameters, plus response shapes.
- **[docs/demo-walkthrough.md](docs/demo-walkthrough.md)** — the five demo use cases (Explore → Search → Profile → Trace Lineage → Analyze Impact) with the exact MCP and REST calls and expected responses.

---

## Resources

### Data 360 MCP server
- [Introducing the Data 360 MCP Server (Developer Preview)](https://developer.salesforce.com/blogs/2026/05/introducing-the-data-360-mcp-server-developer-preview) — announcement blog
- [Data 360 MCP server — guide](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/data360-mcp.html)
- [Data 360 MCP server — reference](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/references/reference/data360-mcp.html)
- [`forcedotcom/d360-mcp-server`](https://github.com/forcedotcom/d360-mcp-server) — open-source implementation (facade-tool architecture)

### Salesforce Catalog & Catalog APIs
- The Catalog Metadata APIs are currently surfaced through the Data 360 MCP server (see the guide/reference above). A dedicated public API reference is expected at GA.
- [Salesforce Hosted MCP Servers — servers reference](https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/servers-reference.html)

### Metadata grounding (core metadata for AI)
> These are the closest publicly documented "metadata grounding" servers. Names may change — confirm against current docs.
- [Salesforce API Context MCP Server (Beta)](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_salesforce_api_mcp_intro.htm) — supplies metadata-type context to ground AI-generated metadata
- [Salesforce Hosted MCP Servers (Salesforce Help)](https://help.salesforce.com/s/articleView?id=platform.hosted_mcp_servers.htm)

### Protocol
- [Model Context Protocol](https://modelcontextprotocol.io/)

---

## Disclaimer

The Catalog APIs are pre-GA ("GA soon") and the Data 360 MCP server is in Developer
Preview at the time of the talk. Endpoint shapes and server names may change. This is
a community resource for the session, not official Salesforce documentation — always
defer to the official docs linked above.
