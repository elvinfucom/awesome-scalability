# Rapid Event Notification System at Netflix

**Source:** https://netflixtechblog.com/rapid-event-notification-system-at-netflix-6deb1d2b57d1

> **Note:** Direct fetch blocked (403); summary compiled from search-engine snippets.

## Problem
With over 220 million active members across many devices, Netflix needed to push server-initiated updates to devices in near real time so state stays consistent across a user's devices. Relying solely on client polling was too chatty, while relying solely on push failed when devices were offline, and the system needed to scale to very high event throughput with per-device reliability.

## Solution
Netflix built RENO (Rapid Event Notification System), which segments incoming events by priority into priority-specific AWS SQS queues consumed by dedicated compute clusters generating per-device notifications, storing notification history in Cassandra. It uses a hybrid push-pull delivery model with a fan-out pattern so a delivery failure to one device doesn't block others, handling roughly 150,000 events per second at peak.