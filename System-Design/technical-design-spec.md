# Technical Design Specification
## A Metadata-Driven, Modular Platform Framework

**Companion to:** [`requirements.md`](./requirements.md) and [`architecture.md`](./architecture.md).
**Convention:** This spec defines the **meta-model** (the schema that describes schemas) and **module contracts** only. It intentionally contains **no business-domain schema** (no fixed `Asset`, `WorkOrder`, etc. tables/fields) — those are defined at runtime, by a product, as metadata records against the meta-model in Part B.1. The Asset Management examples throughout are illustrative metadata *instances*, not part of the framework's own schema.

---

# Part A — High-Level Spec

## A.1 Module List, Responsibilities, and Tech Choice

| Module | Responsibility | Primary language | Key tech | Case-study lineage |
|---|---|---|---|---|
| Metadata & Schema Engine | Store/serve EntityDefinition, FieldDefinition, RelationshipDefinition, PermissionPolicy, WorkflowDefinition, ViewDefinition | Go | PostgreSQL (JSONB + relational catalog), gRPC | Frappe DocType, ServiceNow dictionary, LinkedIn Settings Platform |
| Entity/Data Engine | Dynamic CRUD/query API generated from metadata | Go | PostgreSQL, GraphQL/REST codegen, read replicas | Hasura/PostgREST/Directus pattern; Netflix Media DB; Walmart MTH |
| Workflow & Process Engine | Execute externally supplied process definitions | Go / Java (engine core) | Event-sourced state machine store, BPMN/JSON process defs | Camunda/Temporal pattern; LinkedIn Settings lifecycle |
| Identity & Permission Engine | AuthN/AuthZ, tenant scalability-unit routing | Go | OIDC/OAuth2, Postgres RLS, policy engine (OPA-style) | ServiceNow ACL, Salesforce permission sets |
| Event Backbone | Durable, replayable, partitioned log | — (infra) | Kafka (or Kafka-API-compatible), schema registry | LinkedIn Kafka, Monzo, Wealthsimple |
| API Gateway / Edge | Protocol-agnostic ingress, routing, rate limiting | Go | Envoy/Zanzibar-style gateway, Redis (rate limiting) | Uber Zanzibar, Tinder TAG, Riot Games gateway |
| Notification & Communication Engine | Multi-channel, multi-trigger delivery | Go / Node.js (adapters) | RabbitMQ or SQS-class queues per channel, template engine | Swiggy Kabootar, Netflix RENO |
| Sync / Offline Engine | Client-server state reconciliation | Rust or Go (client+server core) | Three-tree sync model, embedded local store (SQLite-class) on client | Dropbox Nucleus |
| ETL / Data Integration & Analytics | Batch+streaming ingestion, lakehouse tiering, dashboards | Python (pipelines) / Go (services) | Kafka Connect–class connectors, object storage, time-series DB, OLAP store | Domain research (IoT ingestion); Netflix Media DB |
| Testing & Simulation Framework | Auto test-double generation, parallel test execution, production-safe simulation | Go / Python | Container-based worker pool, scheduler (Mesos/K8s-Jobs-class) | Airbnb schema-based testing, Yelp Seagull, Netflix Simone, Netflix Hexagonal Architecture |
| Transactional Integrity & Idempotency | Idempotency-key store, optional event-sourcing/CQRS | Go | Sharded key-value store, event log projections | Airbnb idempotency, Etsy Vitess, Paytm, Nubank |
| AI-Ops / Agent Interface | Auto-generated MCP-compatible tool/resource catalog | Go / TypeScript (MCP SDK) | MCP server, policy-scoped credentials | MCP pattern; every metadata-driven precedent in `architecture.md` §2 |
| Module Registry & Contract Runtime | Module manifest registration, extension-point wiring | Go | In-process registry + distributed config store | Backstage `createBackendModule`, Frappe app/hooks model |

