# Avoiding Double Payments in a Distributed Payments System at Airbnb

**Source:** https://medium.com/airbnb-engineering/avoiding-double-payments-in-a-distributed-payments-system-2981f6b070bb

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from secondary sources summarizing the original post.

## Problem
Airbnb's payments system is a distributed microservice architecture with many network calls between services, creating a high risk of duplicate transaction requests (e.g. from client retries or network failures) resulting in guests being charged multiple times for a single booking.

## Solution
Airbnb built a generic idempotency framework where each transaction request carries a unique idempotency key; incoming requests are checked against a centralized log of previously processed keys so duplicates are safely short-circuited. Transactions are recorded atomically, and retry logic layered on top of idempotency keys lets clients safely retry failed calls. As the single-master idempotency-key database became a bottleneck, Airbnb sharded it by idempotency key.