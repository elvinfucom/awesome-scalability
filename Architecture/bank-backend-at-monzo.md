# Bank Backend at Monzo

**Source:** https://monzo.com/blog/2016/09/19/building-a-modern-bank-backend/

## Problem
As a startup bank, Monzo needed infrastructure that could run continuously with no downtime or maintenance windows, scale toward hundreds of millions of potential customers, and support rapid, frequent feature deployment, all while being manageable by a small engineering team â€” incompatible with traditional batch-oriented banking infrastructure.

## Solution
Monzo runs services as Docker containers on Kubernetes, having migrated from Mesos/Marathon, cutting production infrastructure costs to about a quarter of the prior setup. Services are polyglot and communicate only via RPC, decoupled from shared infrastructure concerns by linkerd (built on Twitter's Finagle), which provides load balancing and automatic retries. Kafka provides an asynchronous, replicated commit-log backbone for high-availability messaging with replay, used for payment processing tasks that must never be dropped.