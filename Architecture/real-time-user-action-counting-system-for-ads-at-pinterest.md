# Real-time User Action Counting System for Ads at Pinterest

**Source:** https://medium.com/@Pinterest_Engineering/building-a-real-time-user-action-counting-system-for-ads-88a60d9c9a

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-indexed excerpts of the article.

## Problem
Pinterest's real-time ads auction system needed accurate, low-latency counts of each user's past actions on ads to support behavioral targeting and frequency capping. Because ad decisions are made in real time during bidding/serving, these action counts had to be served immediately rather than through slower offline aggregation, at Pinterest's scale.

## Solution
Pinterest built Aperture, an in-house time-series data storage and online event-tracking service, to power user action counting for ads. Aperture uses RocksDB as its embedded storage engine and Helix-enabled Rocksplicator for cluster management and replication, exposing low-latency APIs for appending, deduplicating, and aggregating per-user event data that the ads system queries directly for targeting and frequency control.