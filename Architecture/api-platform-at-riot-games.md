# API Platform at Riot Games

**Source:** https://engineering.riotgames.com/news/riot-games-api-deep-dive

## Problem
Riot Games needed to expose a public API to third-party developers at massive global scale, while protecting backend services from abuse/scraping, enforcing per-developer rate limits without introducing latency bottlenecks, and serving traffic reliably across multiple geographic regions.

## Solution
Riot built its API gateway on Netflix's open-source stack (Zuul, Archaius, Ribbon, Hystrix, Eureka), adding custom filters for authentication, rate limiting, and routing. An Edge Service Rate Limiter implements a leaky-bucket algorithm backed by Redis using Lua scripts, keeping response times averaging under 2ms; a decoupled metrics service posts usage data asynchronously so it never blocks the request path. The system runs across three AWS regions (Tokyo, Ireland, NorCal) behind ELBs.