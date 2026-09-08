# Nearline System for Scale and Performance (2 parts) at Glassdoor

**Source:** https://medium.com/glassdoor-engineering/building-a-nearline-system-for-scale-and-performance-part-ii-9e01bf51b23d

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-indexed excerpts of the article.

## Problem
Glassdoor needed to track, close to real-time, which job listings a user has already been shown, so it could avoid resurfacing the same jobs and improve search relevance. Doing this purely in an offline batch pipeline was too slow, but doing it fully synchronously risked overwhelming core systems, so they needed an intermediate ('nearline') approach.

## Solution
Glassdoor built a nearline system combining offline batch processing with a near-real-time event pipeline: user and job-listing events are pushed onto AWS Kinesis streams and consumed to update state quickly without requiring full synchronous processing. Processed nearline state is stored in Cassandra, chosen for its fast read/write performance, with the broader stack leaning on existing Kafka, Storm, and Redis infrastructure.