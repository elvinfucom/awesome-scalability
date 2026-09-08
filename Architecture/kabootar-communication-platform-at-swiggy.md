# Kabootar: Communication Platform at Swiggy

**Source:** https://bytes.swiggy.com/kabootar-swiggys-communication-platform-e5a43cc25629

> **Note:** Direct fetch failed after multiple attempts; summary reconstructed from search-indexed excerpts of the article.

## Problem
Swiggy needed a unified way to send communications (notifications, alerts, campaigns) to its many types of end users â€” customers, delivery executives, restaurant vendors, and customer care executives â€” across different triggers, without every team building bespoke, one-off messaging integrations.

## Solution
Swiggy built Kabootar, a communication platform of about 8 microservices connected via a RabbitMQ backbone following Enterprise Integration Patterns. Each service is layered into an integration layer, a core processing layer independent of integration mechanism, and a consumer/delivery layer, letting the platform receive a message over RabbitMQ but emit via Kafka or vice versa. Campaigns can be event-based or schedule-based (one-time or recurring via cron).