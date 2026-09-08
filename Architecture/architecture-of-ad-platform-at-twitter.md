# Architecture of Ad Platform at Twitter

**Source:** https://blog.twitter.com/engineering/en_us/topics/infrastructure/2020/building-twitters-ad-platform-architecture-for-the-future.html

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-result snippets/quotes describing this exact post.

## Problem
Twitter's ad platform started as a small team serving a single ad format built around a monolithic 'AdServer' funnel optimized for low latency. As the ads business grew roughly 10x with multiple ad formats, the monolith's tight coupling meant even adding a single new targeting attribute required threading changes through the entire funnel, hurting developer productivity.

## Solution
Twitter's Revenue Platform team decomposed the AdServer monolith into microservices organized in two layers: an Admixer front-end service fans requests out to multiple product-specific bidder services (ads-default, ads-takeover, ads-video, ads-map) using a scatter-gather pattern, then each bidder talks to a shared ads-selection service. This let each ad product's bidder evolve and scale independently without coordinating changes across the whole funnel.