**Language rationale:** Go is the default for core/edge services needing predictable low-latency concurrency (directly mirroring why Uber rewrote its gateway from Node.js to Go, and why Riot Games' rate limiter and Nubank's authorizer prioritize minimal-dependency, high-throughput runtimes). Python is used only where its ecosystem dominates (data/ETL pipelines, ML). Rust is offered as an alternative for the Sync Engine's client-side core where deterministic memory behavior matters on constrained field devices. Node.js/TypeScript is acceptable for developer-facing tooling (CLIs, the MCP SDK, notification-channel adapters) where ecosystem breadth matters more than raw throughput — mirroring Medium's original rationale for splitting Node.js (developer velocity) from Go (ops simplicity) across their stack.

## A.2 Metadata Meta-Model — Overview

The meta-model has exactly six top-level constructs. A product defines its domain (e.g., Asset Management's `Asset`, `WorkOrder`) purely as *instances* of these constructs — the framework ships **zero** predefined business entities.

1. **EntityDefinition** — a named, versioned "thing" (analogous to a Frappe DocType / ServiceNow table / Salesforce custom object).
2. **FieldDefinition** — a typed attribute on an EntityDefinition.
3. **RelationshipDefinition** — a typed link between two EntityDefinitions (1:1, 1:N, N:N).
4. **PermissionPolicy** — a role/attribute-based rule scoped to an EntityDefinition, field, or operation.
5. **WorkflowDefinition** — a state machine or process graph referencing EntityDefinitions and their transitions.
6. **ViewDefinition** — a declarative description of how an EntityDefinition (or a join across several) is presented (list/form/dashboard/report), consumed by both UI renderers and the AI-ops tool-catalog generator.

## A.3 API/Contract Strategy

- All module-to-module and client-to-module contracts are OpenAPI (REST) or GraphQL-SDL first, generated from metadata where the module fronts the Entity/Data Engine, and hand-authored-but-registered for modules with fixed operational contracts (Gateway, Event Backbone, Registry).
- Every contract is versioned and published into the Module Registry (§B.2), which is also the source the AI-Ops catalog generator reads.

## A.4 Environments & Deployment Topology (summary)

Dev → Staging → Production, each a full scalability-unit-capable environment; promotion is GitOps-based (metadata + module manifests + IaC all move through the same pipeline). Full detail in §B.9.

---

# Part B — Deep Engineering Spec

## B.1 Metadata Meta-Model — Detailed Schema

These are the framework's **own** persisted schemas (the "schema of schemas"). They are fixed; what varies per product is the *data* stored in them.

```json
// EntityDefinition
{
  "id": "string (uuid)",
  "name": "string (e.g. 'asset.Asset') — namespaced by product",
  "version": "integer",
  "status": "draft | active | deprecated",
  "description": "string",
  "primaryKeyField": "string (field name)",
  "storageHint": "oltp | timeseries | document | search",
  "consistencyTier": "strong | eventual | cached",
  "tenancyScope": "global | tenant | scalability-unit",
  "auditEnabled": "boolean",
  "createdBy": "string (user or agent id)",
  "createdAt": "timestamp"
}

// FieldDefinition
{
  "id": "string (uuid)",
  "entityId": "string (FK -> EntityDefinition.id)",
  "name": "string",
  "type": "string | number | boolean | date | enum | reference | json | geo | binary-ref",
  "enumValues": ["string", "..."],           // only if type == enum
  "referenceEntityId": "string",             // only if type == reference
  "required": "boolean",
  "defaultValue": "any",
  "validation": {
    "pattern": "string (regex, optional)",
    "min": "number (optional)",
    "max": "number (optional)"
  },
  "piiClass": "none | internal | sensitive"  // drives encryption/masking defaults
}

// RelationshipDefinition
{
  "id": "string (uuid)",
  "fromEntityId": "string",
  "toEntityId": "string",
  "cardinality": "one-to-one | one-to-many | many-to-many",
  "onDelete": "cascade | restrict | set-null"
}

// PermissionPolicy
{
  "id": "string (uuid)",
  "scope": { "entityId": "string", "fieldId": "string (optional)" },
  "principalType": "role | attribute | agent-identity",
  "principalMatch": "string (role name, attribute expression, or agent-scope id)",
  "operations": ["create", "read", "update", "delete", "execute-workflow"],
  "condition": "string (optional ABAC expression, e.g. 'record.tenantId == principal.tenantId')",
  "effect": "allow | deny"
}

// WorkflowDefinition
{
  "id": "string (uuid)",
  "name": "string",
  "entityId": "string (the entity whose lifecycle this governs)",
  "states": [ { "name": "string", "isInitial": "boolean", "isFinal": "boolean" } ],
  "transitions": [
    {
      "from": "string (state name)",
      "to": "string (state name)",
      "trigger": "manual | event | schedule",
      "eventTopic": "string (if trigger == event)",
      "cronExpression": "string (if trigger == schedule)",
      "guardCondition": "string (optional expression)",
      "actions": ["string (module action id to invoke on transition)"]
    }
  ]
}

// ViewDefinition
{
  "id": "string (uuid)",
  "name": "string",
  "type": "list | form | dashboard | report",
  "primaryEntityId": "string",
  "joins": [ { "relationshipId": "string", "alias": "string" } ],
  "fields": ["string (field ids or aggregate expressions)"],
  "filters": ["string (expression)"],
  "aiOpsExposed": "boolean"   // if true, included in the AI-Ops tool catalog (§B.10)
}
```

**Illustrative instance (Asset Management, NOT part of the framework schema):** a product team would create one `EntityDefinition` row named `assetmgmt.Asset` with `FieldDefinition` rows for `serialNumber`, `assetClass`, `installDate`, etc., a `RelationshipDefinition` to `assetmgmt.WorkOrder`, and a `WorkflowDefinition` for the commissioning → active → decommissioned lifecycle — entirely as data, with no change to the framework.

## B.2 Module Contract Specification

Every module registers a manifest with the Module Registry & Contract Runtime at startup:

```json
{
  "moduleId": "string (namespaced, e.g. 'platform.notification')",
  "version": "semver",
  "providesExtensionPoints": [
    { "id": "string", "interfaceRef": "string (OpenAPI/proto ref)" }
  ],
  "consumesExtensionPoints": [
    { "extensionPointId": "string", "ownerModuleId": "string" }
  ],
  "requiredCoreServices": ["metadata-engine", "event-backbone", "identity-engine"],
  "healthCheckEndpoint": "string (URL)",
  "aiOpsCatalog": {
    "toolsEndpoint": "string (URL serving the MCP tool list for this module)",
    "resourcesEndpoint": "string (URL)"
  }
}
```

- **Lifecycle hooks**: `onRegister`, `onTenantProvisioned`, `onMetadataChanged(entityId)`, `onShutdown` — a module implements only the hooks relevant to it (mirrors Frappe's `hooks.py` and Backstage's plugin lifecycle).
- **Extension points** are the *only* sanctioned cross-module dependency. A module MUST NOT call another module's internal API directly; it calls through a declared extension point or through the Event Backbone.
- The Registry rejects a manifest that declares `consumesExtensionPoints` for an extension point no registered module `provides` — this is the mechanism that keeps the module graph valid at deploy time.

## B.3 Metadata & Schema Engine — Detailed Design

- **API**: gRPC + REST facade; `CreateEntityDefinition`, `UpdateFieldDefinition`, `PublishWorkflowDefinition`, etc. — all mutations versioned (optimistic concurrency via a `version` column).
- **Storage**: PostgreSQL, metadata tables themselves modeled as fixed relational tables (this is the one place the framework *does* have a fixed schema — the schema-of-schemas itself).
- **Change propagation**: every mutation publishes an `metadata.changed` event on the Event Backbone; the Entity/Data Engine and AI-Ops catalog generator are subscribers that regenerate their derived artifacts (API surface, tool catalog) reactively — no polling.
- **Migration model**: additive changes (new field, new entity) apply with zero downtime; destructive changes (remove field, change type) go through a two-phase deprecate → remove cycle gated by the same dry-run/approval mechanism as AI-agent high-risk operations (architecture.md §8.4), so human and agent-driven schema changes share one safety mechanism.

## B.4 Entity/Data Engine — Detailed Design

- On `metadata.changed`, regenerates: (a) the physical storage binding (table/collection per `storageHint`), (b) the REST/GraphQL surface, (c) validation middleware from `FieldDefinition.validation`.
- Query planner routes reads to the store indicated by `EntityDefinition.storageHint` and `consistencyTier` — e.g., a `cached` entity may be served from a read-replica + cache without the caller needing to know.
- Enforces `PermissionPolicy` at the row/field level before returning data (delegates evaluation to the Identity & Permission Engine; never duplicates permission logic locally).

## B.5 Workflow & Process Engine — Detailed Design

- Interprets `WorkflowDefinition` graphs; each `transitions[].actions[]` entry invokes a registered module action (via an extension point), so the engine itself never contains business logic.
- Event-triggered transitions subscribe to the named `eventTopic` on the Event Backbone; schedule-triggered transitions are registered with a cron-class scheduler service.
- State is itself stored as entity data in the Entity/Data Engine (a workflow instance is just another entity), so workflow history is queryable/reportable through the same Views mechanism as any other data.

## B.6 API Gateway / Edge — Detailed Design

- Routing table, rate-limit policy, and auth requirements per route are metadata (a `GatewayRouteDefinition`, analogous in spirit to the six core constructs but scoped to the Gateway module) — not per-endpoint code, generalizing Uber's build-time-generated-from-config approach.
- Rate limiting: token/leaky-bucket algorithm backed by a low-latency store (Redis-class), executed via a single round-trip script — directly the Riot Games pattern (sub-2ms average).
- Metrics/usage recording is asynchronous and decoupled from the request path (in-memory queue + background flush) so metering never adds request latency — directly the Riot Games pattern.

## B.7 Notification & Communication Engine — Detailed Design

- Three internal layers, independently swappable: **Integration layer** (receives triggers via event subscription or a scheduled cron job) → **Core processing layer** (applies templating, throttling, user preference/opt-out rules — transport-agnostic) → **Delivery layer** (per-channel adapters: push, email, SMS, in-app, webhook).
- Campaigns are `WorkflowDefinition`-driven (event-based or schedule-based, reusing §B.5) rather than a bespoke campaign engine — avoiding a second, parallel workflow concept.
- Per-device fan-out failures are isolated (one channel/device failing does not block others) — directly the Netflix RENO pattern.

## B.8 Sync / Offline Engine — Detailed Design

- Client maintains three logical trees: **Remote** (last known server state), **Local** (pending client changes), **Synced** (merge-base) — directly Dropbox's Nucleus model.
- Entities are referenced by stable ID, never by path/position, so structural moves are O(1) rather than O(n).
- Conflict resolution runs a deterministic three-way merge against `EntityDefinition`/`RelationshipDefinition` metadata (so merge behavior is consistent across any product's entities, not hand-coded per entity type).
- Testing: a fuzz-testing harness perturbs randomized entity trees and replays them through the merge algorithm with reproducible seeds (mirrors Dropbox's CanopyCheck), integrated with the Testing & Simulation Framework (§B.9).

## B.9 Testing & Simulation Framework — Detailed Design

- **Test-double generation**: given any `EntityDefinition`/API contract, generates mock data, mock clients, and mock servers automatically — no per-entity test-fixture code (Airbnb's schema-based testing pattern).
- **Parallel execution**: tests are bin-packed into fixed-duration bundles and scheduled across a container worker pool (Kubernetes Jobs or Mesos-class), with an autoscaler sizing the pool to queue depth — directly Yelp's Seagull/FleetMiser pattern (2 days → 30 min, ~80% cost reduction via elastic scaling).
- **Production simulation**: a `Trigger` (narrow match condition, e.g., one tenant + one entity ID) paired with a `Variant` (the simulated behavior) lets any module be safely exercised in production with a scoped blast radius — directly Netflix's Simone model. This reuses `PermissionPolicy`-style condition expressions for the Trigger match syntax, avoiding a third expression-language implementation.
- **Design-for-testability requirement**: every module's business logic must be reachable behind a framework-agnostic interface (no direct coupling to transport or a specific store), per Netflix's Hexagonal Architecture pattern — enforced structurally by the Module Contract (§B.2)'s ban on direct cross-module calls.

## B.10 Transactional Integrity & Idempotency — Detailed Design

- **Idempotency store**: every mutating call may carry an `Idempotency-Key`; the store (sharded by key, per Airbnb's pattern) records first-seen results and short-circuits duplicates.
- **Event-sourcing/CQRS mode** (opt-in per EntityDefinition via `consistencyTier`/a dedicated `sourcingMode` flag): writes append to the Event Backbone as the source of truth; independent projections (read models) are built by dedicated consumers — directly the Nubank/CQRS-bank pattern, letting a product expose e.g. a customer-facing view and an accounting/audit view from one event stream without them becoming coupled.
- **Cross-scalability-unit operations** use compensating transactions rather than distributed locks/2PC (Nubank's pattern), keeping each unit's availability independent of the others.
- **Migration/cutover safety**: before routing production traffic to a newly sharded or resharded entity, the framework exhaustively replays real query patterns against the new routing in a shadow environment — directly Etsy's pre-cutover compatibility-testing pattern, wired into the Testing & Simulation Framework (§B.9).

## B.11 AI-Ops / Agent Interface — Detailed Design

- **Catalog generation**: on `metadata.changed` and on module manifest registration, the AI-Ops module regenerates an MCP-compatible tool list — one tool per (EntityDefinition × allowed operation) and one tool per module-declared action, plus one resource per `ViewDefinition` where `aiOpsExposed: true`.
- **Agent identity**: a distinct principal type (`agent-identity` in `PermissionPolicy`) — never impersonates a human principal; scoped per task, short-lived, and revocable independent of any human session.
- **Guardrails**: operations classified `high-risk` (schema mutation, bulk delete, permission-policy change) require either a `dryRun=true` preview response or an explicit human-approval token before the mutating call is accepted, enforced at the Entity/Data Engine and Metadata Engine layer — not something an individual module can bypass.
- **Audit**: every agent-originated call is tagged `actor.type = agent` in the audit log (§ architecture.md §7) and correlated to the originating task/session id, so agent activity is fully distinguishable and traceable from human activity in the same store.

## B.12 Tech Stack Summary Table

| Concern | Choice | Why |
|---|---|---|
| Primary OLTP + metadata store | PostgreSQL | JSONB support suits metadata-as-data; RLS supports permission enforcement (PostgREST precedent); mature, widely operable. |
| Event backbone | Kafka (or API-compatible) | Proven at the exact scale profile this design targets (LinkedIn's 500B+ events/day precedent). |
| Cache / rate-limit store | Redis (or compatible) | Sub-millisecond latency needed for gateway rate limiting (Riot Games precedent). |
| Time-series store | TimescaleDB, InfluxDB, or an industrial historian-class store | Required for IoT telemetry volume/query patterns (domain research). |
| Search index | Elasticsearch/OpenSearch | Faceted/full-text query beyond point lookups. |
| Object storage | S3-compatible | Files, digital-twin artifacts, lake/cold tier. |
| OLAP / analytics store | ClickHouse or a cloud warehouse (BigQuery/Snowflake-class) | Dashboard/report query performance over large aggregated datasets. |
| Container orchestration | Kubernetes | Industry-standard for the stated deploy/scale requirements. |
| IaC | Terraform (or OpenTofu) | Declarative, versionable infrastructure, matches GitOps promotion model. |
| CI/CD | GitOps (Argo CD–class) + a build system (e.g., Buildkite/GitHub Actions) | Matches Shopify's CI/CD-at-scale precedent (parallel builds, merge queue). |
| Observability | OpenTelemetry + Prometheus/Grafana + a log aggregator | Standard, vendor-neutral instrumentation across all modules. |
| AI-Ops protocol | MCP (Model Context Protocol) | Purpose-built standard for agent tool/resource discovery (research §7). |
| Policy engine | OPA-class (Rego) or an equivalent embedded policy evaluator | Generic evaluation of `PermissionPolicy` condition expressions. |

## B.13 Deployment Topology & CI/CD

- **Environments**: Dev, Staging, Production — each capable of hosting one or more scalability units.
- **Promotion path**: metadata changes, module manifests, and IaC all flow through the same Git-based pipeline: PR → automated schema/contract validation (using the Testing & Simulation Framework's mock generation to validate against dependents) → merge → GitOps sync to the target environment.
- **New module rollout**: register manifest in a target environment → Registry validates extension-point graph (§B.2) → module deployed as an independent Kubernetes workload → health check gates traffic admission at the Gateway.
- **New tenant/scalability-unit provisioning**: a single declarative request to the Identity & Permission Engine triggers: data-plane resource creation (IaC), base metadata seeding, event-topic provisioning, and Gateway route activation — fully automatable and AI-ops-operable per §B.11.
- **Rollback**: because modules are independently versioned and stateless, a bad deploy rolls back one module without a full-platform redeploy (directly addressing the MTTR success metric in `requirements.md` §10).

## B.14 Scalability Detail

- **Horizontal autoscaling** per module based on request/queue-depth metrics (not fixed replica counts).
- **Sharding**: entity data is routed to its scalability unit's store via `EntityDefinition.tenancyScope`; cross-unit joins are avoided by design (bridge-record pattern, `architecture.md` §5) rather than solved with distributed queries.
- **Read scaling**: read replicas + cache tier for `consistencyTier: eventual | cached` entities; strong-consistency entities read from primary only, opt-in per entity so the cost is paid only where needed.
- **Elastic test/ETL compute**: the Testing & Simulation Framework and ETL pipelines run on autoscaled, spot/preemptible-eligible worker pools (Yelp FleetMiser precedent) since both are inherently batchable, bursty workloads.

## B.15 Observability & SLOs

- Every module exports OpenTelemetry traces/metrics/logs by default (a Module Contract requirement, not opt-in).
- Core platform SLOs: Gateway p99 latency, Event Backbone consumer lag, Metadata Engine mutation-to-propagation latency, Entity/Data Engine query p99 — each owned by the module, rolled up to a platform-wide dashboard.
- Alerting reuses the Notification & Communication Engine (§B.7) for on-call paging, rather than a separate alerting stack.

## B.16 Illustrative Metadata Instance — Asset Management (non-normative)

```json
// Example ONLY — not part of the framework's schema. Created by a product team at runtime.
{
  "entityDefinition": { "name": "assetmgmt.Asset", "storageHint": "oltp", "tenancyScope": "scalability-unit" },
  "fields": [
    { "name": "serialNumber", "type": "string", "required": true },
    { "name": "assetClass", "type": "enum", "enumValues": ["pump", "turbine", "vehicle", "sensor-node"] },
    { "name": "installDate", "type": "date" },
    { "name": "oemId", "type": "reference", "referenceEntityId": "assetmgmt.OEM" }
  ],
  "workflow": {
    "name": "AssetLifecycle",
    "states": ["commissioning", "active", "under-maintenance", "decommissioned"],
    "transitions": [
      { "from": "commissioning", "to": "active", "trigger": "manual" },
      { "from": "active", "to": "under-maintenance", "trigger": "event", "eventTopic": "workorder.opened" },
      { "from": "under-maintenance", "to": "active", "trigger": "event", "eventTopic": "workorder.closed" }
    ]
  }
}
```

This is the complete pattern for onboarding *any* vertical — the framework's involvement ends at interpreting these records generically.

## B.17 Non-Functional Acceptance Criteria

| Requirement | Acceptance criterion |
|---|---|
| Zero-downtime additive schema change | Adding a field/entity via the Metadata Engine requires no service restart and is queryable within one propagation cycle. |
| Module independence | Any single module can be deployed, scaled, or rolled back without redeploying another module. |
| AI-ops coverage | 100% of registered modules produce a valid MCP tool/resource listing with no module-specific code. |
| Tenant isolation | Cross-tenant data access attempts are denied by default at the Entity/Data Engine, verified by automated test-double-based penetration tests (§B.9). |
| Offline sync correctness | The fuzz-testing harness (§B.8) runs continuously with zero unresolved merge-divergence regressions before a Sync Engine release. |

## B.18 Traceability Appendix — Case Study → Module Mapping

| Case study (in `Architecture/`) | Module(s) informed |
|---|---|
| Architecture of API Gateway at Uber | API Gateway / Edge |
| Architecture of API Gateway at Tinder | API Gateway / Edge |
| API Platform at Riot Games | API Gateway / Edge |
| Basic Architecture of Slack | Identity & Permission Engine (multi-tenancy) |
| Architecture of Nubank | Transactional Integrity & Idempotency; Identity & Permission Engine |
| Core Banking System at Margo Bank | Transactional Integrity & Idempotency (CQRS/ES) |
| Bank Backend at Monzo | Event Backbone |
| Trading Platform for Scale at Wealthsimple | Event Backbone |
| Ads Pacing Service at Twitter | Event Backbone |
| Back-end at LinkedIn | Event Backbone; Module Registry (service decomposition precedent) |
| Settings Platform at LinkedIn | Metadata & Schema Engine (versioning lifecycle) |
| Real-time Presence Platform at LinkedIn | (future) Presence sub-module of Identity/Event modules |
| Back-end at Flickr | Entity/Data Engine (multi-tenant refactor precedent) |
| Nearline System at Glassdoor | ETL / Data Integration & Analytics |
| Real-time User Action Counting System at Pinterest | ETL / Data Integration & Analytics |
| Architecture of Following Feed at Pinterest | Entity/Data Engine (access-pattern-matched storage) |
| Media Database at Netflix | Entity/Data Engine (tiered consistency) |
| Member Transaction History Architecture at Walmart | Entity/Data Engine (polyglot persistence) |
| Sync Engine at Dropbox | Sync / Offline Engine |
| Kabootar at Swiggy | Notification & Communication Engine |
| Rapid Event Notification System at Netflix | Notification & Communication Engine |
| Building Services at Airbnb | Testing & Simulation Framework; Module Contract (IDL-first) |
| API Specification Workflow at WeWork | Module Contract (OpenAPI-first) |
| Seagull at Yelp | Testing & Simulation Framework |
| Phoenix at Tinder | Testing & Simulation Framework |
| Simone at Netflix | Testing & Simulation Framework (production simulation) |
| Hexagonal Architecture at Netflix | Testing & Simulation Framework (testability constraint); Module Contract |
| Avoiding Double Payments at Airbnb | Transactional Integrity & Idempotency |
| Scaling Payments at Etsy | Transactional Integrity & Idempotency (sharding + cutover testing) |
| Handles Millions of Transactions at Paytm | Transactional Integrity & Idempotency |
| Billing and Payment Platform at Grammarly | Transactional Integrity & Idempotency (outbox pattern) |
| Infrastructure at Zendesk | Deployment Architecture (org/foundation-team precedent) |
| Cloud Infrastructure at Grubhub | Deployment Architecture (IaC/container orchestration) |
| Lightweight Distributed Architecture at eBay | Testing & Simulation / CI pipeline (DAG-based parallelism) |
| Tech Stack case studies (Medium, Shopify, Addepar, TransferWise) | Tech Stack Summary (B.12) rationale |

*(All 47 case studies contributed; the above lists the direct 1:1 mappings. See `architecture.md` §4 for the per-module lineage narrative.)*

---
*End of technical design specification.*
