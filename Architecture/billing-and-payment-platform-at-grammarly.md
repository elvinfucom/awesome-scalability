# Billing and Payment Platform at Grammarly

**Source:** https://www.grammarly.com/blog/engineering/billing-and-payments-platform/

## Problem
Grammarly needed to manage billing and payments globally for roughly 30 million daily users across multiple product tiers, supporting many currencies and payment methods while complying with varied local financial regulations, through a platform other internal teams could build on via self-service APIs.

## Solution
Grammarly built a billing and payments platform as decoupled but tightly integrated services covering packaging, pricing, invoicing, and fraud detection. For scalability, they use concurrent processing with independently scalable services and optimistic locking; for reliability, they rely on the transactional outbox pattern, circuit breakers, dead letter queues, and idempotent message handling, with event-driven interservice communication and distributed tracing for visibility into complex asynchronous payment flows.