# Lightweight Distributed Architecture to Handle Thousands of Library Releases at eBay

**Source:** https://tech.ebayinc.com/engineering/a-lightweight-distributed-architecture-to-handle-thousands-of-library-releases-at-ebay/

## Problem
eBay needed to migrate and release over 3,000 legacy libraries from Ant to Maven, a process complicated by intricate interdependencies between libraries. The release system needed to maximize parallelism across multiple Jenkins build nodes while remaining lightweight and fault-tolerant â€” the original manual/centralized approach was slow and hard to scale.

## Solution
eBay's engineers parsed each library's pom.xml to build a dependency DAG, prioritizing libraries with no dependencies first and giving higher release priority to nodes with more downstream dependents. They replaced an initial 'push mode' with a 'pull mode' where Jenkins nodes continuously request the next available release job from a central service, reusing local Maven dependency caches. This pull-based, dependency-prioritized design cut total release time for 3,000+ libraries from 2-3 days down to about 2 hours.