# Architecture of Evernote

**Source:** https://evernote.com/blog/a-digest-of-evernotes-architecture/

> **Note:** The original URL now 404s; summary reconstructed from a detailed third-party (High Scalability) write-up of the original post and corroborating search snippets.

## Problem
Evernote needed to serve roughly 9-9.5 million users generating up to 150 million HTTPS requests per day, on a freemium business model with only about a 1% paid-conversion rate, meaning they had to handle a lot of data without spending a lot of money on expensive cloud infrastructure.

## Solution
Evernote ran its own hardware out of two dedicated data-center cages rather than using cloud infrastructure, sharding its 9.5 million users across roughly 90 shards of about 100,000 users each. Each shard was a pair of dual quad-core servers with direct-attached drives running Debian, Java, Tomcat, and MySQL, with primary-secondary DRBD replication and Heartbeat-managed failover for redundancy, plus nightly backups to a secondary data center. A separate fleet of servers handled specialized workloads like image processing, handwriting recognition, and search.