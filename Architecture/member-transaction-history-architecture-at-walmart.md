# Member Transaction History Architecture at Walmart

**Source:** https://medium.com/walmartlabs/member-transaction-history-architecture-8b6e34b87c21

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-engine snippets/cached descriptions.

## Problem
Walmart needed a single source of truth for omnichannel (club and online) order data that could serve as a canonical, reliable, low-latency read layer across many downstream consumers, each needing different access patterns: simple key-value lookups, filtered point queries, predefined aggregations, and dynamic field-level search.

## Solution
The team built Member Transaction History (MTH) as an omnichannel read layer ingesting order events from store devices and Kafka feeds, exposing a canonical order model in near real time. They evaluated multiple Azure data stores against each use-case category, choosing Azure Cosmos DB for low-latency queries (leveraging its change feed) while considering Azure Search and Azure SQL for aggregation/faceted search, arriving at a polyglot persistence architecture tailored to each query pattern.