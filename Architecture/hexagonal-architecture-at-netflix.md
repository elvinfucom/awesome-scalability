# Hexagonal Architecture at Netflix

**Source:** https://netflixtechblog.com/ready-for-changes-with-hexagonal-architecture-b315ec967749

> **Note:** Direct fetch blocked (403); summary compiled from search-result snippets/excerpts of the original post.

## Problem
Netflix's Studio Engineering built 30+ tightly-integrated applications supporting the entire creative workflow for producing original content. Because these systems were tightly coupled from the start, the team anticipated core data sources and dependencies would inevitably need to change, and needed business logic to survive such changes without costly rewrites.

## Solution
Netflix adopted Hexagonal Architecture (ports and adapters): core business logic lives in framework-agnostic 'interactors' that know nothing about transport or persistence, while transport layers and data sources are adapters conforming to interfaces defined by the core. This paid off sooner than expected: when a read constraint forced a switch to a newer microservice via GraphQL, the team swapped the underlying data source with minimal disruption to business logic.