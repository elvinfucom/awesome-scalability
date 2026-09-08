# Cloud Infrastructure at Grubhub

**Source:** https://bytes.grubhub.com/cloud-infrastructure-at-grubhub-94db998a898a

> **Note:** Direct fetch failed (no response); summary reconstructed from web search snippets referencing the article and related coverage.

## Problem
Grubhub needed infrastructure that could reliably serve rapidly growing order volume and application traffic while running across a mix of environments: two AWS regions plus a legacy, self-owned data center. Managing consistent deployment, scaling, and orchestration for a large and growing number of microservices across this hybrid, multi-region footprint was a significant operational challenge.

## Solution
Grubhub built infrastructure-as-code and custom container-orchestration tooling to manage roughly 9,000 Docker containers running on AWS across its microservices fleet. Their engineering teams designed the AWS infrastructure and around 300 application microservices to automatically scale up and down to absorb large swings in demand, letting site reliability engineers extend the same tooling to developer teams for resilience during major demand spikes.