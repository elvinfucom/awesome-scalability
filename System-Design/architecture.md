# Architecture Document
## A Metadata-Driven, Modular Platform Framework

**Companion to:** [`requirements.md`](./requirements.md) (business requirements) and [`technical-design-spec.md`](./technical-design-spec.md) (detailed technical design).

---

## 1. Architectural Vision & Principles

1. **Metadata is the interface, not code.** Every schema, permission, workflow, and view is a versioned metadata record interpreted generically at runtime by the core — never compiled into product-specific code. This is the single mechanism every mature reference platform studied (Frappe, ServiceNow, Salesforce, Directus, Strapi, Hasura, PostgREST) converges on independently.
2. **Strict core/product separation.** The core (framework) knows *how* to store an entity, run a workflow, route a request, and notify a user. It never knows *what* an "Asset" or "Work Order" is. That knowledge lives entirely in product-layer metadata + thin product code, exactly as ERPNext is "just another app" on Frappe, and ServiceMax is "just another managed package" on Salesforce.
3. **Modularity via declared contracts, not shared mutable code.** Capabilities are modules that register against a defined contract and extend one another only through declared extension points (Backstage's `createBackendModule` pattern), never by patching core source.
4. **Scalability units over one big database.** Tenants/workloads scale by replicating an isolatable unit (shard, pod, or scalability unit), not by vertically scaling one shared store — the pattern common to Nubank, Shopify, and Slack's shared-channel design.
5. **The same metadata that drives a UI drives an AI agent.** Schema introspection is the platform's native AI-ops interface (MCP tool/resource discovery is mechanically the same operation as a UI form renderer reading a DocType) — there is no separate "AI integration layer" to bolt on later.
6. **Everything is built to be tested and simulated safely in production.** Test doubles are generated from the same metadata as the real API (Airbnb's pattern); production behavior can be narrowly and safely simulated (Netflix Simone's pattern) without customer blast radius.

## 2. Reference Architectures Studied

| Studied system | What it validates in this design |
|---|---|
| **Frappe / ERPNext** | DocType metadata model → schema, API, UI, permissions all generated from one metadata record; apps as installable modules with zero core business schema. |
| **ServiceNow** | Self-describing dictionary tables (`sys_dictionary` describes itself); scoped applications as the modularity boundary; Business Rules/ACLs as data, not code. |
| **Salesforce Platform** | Universal data dictionary enabling true multi-tenancy on shared infrastructure; Metadata API as a serializable, version-controllable schema; packages as the plugin/distribution unit. |
| **Backstage.io** | App-shell-plus-plugins architecture; explicit "extension point" pattern for safe cross-plugin extension; entity-kind catalog as a metadata-driven "system of systems" model. |
| **Directus / Strapi / Hasura / PostgREST** | Progressively leaner implementations of "introspect or describe a schema, generate the API from it" — validates that the metadata engine + dynamic API layer is implementable at very different levels of framework weight. |
| **Temporal / Camunda / Airflow / n8n** | Workflow-as-declarative-metadata (BPMN/JSON) vs. workflow-as-code, and the corresponding plugin points (workers, connectors, operators, nodes) for adding new task types without touching the engine. |
| **MCP (Model Context Protocol)** | Tools/Resources/Prompts discovery model as the standard shape for exposing a system to an AI agent — directly reusable on top of this platform's metadata registry. |
| **47 internal case studies (`Architecture/`)** | Concrete, battle-tested patterns for each module below (cited per-module in §4). |

## 3. High-Level Architecture

```
                                   ┌─────────────────────────────────────────┐
                                   │              Product Layer               │
                                   │  (Asset Management, or any other vertical)│
                                   │   — metadata definitions + thin product   │
                                   │        code + product-specific UI         │
                                   └───────────────────┬───────────────────────┘
                                                        │  (metadata + module contract)
        ┌───────────────────────────────────────────────┼───────────────────────────────────────────────┐
        │                                         Platform Core                                          │
        │                                                                                                 │
        │   ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐                   │
        │   │  Metadata &   │  │   Entity/Data  │  │   Workflow &   │  │  Identity &    │                 │
        │   │ Schema Engine │  │     Engine     │  │ Process Engine │  │  Permission    │                 │
        │   │  (FR-1)       │  │   (FR-1/FR-8)  │  │    (FR-5)      │  │  Engine (FR-4) │                 │
        │   └───────┬───────┘  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘                 │
        │           │                   │                   │                   │                          │
        │   ┌───────┴───────────────────┴───────────────────┴───────────────────┴───────┐                 │
        │   │                    Event Backbone (durable, replayable log)  (FR-5)         │                │
        │   └───────┬───────────────────┬───────────────────┬───────────────────┬───────┘                 │
        │           │                   │                   │                   │                          │
        │   ┌───────┴──────┐   ┌────────┴───────┐  ┌────────┴───────┐  ┌────────┴───────┐                 │
        │   │  API Gateway │   │  Notification  │  │  Sync/Offline  │  │  ETL / Data    │                 │
        │   │  / Edge (FR-3)│   │  Engine (FR-6) │  │  Engine (FR-7) │  │  Integration   │                 │
        │   └──────────────┘   └────────────────┘  └────────────────┘  │  & Analytics    │                 │
        │                                                                │  (FR-8)         │                │
        │   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐ └────────────────┘                 │
        │   │ Testing &      │  │ Transactional  │  │  AI-Ops /      │                                      │
        │   │ Simulation     │  │ Integrity &    │  │  Agent         │                                      │
        │   │ Framework (FR-9)│  │ Idempotency    │  │  Interface     │                                      │
        │   └────────────────┘  │  (FR-10)       │  │  (FR-11)       │                                      │
        │                       └────────────────┘  └────────────────┘                                      │
        │                                                                                                 │
        │   ┌───────────────────────────────────────────────────────────────────────────────────────┐   │
        │   │                     Module Registry & Contract Runtime  (governs all modules above)      │   │
        │   └───────────────────────────────────────────────────────────────────────────────────────┘   │
        └─────────────────────────────────────────────────────────────────────────────────────────────┘
                                                        │
        ┌───────────────────────────────────────────────┴───────────────────────────────────────────────┐
        │                                        Data Plane                                              │
        │  Metadata store │ OLTP (per-tenant/shard) │ Event log │ Time-series store │ Lake/warehouse │    │
        │  Cache │ Search index │ Object storage                                                          │
        └─────────────────────────────────────────────────────────────────────────────────────────────┘
```

## 4. Core Platform Components

Each component is a module registered against the Module Registry & Contract Runtime (§6). Case-study lineage is cited per component.

### 4.1 Metadata & Schema Engine
Stores **EntityDefinition**, **FieldDefinition**, **RelationshipDefinition**, **PermissionPolicy**, **WorkflowDefinition**, and **ViewDefinition** records — the "schema of schemas." Nothing about *Asset* or *Work Order* lives here; only the generic constructs a product uses to describe them.
*Lineage:* Frappe DocType/DocField, ServiceNow `sys_dictionary`, Salesforce Metadata API, Directus `directus_fields`, LinkedIn's Settings Platform (metadata-defined settings with DRAFT/ACTIVE/DEPRECATED versioning — directly informs the versioning model here).

### 4.2 Entity/Data Engine
Given an EntityDefinition, dynamically exposes CRUD + query APIs (REST/GraphQL) without hand-written resolvers, and generates the underlying storage (table/collection) — the same mechanism Hasura, PostgREST, and Directus use to turn a schema into an API for "free."
*Lineage:* Netflix Media DB's tiered-consistency-per-consumer-class model (different products may need different consistency guarantees from the same entity engine); Walmart MTH's polyglot-persistence-by-access-pattern (point lookups vs. faceted search vs. aggregation route to different backing stores under one logical entity API).

### 4.3 Workflow & Process Engine
Executes externally supplied process definitions (state machines or BPMN-like graphs) generically; new task types are added via pluggable Job Workers/Connectors, never by modifying the engine.
*Lineage:* Camunda/Temporal/n8n plugin-worker pattern; LinkedIn Settings Platform's metadata lifecycle (DRAFT → ACTIVE → DEPRECATED) as a workflow-as-metadata example already proven at scale.

### 4.4 Identity & Permission Engine
Evaluates RBAC/ABAC policies stored as metadata against every request, uniformly across modules; supports the tenant scalability-unit model (§5).
*Lineage:* ServiceNow ACLs, Salesforce permission sets, Postgres RLS (as used by PostgREST); Nubank's isolated-infrastructure-per-scalability-unit for high-sensitivity operations (e.g., a payment authorizer).

### 4.5 Event Backbone
A durable, replayable, partitioned commit log is the platform's async spine; every module publishes/consumes through it rather than calling other modules directly, decoupling producers from consumers and enabling replay/audit.
*Lineage:* LinkedIn's Kafka backbone (500B+ events/day), Monzo's Kafka-backed payment/enrichment pipeline, Twitter's Ads Pacing feedback loop (spend counter → pacing recompute → serving), Wealthsimple's Kafka-synced general ledger.

### 4.6 API Gateway / Edge Layer
Single, protocol-agnostic (REST/GraphQL/gRPC/Thrift) ingress; auth, rate limiting, routing, and data-center/region affinity are all metadata/config-driven, not per-endpoint code.
*Lineage:* Uber's Zanzibar-based gateway (build-time-generated, statically typed, ~500K QPS after migrating 1,500+ APIs); Tinder's TAG (Spring Cloud Gateway, config-driven per-team gateway instances); Riot Games' Zuul-based gateway with a Redis/Lua leaky-bucket rate limiter averaging <2ms and a decoupled async metrics path so metering never blocks serving.

### 4.7 Notification & Communication Engine
Multi-channel (push/email/SMS/in-app), multi-trigger (event- and schedule-based) delivery, structured as an integration layer → core processing layer → delivery layer so the transport (queue/protocol) is swappable without touching business rules.
*Lineage:* Swiggy's Kabootar (8 microservices over RabbitMQ, Enterprise Integration Patterns); Netflix's RENO (priority-segmented SQS queues, hybrid push-pull with per-device fan-out isolating failures, ~150K events/sec at peak).

### 4.8 Sync / Offline Engine
Reconciles client and server state for intermittently connected clients using an explicit three-way model (remote / local / last-synced) and ID-based (not path-based) referencing so structural moves are O(1), plus deterministic, seed-reproducible fuzz testing of the sync algorithm itself.
*Lineage:* Dropbox's Nucleus rewrite (Remote/Local/Synced trees, CanopyCheck + Trinity fuzzers running tens of millions of nightly test runs) — directly the model needed for Asset Management's offline field-technician mobile use case.

### 4.9 ETL / Data Integration & Analytics Engine
Batch + streaming ingestion with hot/warm/cold tiering; entities defined in the Metadata Engine are automatically reportable (a new entity is dashboard-ready without bespoke BI work).
*Lineage:* Asset-management domain research (MQTT/OPC-UA telemetry → time-series store → lambda-architecture batch/stream split → lake/warehouse); Netflix Media DB and Walmart MTH (consumer-tiered consistency and access-pattern-matched storage, generalized here to the analytics tier).

### 4.10 Testing & Simulation Framework
Auto-generates test doubles (mock data/clients/servers) directly from EntityDefinition/API metadata; runs massively parallel test execution via bin-packed containerized workers; supports narrowly-scoped, production-safe behavioral simulation.
*Lineage:* Airbnb's schema-based testing infrastructure (auto-generated from the Thrift IDL); Yelp's Seagull (Mesos+Docker bin-packing across ~10K cores, cutting a 2-day suite to ~30 minutes, with FleetMiser autoscaling cutting cost ~80%); Tinder's Phoenix experimentation platform (decoupling feature/config rollout from app release cycles); Netflix's Simone (Trigger/Variant model scoping simulation blast radius to a single device/account) and Netflix's Hexagonal Architecture (core business logic isolated behind ports/adapters specifically so it stays cheaply testable as dependencies churn).

### 4.11 Transactional Integrity & Idempotency Module
Generic idempotency-key framework (sharded by key at scale) for exactly-once side effects; optional event-sourced/CQRS mode giving an immutable ledger with multiple independent read projections from one event log.
*Lineage:* Airbnb's idempotency framework; Etsy's Vitess sharding of a 40B+-row payments dataset plus exhaustive pre-cutover query compatibility testing; Paytm's Paxos-based single-leader consensus and LSM-tree write path; Nubank's Clojure/Datomic event-sourced core with compensating transactions across scalability units; the Memo/"Margo" Bank CQRS+event-sourcing model (multiple projections — client API vs. accounting/ledger API — from one event stream).

### 4.12 AI-Ops / Agent Interface
Auto-generates a versioned MCP-compatible tool/resource catalog directly from the Metadata Engine and each module's contract, so any module is agent-operable without bespoke integration; agent identities are scoped through the same Identity & Permission Engine as human roles, every action is audit-logged, and high-risk operations (schema migration, deletion, permission change) require dry-run or human-in-the-loop approval.
*Lineage:* MCP client-host-server/tool-discovery model; the general principle (validated across every reference platform in §2) that the metadata already driving the UI/API is the same metadata that can drive an agent tool catalog.

## 5. Multi-Tenancy & Scalability Strategy

- **Scalability unit** = the isolatable deployment/data unit (a shard, a pod, or a full stack replica) that can be stamped out per tenant, tenant group, or high-sensitivity workload.
  - *Shared-schema mode* (row-level tenant isolation via the Identity & Permission Engine) for low/medium-scale tenants — cheapest, fastest to onboard.
  - *Dedicated scalability unit* (own data store, optionally own compute) for large or highly regulated tenants — mirrors Nubank's full-stack replication per customer partition and Shopify's pod model (100+ isolated pods, zero platform-wide outages since adoption).
- **Single source of truth per originating shard**, with cross-shard references handled via a bridge/pointer record rather than data duplication — directly modeled on Slack's `shared_channels` bridge table design for shared channels across workspace shards.
- **Compensating transactions**, not distributed locks, for operations spanning scalability units (Nubank's pattern), keeping each unit independently available.
- **Edge/read-side caching** for read-heavy, globally distributed access (Flickr/Tripod's five-cache-plus-CDN pattern for media delivery generalizes to any read-heavy entity in the Entity/Data Engine).

## 6. Module Registry & Contract Runtime

- Every module ships a **manifest** (id, version, declared extension points it exposes, declared extension points it consumes, required core services) — the mechanism by which the core stays ignorant of module internals.
- Modules extend each other **only** through declared extension points (Backstage's `createBackendModule`/extension-point pattern), never by importing or patching another module's internals.
- The registry itself is introspectable, which is what allows §4.12's AI-ops catalog and §7's tenant provisioning to be generic operations rather than per-module special cases.
- Full contract shape is specified in `technical-design-spec.md` §B.2.

## 7. Security & Compliance Model

- **AuthN**: OIDC/OAuth2 for human users and services; short-lived, scoped credentials for AI agents (distinct identity class, never impersonating a human).
- **AuthZ**: single Identity & Permission Engine (§4.4) evaluated by every module; no module implements its own bespoke permission logic.
- **Audit**: every mutating action (human or agent) is appended to an immutable audit log correlated to the Event Backbone, satisfying ISO 55000-style governance/audit requirements out of the box for any product built on the platform (Asset Management's compliance/audit workflow is a direct consumer, not a special case).
- **Data protection**: encryption in transit and at rest by default at the data-plane layer; IEC 62443 considerations apply to any module bridging OT/IoT devices (relevant to Asset Management's telemetry ingestion, §4.9).

## 8. AI-Ops Architecture (detail)

1. Metadata Engine + Module Registry are introspected to auto-generate an MCP tool/resource catalog (one tool per entity operation, workflow trigger, or module-declared action).
2. Agent identities are provisioned through the same Identity & Permission Engine as human roles — least privilege by default, no standing broad-scope credentials.
3. Every agent call passes through the same audit/observability path as human-triggered calls (§7), tagged distinguishably as agent-originated.
4. High-risk operation classes (schema/metadata migration, bulk delete, permission change) are gated behind a dry-run mode and/or human-in-the-loop approval step, configurable per tenant risk profile.
5. This design requires no per-module "AI integration" work going forward — a new module is agent-operable the moment it registers its manifest (§6), because the catalog generation reads the manifest and the Metadata Engine, not module-specific code.

## 9. Deployment Architecture

- Each module is an independently deployable, stateless, containerized service; state lives only in the designated Data Plane stores (§10).
- Kubernetes-class orchestration; Helm/Kustomize-style templated manifests; GitOps promotion across environments; Terraform-class IaC for underlying cloud/data-plane resources.
- New tenant provisioning = a declarative operation against the Module Registry + Identity Engine (create scalability unit, apply base metadata, wire event topics) — not a bespoke deployment.
- Full deployment topology and CI/CD pipeline detail: `technical-design-spec.md` §B.9.

## 10. Data Plane (storage tiers, not schemas)

| Tier | Purpose | Notes |
|---|---|---|
| Metadata store | EntityDefinition/FieldDefinition/PermissionPolicy/WorkflowDefinition/ViewDefinition records | Source of truth for §4.1; must support fast reads with strong consistency. |
| OLTP store (per scalability unit) | Actual entity instance data, generated per EntityDefinition | Sharded/isolated per §5. |
| Event log | Durable, replayable, partitioned commit log | Backbone for §4.5, §4.9, §4.11. |
| Time-series store | High-volume telemetry (IoT/sensor data in the Asset Management example) | Separate from OLTP for volume/query-pattern reasons (domain research §3). |
| Lake/warehouse | Batch analytics, long-term archival, hot/warm/cold tiering | Feeds §4.9 dashboards/reporting. |
| Cache | Read-side acceleration | Supports §5's edge-caching strategy. |
| Search index | Full-text/faceted query over entity data | Complements the Entity/Data Engine's query API for access patterns point-lookup can't serve efficiently. |
| Object storage | Files, attachments, digital-twin artifacts | Referenced by entity metadata, not modeled as relational schema. |

No business-specific schema is defined at this layer — see `technical-design-spec.md` §B.1 for the metadata meta-model that lets a product (e.g., Asset Management) define its own entities against these tiers.

## 11. Key Architectural Decisions (ADR summary)

| Decision | Chosen approach | Rejected alternative | Rationale |
|---|---|---|---|
| Schema definition | Metadata-driven (runtime-interpreted) | Hard-coded per-product schema | Every reference platform studied converges here; case studies show hard-coded schema is the recurring scaling wall (LinkedIn Leo, Twitter AdServer). |
| Module boundary | Declared contract + extension points | Shared library / monolith | Backstage/ServiceNow scoped-app precedent; avoids core/product coupling that forced later rewrites in the case studies. |
| Tenancy | Configurable scalability unit (shared or dedicated) | One-size-fits-all shared DB or one-size-fits-all full isolation | Matches Nubank/Shopify's proven approach and avoids Shopify's pre-pod single-point-of-failure incident. |
| Async backbone | Durable replayable log (Kafka-class) | Direct synchronous service-to-service calls | LinkedIn/Monzo precedent; enables replay, audit, and decoupled scaling. |
| AI-ops | Auto-generated from existing metadata (MCP-compatible) | Bespoke API per agent integration | Avoids N-times integration cost; matches the "metadata already does this" insight from prior-art research. |
| Consistency model | Per-entity configurable consistency tier | One global consistency guarantee | Netflix Media DB precedent — different consumers legitimately need different guarantees from the same data. |

## 12. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Over-genericizing the metadata engine makes it slow/complex to use | Start with a minimal EntityDefinition/FieldDefinition/RelationshipDefinition surface (§B.1 of the technical spec); expand only against a validated product need (Asset Management). |
| Module contract becomes a bottleneck for innovation | Extension points are additive and versioned; modules can propose new extension points via the same manifest mechanism, reviewed like an API change. |
| Multi-tenancy misconfiguration leaks data across tenants | Identity & Permission Engine enforcement is mandatory at the Entity/Data Engine layer, not optional per-module; default-deny. |
| AI-agent misuse causes destructive changes | Dry-run + human-in-the-loop gates mandatory by default for high-risk operation classes; can only be relaxed per tenant policy, never globally. |
| Case-study patterns don't generalize cleanly | Validate against a second, unrelated vertical before declaring the core "done" (a concrete follow-up recommendation, not yet executed). |

---
*See [`technical-design-spec.md`](./technical-design-spec.md) for module contracts, the metadata meta-model, tech stack, and deployment/scaling detail.*
