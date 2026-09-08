# Architecture of Following Feed, Interest Feed, and Picked For You at Pinterest

**Source:** https://medium.com/@Pinterest_Engineering/building-a-dynamic-and-responsive-pinterest-7d410e99f0a9

> **Note:** Direct fetch returned HTTP 403; summary compiled from search-result snippets describing the article's architecture details.

## Problem
Pinterest's original feed products relied on pregenerated content and batch jobs, so ranking features could be days or weeks stale. Directly querying MySQL and HBase at request time was too slow for an online, low-latency feed, so Pinterest needed real-time, responsive feed generation at scale.

## Solution
Pinterest built a 'smart feed' system where a SmartFeed worker ingests Pins from multiple sources, scores each Pin proportional to predicted value, and writes scored Pins into per-source-type pools stored in HBase using key-based sorting so the system can efficiently scan pins in score order for any user. The worker is invoked via PinLater, an asynchronous job execution system tolerant of delays and failures.