# Back-end at Flickr

**Source:** https://yahooeng.tumblr.com/post/157200523046/introducing-tripod-flickrs-backend-refactored

## Problem
Yahoo wanted to reuse Flickr's proven, high-throughput photo infrastructure (sustaining over 500 photo uploads per second) across other Yahoo products such as Mail, Messenger, and Answers Now, serving roughly a billion users collectively. However, Flickr's existing backend was a monolithic system built specifically for Flickr and was not designed to be multi-tenant or configurable for other applications.

## Solution
Yahoo refactored Flickr's backend into 'Tripod,' three specialized, reusable microservices: a Pixel Service for ingestion/storage/resizing/delivery, an Enrichment Service applying computer-vision algorithms to extract metadata, and an Aggregation Service built on Vespa to index that metadata for cross-application search. The system supports multi-tenancy via a bucket-based data model with OAuth 2.0 controlling per-application permissions, and the stack includes Java Spring MVC, a Pulsar event bus, Redis Cluster, HBase/Storm, and Hadoop for batch backfills.