# Catalog APIs — Endpoint Cheat Sheet

All endpoints are **read-only** and live under the base path `/ssot/catalog-metadata`.

Legend: `?` after a parameter = **optional**. Everything else is required. `{…}` = path segment.

---

## 1. Describe — what kinds of metadata exist?

Returns metadata **types** and their **subtypes**, plus the properties each type carries
(as a JSON Schema). No identifier needed — this is static platform schema.

```
GET /types                     → list every metadata type (e.g. SObject, Flow, ApexClass…)
GET /types/{typeName}          → one type's subtypes + property definitions
```

- `SObject` has subtypes like `DataModelObject` (`__dlm`), `DataLakeObject` (`__dll`),
  `CalculatedInsightObject` (`__cio`), `CustomObject` (`__c`)…
- Fields are their own type: `SObjectField` (subtypes `CustomDataModelField`, `StandardField`…).
- Properties can be scoped to specific subtypes (e.g. DMO-only props like
  `dataCloud.isSegmentable`, `isUsedForMetrics`, `dataSpaces`).

---

## 2. Search — find assets by keyword / intent

Hybrid (keyword + semantic) search. Returns a **ranked list of lightweight asset
descriptors** — array order is relevance; each descriptor includes the asset's
**identifier** for follow-up calls.

```
GET /search?query=…&limit=?&metadataType=?&metadataSubType=?
```

| Param | Required | Notes |
|-------|----------|-------|
| `query` | ✅ | keyword or natural-language intent |
| `limit` | optional | page size |
| `metadataType` | optional | narrow to one type, e.g. `SObject` |
| `metadataSubType` | optional | narrow further, e.g. `CalculatedInsightObject` |

> One of two ways to obtain an **identifier** (the other is Inspect-by-name below).

---

## 3. Inspect — read one asset you already know

Two addressing modes reach the same asset:

```
GET /?metadataType=…&name=…&metadataSubType=?    → full profile, by composite key (name)
GET /{identifier}                                → full profile, by identifier
GET /{identifier}/children?limit=?&offset=?      → the asset's children (e.g. an object's fields)
```

| Param | Required | Notes |
|-------|----------|-------|
| `metadataType` | ✅ | e.g. `SObject` |
| `name` | ✅ | the API name, e.g. `AcmeCustomer__dlm` |
| `metadataSubType` | optional | e.g. `DataModelObject` |
| `limit` / `offset` | optional | paging for `children` |

**What the profile returns** (beyond the object's own attributes): the identifier,
type/subtype, label, data space, key qualifiers, primary-key info, and — via
`children` — the field list. To page through **fields**, you need the identifier
(`/{identifier}/children`); the by-name call gives you the profile + identifier to
start from.

> The by-name call is the **second** way to get an identifier (the first is Search).

---

## 4. Relationships — how is an asset connected?

Traverse the metadata graph **upstream** (lineage / sources) or **downstream**
(impact / consumers).

```
GET /{identifier}/relationships?upstreamDepth=?&downstreamDepth=?
GET /relationships?metadataType=…&name=…&metadataSubType=?&upstreamDepth=?&downstreamDepth=?
```

| Param | Required | Notes |
|-------|----------|-------|
| `metadataType` | ✅ (by-name form) | e.g. `SObject` |
| `name` | ✅ (by-name form) | e.g. `AcmeCustomerLifetimeValue__cio` |
| `metadataSubType` | optional | e.g. `CalculatedInsightObject` |
| `upstreamDepth` | optional* | how far to walk toward sources |
| `downstreamDepth` | optional* | how far to walk toward consumers |

> \* Both depths default to 0, but **you must set at least one to a non-zero value** —
> a call with no traversal depth returns an error. Keep depths small (1–2) for
> interactive use; deep two-directional traversals are slow.

**Edge types you'll see in responses:**
- `Primary` — lineage edges (e.g. a Calculated Insight to its source DMOs, routed
  through the `MktCalculatedInsight` definition node).
- `EntityJoinRelationship` — a foreign-key join between two DMOs (on the join field,
  e.g. `CustomerId__c`).

---

## Note on Calculated Insights

A Calculated Insight appears **twice** in the catalog:
- as `SObject` / `CalculatedInsightObject` — the `__cio` **result table** you query and profile;
- as `MktCalculatedInsight` — the **definition** that carries its lineage.

That's why tracing lineage from a `__cio` routes *through* the `MktCalculatedInsight`
node on its way to the source DMOs.
