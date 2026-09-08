# Architecture of Chat Service (3 parts) at Riot Games

**Source:** https://engineering.riotgames.com/news/chat-service-architecture-persistence

## Problem
Riot's chat infrastructure relied on a single MySQL primary that could only be scaled vertically, requiring continual, expensive hardware additions to gain capacity. This created a single point of failure causing timeouts, and schema migrations required extensive planning and scheduled downtime.

## Solution
Riot evaluated MySQL sharding, MySQL Cluster, Cassandra, and Riak, ultimately migrating chat persistence to Riak, a distributed NoSQL key-value store, co-located with chat services and accessed via a protocol buffers interface. They used a replication factor of 3-5, a LevelDB storage backend with multi-version objects, CRDTs for conflict resolution, and JSON compression that shrank object sizes roughly 10x. The result was elimination of schema-migration downtime, friend-list loads under 10ms for 99% of requests, and zero uptime loss despite disk/network failures.