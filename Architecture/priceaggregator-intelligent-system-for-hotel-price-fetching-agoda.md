# PriceAggregator: Intelligent System for Hotel Price Fetching (3 parts) at Agoda

**Source:** https://medium.com/agoda-engineering/priceaggregator-an-intelligent-system-for-hotel-price-fetching-part-3-52acfc705081

## Problem
Agoda aggregates hotel room prices from non-direct suppliers, but each supplier only permits Agoda to fetch prices at a limited number of Queries Per Second, while Agoda's user search traffic far exceeds what that QPS allowance can cover, meaning most searches cannot get a fresh, direct price lookup.

## Solution
Agoda built PriceAggregator, combining SmartTTL, which computes an itinerary-specific cache time-to-live so valuable prices stay fresh longer, with SmartScheduler, which proactively sends fetch requests to suppliers at optimal QPS levels, balancing load and prioritizing the most valuable itineraries. This shift from passive/reactive to proactive, prediction-driven fetching significantly increased bookings in A/B experiments.