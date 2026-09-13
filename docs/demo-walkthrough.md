# Demo Walkthrough — Five Use Cases

The demo runs against a small **Acme** model in Data 360:

- `AcmeCustomer__dlm` — customers (PK `CustomerId__c`)
- `AcmeOrder__dlm`, `AcmeSubscription__dlm`, `AcmeSupportTicket__dlm` — child DMOs joined to the customer on `CustomerId__c`
- `AcmeCustomerLifetimeValue__cio` — a Calculated Insight over Customer ⨝ Order
- `AcmeSubscriptionChurn__cio` — a Calculated Insight over Subscription

Each use case shows the **question**, the **endpoint/tool** that answers it, the
**call** (MCP + REST), and the **expected result**. Narrative order: describe the
model → discover an asset → inspect it → trace its lineage → assess its impact.

> Identifiers below are illustrative placeholders of the form
> `{orgId}::{objectId}::SObject`. Yours will differ — get them from Search or
> Inspect-by-name.

The MCP calls use the Data 360 MCP server's `execute` facade tool
(`toolName` + `paramsJson`).

---

## 1. Explore the Model — *Describe*

> **"What kinds of objects can the catalog model, and what properties does it track for a DMO?"**

**Tool:** `d360_catalog_metadata_type_describe` · **REST:** `GET /ssot/catalog-metadata/types/SObject`

```jsonc
// MCP
{ "toolName": "d360_catalog_metadata_type_describe",
  "paramsJson": "{\"typeName\":\"SObject\"}" }
```

**Returns:** the JSON Schema for `SObject` — ~22 subtypes (`DataModelObject`,
`DataLakeObject`, `CalculatedInsightObject`, `CustomObject`…) and the property set,
with DMO-scoped properties like `dataCloud.isSegmentable`, `isUsedForMetrics`,
`creationType`, `dataSpaces`, `queryable`. Pair with `.../types` (or
`d360_catalog_metadata_type_list`) to get the valid type names first.

---

## 2. Semantic Search — *Search*

> **"Show me anything related to churn or subscription cancellations."**

**Tool:** `d360_catalog_metadata_search` · **REST:** `GET /ssot/catalog-metadata/search?query=churn+or+subscription+cancellations&limit=5`

```jsonc
// MCP
{ "toolName": "d360_catalog_metadata_search",
  "paramsJson": "{\"catalogMetadataSearchInputRepresentation\":{\"query\":\"churn or subscription cancellations\"},\"limit\":5}" }
```

**Returns:** a ranked list of descriptors, top hit `AcmeSubscriptionChurn__cio` (it
surfaces both as the `__cio` object and as its `MktCalculatedInsight` definition).
Each descriptor carries the identifier you use in the next steps.

> `limit` is a **top-level sibling** of the search representation, not nested inside it.

---

## 3. Profile an Asset — *Inspect*

> **"What fields does AcmeCustomer have, and which is the primary key?"**

A three-step chain: resolve by name → read profile → list fields.

```jsonc
// (a) resolve identifier by composite key
{ "toolName": "d360_catalog_metadata_lookup",
  "paramsJson": "{\"metadataType\":\"SObject\",\"name\":\"AcmeCustomer__dlm\",\"metadataSubType\":\"DataModelObject\"}" }
// → identifier: {orgId}::{objectId}::SObject

// (b) full profile by identifier
{ "toolName": "d360_catalog_metadata_get",
  "paramsJson": "{\"identifier\":\"{orgId}::{objectId}::SObject\"}" }

// (c) the fields (children)
{ "toolName": "d360_catalog_metadata_children_list",
  "paramsJson": "{\"identifier\":\"{orgId}::{objectId}::SObject\",\"limit\":200}" }
```

**REST:** `GET /ssot/catalog-metadata/?metadataType=SObject&name=AcmeCustomer__dlm&metadataSubType=DataModelObject`
then `GET /ssot/catalog-metadata/{identifier}/children?limit=200`

**Returns:** the customer profile plus its fields — business fields
(`CustomerId__c`, `FullName__c`, `Email__c`, `City__c`, `LifetimeValue__c`,
`SignupDate__c`) and system fields. The **primary key** is `CustomerId__c`
(`primaryIndexOrder: 1`, key qualifier `KQ_CustomerId`).

> `get` alone returns the object's attributes; you need `children` to answer
> "what fields / which PK".

---

## 4. Trace Lineage — *Relationships (upstream)*

> **"What is the source of the Customer Lifetime Value insight?"**

**Tool:** `d360_catalog_metadata_relationships_lookup`

```jsonc
// MCP
{ "toolName": "d360_catalog_metadata_relationships_lookup",
  "paramsJson": "{\"metadataType\":\"SObject\",\"name\":\"AcmeCustomerLifetimeValue__cio\",\"metadataSubType\":\"CalculatedInsightObject\",\"upstreamDepth\":2,\"downstreamDepth\":0,\"includeFieldDetails\":true}" }
```

**REST:** `GET /ssot/catalog-metadata/relationships?metadataType=SObject&name=AcmeCustomerLifetimeValue__cio&metadataSubType=CalculatedInsightObject&upstreamDepth=2&downstreamDepth=0`

**Returns:** the CI's sources — `AcmeCustomer__dlm` + `AcmeOrder__dlm` — reached via
the `MktCalculatedInsight` definition node (`Primary` edges). `metadataType` is
required.

---

## 5. Analyze Impact — *Relationships (downstream)*

> **"What breaks if I remove OrderTotal__c from AcmeOrder?"**

**Tool:** `d360_catalog_metadata_relationships_lookup` (downstream)

```jsonc
// MCP
{ "toolName": "d360_catalog_metadata_relationships_lookup",
  "paramsJson": "{\"metadataType\":\"SObject\",\"name\":\"AcmeOrder__dlm\",\"metadataSubType\":\"DataModelObject\",\"upstreamDepth\":0,\"downstreamDepth\":2,\"includeFieldDetails\":true}" }
```

**REST:** `GET /ssot/catalog-metadata/relationships?metadataType=SObject&name=AcmeOrder__dlm&metadataSubType=DataModelObject&upstreamDepth=0&downstreamDepth=2`

**Returns:** downstream consumers — the `AcmeCustomerLifetimeValue` CI (→ its `__cio`)
depends on AcmeOrder, plus an `EntityJoinRelationship` join edge to `AcmeCustomer__dlm`
on `CustomerId__c` / `KQ_CustomerId`. That's your blast radius before you touch a field.

---

## The through-line

**Describe → Search → Inspect → Trace Lineage → Analyze Impact.** Understand the model,
discover an asset, read it in full, walk its sources, then assess what depends on it —
each step composes with the next, and each stands on its own.
