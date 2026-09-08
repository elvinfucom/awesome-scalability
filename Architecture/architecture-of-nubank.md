# Architecture of Nubank

**Source:** https://www.infoq.com/presentations/nubank-architecture

## Problem
Nubank needed to build a modern digital banking platform from scratch handling complex financial domain logic (credit, authorization, billing, payments, double-entry accounting) across millions of customers, avoiding the rigidity of traditional mainframe-based banking systems. As transaction volume grew into the hundreds of millions per month, single-database write throughput and tight coupling between financial operations became major constraints.

## Solution
Nubank built a Clojure-based, event-sourced architecture using Datomic and Kafka as a central nervous system for asynchronous messaging across 100+ microservices deployed up to 20 times a day. A custom purchase authorizer runs on isolated infrastructure with Hardware Security Modules, syncing state via Kafka logs with periodic snapshots to S3. Instead of traditional sharding, Nubank replicates entire infrastructure stacks into 'scalability units' mapped to customer partitions, with compensating transactions handling cross-shard transfers, and dead-letter queues plus circuit breakers for resilience.