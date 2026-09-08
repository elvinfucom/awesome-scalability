# Business Requirements Document
## A Metadata-Driven, Modular Platform Framework

**Derived from:** pattern analysis of the 47 real-world scalability case studies in [`Architecture/`](../Architecture), cross-referenced against production metadata-driven platforms (Frappe/ERPNext, ServiceNow, Salesforce Platform, Backstage.io, Directus, Strapi, Hasura, PostgREST) and against Enterprise Asset Management / Industrial IoT / Field Service domain practice (ISO 55000, ISO 14224, MIMOSA, IBM Maximo, SAP EAM, PTC ThingWorx/Servigistics, ServiceMax).

**Status:** Draft v1
**Owner:** Platform Engineering (author: efu@ptc.com)

---

## 1. Executive Summary

Every case study in this repository's `Architecture/` folder tells some version of the same story: a company builds a product on a monolith or a single hard-coded schema, succeeds, and then hits a wall where the schema, the service boundary, or the operational model can no longer flex to the next order of magnitude of scale, feature diversity, or organizational size (LinkedIn's "Leo" monolith, Twitter's AdServer funnel, Shopify's shared-Redis single point of failure, Slack's per-workspace sharding assumption breaking under shared channels). The companies that recovered did so by pulling a small number of recurring moves out of their monolith and turning them into **generic, metadata-described, independently pluggable capabilities** — an API gateway driven by config instead of code (Uber, Tinder, Riot Games), a settings/feature model defined by schema metadata instead of hard-coded fields (LinkedIn Settings Platform), a service contract defined by an IDL instead of tribal knowledge (Airbnb Thrift, WeWork OpenAPI), a scalability unit that can be stamped out per tenant instead of one giant shared database (Nubank, Shopify pods, Slack shared-channel bridge).

This document defines the business requirements for a **platform framework** that generalizes those recurring moves into a reusable core, so that any product team — starting with an illustrative **Asset Management** vertical (manufacturing → product → OEM → installation → field service → maintenance, plus IoT data sync/ETL, analytics, dashboards, reporting, AI, and notifications) — can build a domain product on top of the framework by *defining metadata*, not by *forking or hard-coding* a new backend.

## 2. Problem Statement

