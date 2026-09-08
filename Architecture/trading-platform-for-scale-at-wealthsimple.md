# Trading Platform for Scale at Wealthsimple

**Source:** https://medium.com/@Wealthsimple/engineering-at-wealthsimple-reinventing-our-trading-platform-for-scale-17e332241b6c

> **Note:** Direct fetch blocked (403); summary compiled from search-engine snippets.

## Problem
Wealthsimple's original trading process started as a manual spreadsheet workflow that could not scale as the client base grew, and each new order-of-magnitude of growth (Canadian growth, US market entry, new offerings) broke the previous approach, forcing a rethink of the platform.

## Solution
Wealthsimple rebuilt trading as a distributed, massively parallel microservice architecture, where each service owns a specific resource/behavior (e.g. a 'Portfolio Manager' service kicking off daily decision-making). Transaction data from multiple products is synced into a shared general ledger via Kafka, with many producer services publishing to a single topic, improving transparency and auditability of trades.