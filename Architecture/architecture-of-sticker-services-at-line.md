# Architecture of Sticker Services at LINE

**Source:** https://www.slideshare.net/linecorp/architecture-sustaining-line-sticker-services

> **Note:** SlideShare page returned only a client-side loading message; summary compiled from search-result snippets describing the deck's contents.

## Problem
LINE's Sticker Shop started in 2015 as a monolithic Java application. As sticker usage and shop traffic grew massively, the monolith needed to evolve to handle very high request volume, remain resilient to faults, and support safe rollout of new features without downtime.

## Solution
By 2018, LINE had decomposed the sticker shop into roughly 30 microservices capable of handling about 70,000 requests per second at peak, communicating primarily via Thrift over HTTP/2. Key components include a Configuration Repository Service (Central Dogma) for dynamic config, Elasticsearch and Redis for search/caching, and MySQL for persistent storage, with an emphasis on stable/gradual feature rollout.