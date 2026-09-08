# Scaling Payments (3 parts) at Etsy

**Source:** https://www.etsy.com/sg-en/codeascraft/scaling-etsy-payments-with-vitess-part-3--reducing-cutover-risk

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-result snippets.

## Problem
Etsy's payments data lived in four large unsharded MySQL databases that were becoming a scaling bottleneck. The team needed to migrate 23 tables totaling over 40 billion rows into a sharded environment managed by Vitess without disrupting a critical, high-traffic production system, with the final cutover being especially risky since certain query errors only surface after traffic actually moves to the new sharded setup.

## Solution
This three-part series covers reshaping the payments data model to be 'shard-friendly' (Part 1), the 'seamless' migration of 23 tables/40+ billion rows into a Vitess-sharded environment executed over roughly 18 months (Part 2), and de-risking the production cutover (Part 3) by exhaustively testing every application query for compatibility against the sharded keyspace beforehand, letting the team fix incompatible queries proactively rather than after cutover.