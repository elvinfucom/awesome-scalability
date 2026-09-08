# Simone: Distributed Simulation Service at Netflix

**Source:** https://medium.com/netflix-techblog/https-medium-com-netflix-techblog-simone-a-distributed-simulation-service-b2c85131ca1b

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-indexed excerpts and a related devblogs recap.

## Problem
Netflix needed a safe way to simulate specific edge-case conditions in production â€” such as a member exhausting the maximum number of simultaneous streams â€” without writing one-off test hooks into every service or risking that a test scenario accidentally affects real customer traffic.

## Solution
Netflix built Simone, a generic distributed simulation service that lets service owners configure and run simulations across arbitrary domains in production. Simone's core abstractions are Triggers, defining the precise condition under which simulated behavior activates (e.g. a specific device or account), and Variants, the simulated behavior applied only when a Trigger matches, minimizing the 'blast radius' of any simulation.