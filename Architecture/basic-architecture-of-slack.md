# Basic Architecture of Slack

**Source:** https://slack.engineering/how-slack-built-shared-channels-8d42c895b19f

## Problem
Slack's core architecture treats each workspace as the atomic unit for data partitioning, with data sharded per workspace. Introducing shared channels, which let users access channels across workspace boundaries, broke that core assumption, and naively duplicating channel data across every participating workspace's shard risked consistency problems and would not scale given Slack was already handling more than a billion messages per week.

## Solution
Slack kept a single source of truth: one copy of a shared channel's data lives on the shard of the workspace that originated it. A new shared_channels database table acts as a bridge, storing one row per participating workspace with the channel ID and any property overrides. They decoupled channel privacy from the ID scheme, extended the Flannel edge-cache service to support cross-workspace user profiles and presence, and implemented cross-workspace data governance including retention policies and Enterprise Key Management.