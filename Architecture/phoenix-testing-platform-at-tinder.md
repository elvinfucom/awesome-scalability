# Phoenix: Testing Platform (3 parts) at Tinder

**Source:** https://medium.com/tinder-engineering/phoenix-tinders-testing-platform-part-iii-520728b9537

> **Note:** Direct fetch returned HTTP 403; summary based on search-result snippets and article metadata.

## Problem
Tinder needed to run large-scale A/B experiments and control feature rollouts across its mobile clients without coupling experiment logic to slow, inflexible app release cycles. They needed a reliable way to assign users to treatments, remotely configure experience delivery, and measure results.

## Solution
Tinder built Phoenix, an in-house experimentation platform composed of Ground Control (managing experiment lifecycle), an Assignment service (determining treatment per user), Levers (decoupling configuration from app releases), and a Metrics System. This let Tinder iterate on experiments quickly without shipping new app builds for every change.