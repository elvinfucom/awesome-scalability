# Architecture of API Gateway at Uber

**Source:** https://eng.uber.com/architecture-api-gateway/

## Problem
Uber needed a single point of entry for all its apps to reach thousands of internal microservices, spanning multiple wire protocols (JSON, Thrift, Protobuf/gRPC) while consistently enforcing auth, rate limiting, load shedding, and security auditing. They also needed to retire a legacy Node.js-based gateway that had become a bottleneck, migrating 1,500+ APIs off it without disrupting backend teams.

## Solution
Uber built a gateway (via the open-sourced Zanzibar framework) with a protocol manager, middleware stack, endpoint handler, and protocol-aware client layer. Rather than dynamic runtime configuration, they generate the gateway statically at build time from YAML plus Thrift IDL, compiling to native Go code for performance and type safety, which also auto-generates mobile client SDKs. The gateway scaled to roughly 500K QPS while migrating 1,500+ APIs from the legacy system.