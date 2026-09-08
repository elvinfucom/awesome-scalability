# Back-end at LinkedIn

**Source:** https://engineering.linkedin.com/architecture/brief-history-scaling-linkedin

## Problem
LinkedIn launched in 2003 as a single monolithic application called 'Leo' that hosted all web servlets, business logic, and connections to a handful of databases. As membership grew, this monolith and its databases became a bottleneck, unable to scale independently for different features or handle rapidly growing read/write and data-pipeline demands.

## Solution
LinkedIn incrementally decomposed the monolith into services, starting with 'Cloud' for member-to-member connections, then extracting further microservices for search, profile, communications, and groups, growing to over 750 services in later years. To handle growing data movement between systems, they built Kafka as a universal, commit-log-based pipeline for near-real-time data access, which by the time of writing was handling more than 500 billion events per day, letting each piece of infrastructure scale independently.