1. **Duplicated invention.** Every product team that needs an API gateway, a notification system, a settings/config model, a workflow engine, or a multi-tenant sharding scheme re-invents one, at the cost the case studies document directly (LinkedIn: 5 weeks average lead time to add one new setting; Zendesk: org redesign forced by ad hoc infra ownership; WeWork: duplicated Postman/mock/test artifacts per API team).
2. **Schema rigidity.** Hard-coded, product-specific schemas make every new business requirement a code change and a migration, which does not fit domains like Asset Management where the entity model (asset classes, failure codes, inspection checklists) legitimately differs per customer, per industry, and per regulatory regime — a pattern the domain research confirms is why EAM/APM vendors (Maximo, SAP, ServiceMax) all converge on configurable objects/fields as a baseline requirement.
3. **AI operability is bolted on, not designed in.** Most legacy platforms expose their capabilities through hand-written, inconsistent APIs that an AI agent cannot safely discover or operate without bespoke integration work per system.
4. **Scale discontinuities.** The case studies show that systems built without an explicit scalability/tenancy strategy hit a wall precisely at a growth inflection (Shopify's "Redismageddon," Etsy's 40B-row unsharded payments tables, Yelp's 2-day test suite) and require a costly, high-risk re-architecture rather than a planned capacity increase.

## 3. Vision

> A single platform core — a metadata engine, a small set of generic runtime services (identity, data, workflow, events, gateway, notifications, sync, analytics, AI-ops), and a plugin/module contract — on which any business product is built entirely by **describing** its entities, workflows, permissions, and views as metadata, never by hard-coding them into the core. The core never knows what an "Asset" or a "Work Order" is; a product module does.

## 4. Goals

| # | Goal |
|---|---|
| G1 | New business entities, fields, relationships, workflows, and permissions can be added by **configuration/metadata**, with no core code change and no downtime. |
| G2 | Every capability (gateway, events, workflow, notifications, sync, ETL, analytics, testing/simulation, AI-ops) is a **module** with a defined contract, independently deployable and independently scalable. |
| G3 | The platform is **AI-operations-friendly**: any module's metadata and capabilities are automatically discoverable and safely operable by an AI agent (schema introspection doubles as an agent tool catalog, per the MCP pattern). |
| G4 | The platform supports **multi-tenant, horizontally scalable** deployment from day one, using a scalability-unit/sharding model proven in the case studies (Nubank, Shopify, Slack). |
| G5 | A reference vertical — **Asset Management** — can be built entirely as metadata + product-specific module code on top of the framework, validating that the framework holds no hidden assumptions about any one domain. |
| G6 | The platform is **easy to deploy** (containerized, IaC, one-command bootstrap of a new module or tenant) and **easy to scale** (stateless services, horizontal autoscaling, no shared mutable state outside the designated data-plane stores). |

### Non-Goals

- The framework will **not** ship a pre-built Asset Management data model (no hard-coded `Asset`, `WorkOrder`, etc. tables) — that belongs to the product layer, defined via the framework's metadata engine.
- The framework will **not** mandate a single business workflow engine's process definitions be embedded in the core; the core provides a generic workflow *runtime* that interprets externally supplied process metadata.
- The framework is not a low-code UI builder in v1 — UI is out of scope beyond the metadata needed to drive one (forms/views schema), though this should not be precluded later (see Architecture doc §11).

## 5. Stakeholders & Personas

| Persona | Needs from the platform |
|---|---|
| **Platform engineer** | A stable, well-documented core (metadata engine, module contract, event backbone) they maintain once for all products. |
| **Product engineer** (e.g., Asset Management team) | Ability to model their domain via metadata/config and ship product-specific module code without touching the core. |
| **Business/domain expert** (e.g., reliability engineer, service ops manager) | Ability to configure entities, workflows, and dashboards without engineering involvement, once tooling matures. |
| **Field technician / end user** | A fast, offline-capable mobile experience regardless of which product module they're using. |
| **Data/AI engineer** | A consistent way to ingest, transform, and expose data for analytics and ML across all product modules, without per-product bespoke pipelines. |
| **AI agent (automated operator)** | A discoverable, schema-driven interface (tool catalog) to introspect and safely operate the system — provisioning entities, querying data, running workflows, diagnosing incidents — under least-privilege, auditable guardrails. |
| **SRE / operations** | Uniform observability, deployment, and scaling model across all modules and tenants. |
| **Security/compliance officer** | Centralized, metadata-driven permission and audit model that applies uniformly regardless of which product module is in use. |

## 6. Business Value Drivers

1. **Time-to-market** — a new product vertical (Asset Management, or a future CRM/ITSM-style product) is built by writing metadata + thin product logic, not a new backend — directly mirroring how ERPNext ships as "just another app" on Frappe, or how ServiceMax ships as a managed package on Salesforce.
2. **Reduced total cost of ownership** — one gateway, one event backbone, one notification engine, one workflow engine, one testing/simulation framework serve every product, instead of N reinventions (the LinkedIn Settings Platform case study alone reports collapsing a 5-week per-setting lead time into a self-service metadata operation).
3. **Consistent security & compliance posture** — a single permission/ACL metadata model (per the ServiceNow ACL / Salesforce permission-set / Postgres RLS pattern) is audited once, not once per product.
4. **AI-native leverage** — because schema is metadata, the same metadata that renders a UI or generates an API can generate an agent tool catalog "for free" (per the MCP research), so AI copilots/agents (e.g., a technician-facing copilot in Asset Management, or a platform-ops agent) don't require bespoke integration per product.
5. **Proven scalability patterns baked in** — sharding/scalability-unit design, event-sourced ledgers, idempotency frameworks, and offline-sync patterns are available to every product from day one instead of being learned the hard way (as every case study company did).

## 7. Functional Requirements

Grouped by capability domain, each traced back to the case studies and prior-art research that motivate it.

### FR-1 Metadata & Schema Engine
- Define entities, fields, relationships, views, and validation rules as versioned metadata records (not code), in the style of Frappe DocTypes / ServiceNow dictionary tables / Salesforce custom objects / Directus collections.
- Support runtime schema evolution (add/change a field or entity) without service downtime.
- Support metadata export/import (as JSON/YAML) for versioning in source control and promotion across environments (mirrors Salesforce Metadata API, Hasura declarative metadata).

### FR-2 Modular Capability Runtime
- Every capability (see FR-3 through FR-11) is a **module** registered against a defined module contract (manifest + lifecycle hooks + extension points), addable/removable without modifying core code — mirrors Backstage's `createBackendModule`, Frappe's installable apps, ServiceNow scoped applications.
- Modules can extend other modules' behavior only through declared extension points, never by patching core source.

### FR-3 API Gateway / Edge Layer
- Single, protocol-agnostic entry point for all client and inter-module traffic (REST/GraphQL/gRPC), with centrally enforced authentication, rate limiting, and routing — generalized from Uber's Zanzibar gateway, Tinder's TAG, and Riot Games' Zuul-based gateway.
- Gateway routing/policy is metadata-driven (config, not per-endpoint code).

### FR-4 Identity, Permissions & Multi-Tenancy
- Tenant/workspace isolation with a configurable **scalability unit** model (a tenant's data and, optionally, compute can be isolated into its own shard/pod) — generalized from Nubank's scalability units, Shopify's pods, Slack's per-workspace shard-of-origin model.
- Role/attribute-based permission policies defined as metadata, evaluated generically by the runtime (mirrors ServiceNow ACLs, Salesforce permission sets, Postgres RLS).

### FR-5 Event Backbone & Workflow Engine
- Durable, replayable event log as the system's async backbone (generalized from LinkedIn's Kafka-as-universal-pipeline, Monzo's Kafka payments backbone).
- Workflow/process definitions supplied as external metadata (state machine or BPMN-like), executed generically by a workflow runtime (generalized from LinkedIn Settings Platform's DRAFT/ACTIVE/DEPRECATED lifecycle and from Camunda/Temporal patterns).

### FR-6 Notification & Communication Engine
- Multi-channel (push, email, SMS, in-app), multi-trigger (event-based and scheduled) notification delivery with pluggable integration adapters — generalized from Swiggy's Kabootar (integration/core/delivery layering) and Netflix's RENO (priority-queued hybrid push-pull with per-device fan-out).

### FR-7 Data Sync & Offline Support
- Conflict-aware sync engine for intermittently connected clients (field-technician mobile use is the primary driver) — generalized from Dropbox's Nucleus three-tree (Remote/Local/Synced) sync model.

### FR-8 Data Integration / ETL & Analytics
- Batch and streaming ingestion pipelines with a lakehouse-style hot/warm/cold tiering strategy, feeding both real-time alerting and periodic reporting — informed directly by the Asset Management domain research (IoT telemetry ingestion via MQTT/OPC-UA, time-series storage, lambda-architecture batch+stream split).
- Self-service dashboard/reporting layer over the same metadata model that defines the underlying entities (so a new entity is reportable without custom BI work), generalized from Walmart's polyglot-persistence-matched-to-access-pattern approach and Netflix's tiered-consistency Media DB.

### FR-9 Testing & Production Simulation Framework
- Auto-generated test doubles (mocks/stubs) directly from entity/API metadata — generalized from Airbnb's schema-based testing infrastructure.
- Massively parallel test execution — generalized from Yelp's Seagull (Mesos/Docker bin-packing, 2 days → 30 minutes).
- Narrowly-scoped, production-safe simulation triggers for validating edge-case behavior without customer impact — generalized from Netflix's Simone (Trigger/Variant model).

### FR-10 Transactional Integrity & Idempotency
- A generic idempotency-key framework for exactly-once side-effecting operations — generalized from Airbnb's idempotency framework (sharded by key) and Paytm's Paxos-based consensus/commit model.
- Optional event-sourcing/CQRS mode for domains needing an immutable ledger and multiple read projections — generalized from Nubank and the CQRS/ES bank case study, directly relevant to Asset Management's audit/compliance requirements (ISO 55000).

### FR-11 AI-Ops Interface
- Every module's metadata and operations are exposed as a structured, versioned tool/resource catalog (MCP-compatible) so an AI agent can discover and safely operate the system without bespoke per-module integration.
- Agent actions are scoped to least-privilege identities reusing the same permission metadata as human roles, are fully audit-logged and distinguishable from human actions, and support dry-run/human-in-the-loop approval gates for high-risk operations (deletes, schema migrations, permission changes) — per current agentic-architecture best practice.

## 8. Illustrative Example Domain: Asset Management

To validate the framework holds no hidden assumptions, the following Asset Management capabilities should be expressible **entirely as metadata + thin product code** on top of the framework, with no core changes:

- **Entities** (defined via FR-1, not hard-coded): Asset, Equipment/Functional Location, Product/Item Master, BOM, Site/Location Hierarchy, OEM/Supplier, Customer/Account, Technician, Service Contract/SLA, Warranty, Work Order, Job Plan/PM Schedule, Failure Code, Meter/Reading, Sensor/Telemetry Point, Digital Twin, Spare Part/Inventory Item.
- **Workflows** (via FR-5): commissioning/installation, preventive/predictive/condition-based maintenance triggers, work-order lifecycle (create → approve → schedule → dispatch → execute → close), warranty claims, decommissioning.
- **Data integration** (via FR-8): IoT telemetry ingestion (MQTT/OPC-UA) into time-series storage, ERP/CRM master-data sync, digital-twin contextualization.
- **Analytics** (via FR-8): asset uptime/availability, MTBF/MTTR, OEE, SLA compliance, technician utilization, TCO, PM compliance rate, asset health score.
- **AI/ML** (via FR-11 + FR-8): predictive maintenance, anomaly detection, remaining-useful-life estimation, technician copilots, parts/service recommendations, AI-optimized dispatch scheduling.
- **Notifications** (via FR-6): failure/fault alerts, SLA breach escalation, PM due reminders, warranty expiration, parts stock-out, dispatch push notifications, with alert correlation/deduplication to avoid alarm fatigue.
- **Non-functional fit**: multi-tenant SaaS deployment, offline-first field-technician mobile, industrial-IoT-scale telemetry volume, ISO 55000/14224-aligned audit trail.

This example is illustrative only — the framework must remain equally capable of hosting an unrelated vertical without change.

## 9. Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Scalability** | Horizontal scaling of every module; tenant-level scalability units (shard-per-tenant or shard-per-tenant-group) supporting industrial-IoT-scale telemetry volumes (per domain research: tens of thousands of assets, orders of magnitude more sensor tags, sub-second sampling). |
| **Availability** | No single point of failure in the data or event plane (learned directly from Shopify's Redismageddon and LinkedIn's original single-database Leo monolith); target 99.9%+ for core platform services. |
| **Multi-tenancy** | Configurable isolation level per tenant (shared schema with row-level isolation, or dedicated scalability unit) — configurable, not hard-coded, per tenant risk/scale profile. |
| **Security & Compliance** | Centralized metadata-driven RBAC/ABAC; full audit trail of human and AI-agent actions; encryption in transit and at rest; alignment with ISO 55000 (asset-management governance) and IEC 62443 (OT/IoT security) for the Asset Management example domain. |
| **Extensibility/Configurability** | New entity, field, workflow, permission, or dashboard addable via metadata with no downtime and no core code change. |
| **AI-Ops Friendliness** | All modules discoverable/operable by AI agents via a structured, versioned tool catalog generated from the same metadata that drives the human-facing schema. |
| **Deployability** | Containerized modules, one-command environment bootstrap, GitOps-based promotion across environments. |
| **Observability** | Uniform metrics/tracing/logging across all modules and tenants, with SLOs defined per module. |
| **Offline Support** | Field/mobile clients must operate against cached data and sync conflict-free on reconnect (Asset Management field-technician use case). |
| **Data Retention** | Configurable hot/warm/cold tiering to support both real-time operations and multi-year archival (industrial IoT/compliance need). |

## 10. Success Metrics

| Metric | Target signal |
|---|---|
| Time to add a new entity/field/workflow | Minutes (metadata change), not a sprint (code change + migration + deploy). |
| Time to onboard a new product vertical | Weeks (metadata + thin module code), not a new backend build. |
| % of platform capability accessible to AI agents without bespoke integration | Approaching 100% via auto-generated tool catalog. |
| Cross-product reuse of core modules (gateway, notifications, workflow, sync, testing) | 100% — no product forks the core. |
| Mean time to recover from a module failure | Minutes, via independent module deploy/rollback, not a full-platform redeploy. |

## 11. Assumptions & Constraints

- The platform targets container-orchestrated cloud/on-prem deployment (Kubernetes-class environments); it does not target embedded/edge-only deployment for the core (edge gateways for IoT ingestion are a module concern, not a core concern).
- Initial reference implementation targets one illustrative vertical (Asset Management) but must not encode Asset-Management-specific assumptions into the core — this is a hard architectural constraint, not a soft preference.
- The organization has (or will build) platform-engineering capacity separate from product engineering, mirroring the case studies' consistent pattern of a dedicated platform/foundation team (Zendesk Foundation team is the direct precedent).

## 12. Out of Scope (v1)

- A finished, shippable Asset Management product (the example domain is used to validate the framework's generality, not to deliver a commercial EAM product).
- A visual low-code UI builder (metadata schema for views is in scope; a drag-and-drop builder on top of it is a later-phase concern).
- Vendor-specific integrations (specific ERP/CRM connectors) beyond a generic integration/ETL module contract.

---
*See [`architecture.md`](./architecture.md) for the system architecture and [`technical-design-spec.md`](./technical-design-spec.md) for the detailed technical design.*
