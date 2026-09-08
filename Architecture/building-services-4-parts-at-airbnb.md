# Building Services (4 parts) at Airbnb

**Source:** https://medium.com/airbnb-engineering/building-services-at-airbnb-part-4-23c95e428064

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-result snippets and third-party mirrors describing this post and the related series.

## Problem
As Airbnb scaled from a monolith to many independent services, engineers needed a standardized way to define and enforce service contracts and best practices across teams. Testing was a major pain point: the API is the boundary between many services, so it became a hotspot for hand-built mock data, clients, and fake services duplicated across teams.

## Solution
Airbnb built its service platform around Thrift as the service Interface Definition Language (IDL), using the schema as a single source of truth to standardize service definitions across the company. Part 4 describes a Schema Based Testing Infrastructure that auto-generates test doubles (mock data, clients, and servers) directly from the Thrift schema, eliminating duplicated effort and giving teams consistent, low-maintenance testing support automatically.