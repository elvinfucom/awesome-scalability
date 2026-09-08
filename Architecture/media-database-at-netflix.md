# Media Database at Netflix

**Source:** https://medium.com/netflix-techblog/implementing-the-netflix-media-database-53b5a840b42a

> **Note:** Direct fetch returned HTTP 403; summary compiled from search-result snippets.

## Problem
Netflix's media processing systems needed a data store to hold metadata about media assets that could serve multiple, logically tiered business applications with very different requirements, some demanding strict consistency, durability, and availability guarantees at large scale, which existing general-purpose stores didn't cleanly satisfy.

## Solution
Netflix built the Netflix Media Database (NMDB), using immutability and read-after-write consistency as core design precepts, with Apache Cassandra as the backing store serving as the system of record for all media metadata. The design provisions different consistency/availability tiers for different classes of applications built atop NMDB, with Elasticsearch layered in for query needs.