# Tech Stack at Shopify

**Source:** https://engineering.shopify.com/blogs/engineering/e-commerce-at-scale-inside-shopifys-tech-stack

## Problem
Shopify had to support massive, unpredictable traffic spikes from merchant flash sales while serving around 80,000 requests per second at peak across 600,000+ merchants on a single Ruby on Rails monolith. Shared infrastructure created dangerous single points of failure â€” most notably a Redis outage nicknamed 'Redismageddon' that took down the entire platform at once.

## Solution
Shopify sharded its database starting in 2014, splitting merchant data across dozens of partitions, and evolved this into a 'pod' architecture: fully isolated instances of the Shopify stack each with their own dedicated MySQL, Redis, and memcached, with 100+ pods deployed across regions so no single failure can cascade platform-wide. The core stack remains Ruby on Rails plus React/TypeScript on the frontend and GraphQL for Admin client communication, all running on Docker/Kubernetes (GKE) behind Nginx/OpenResty load balancers, with deploys managed via BuildKite and ShipIt â€” resulting in zero major platform-wide outages since adopting the pod model.