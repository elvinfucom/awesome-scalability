# Real-time Presence Platform at LinkedIn

**Source:** https://engineering.linkedin.com/blog/2018/01/now-you-see-me--now-you-dont--linkedins-real-time-presence-platf

## Problem
LinkedIn needed to show accurate online/offline presence indicators for roughly 500 million members in real time, processing thousands of status changes per second. Mobile connections are inherently unstable â€” frequent disconnects from poor networks or brief interruptions â€” so naively tying presence to raw connection state would make a member's status flicker rapidly.

## Solution
LinkedIn built a heartbeat-based presence system where each client emits periodic heartbeats, and a member is kept marked online as long as a heartbeat arrives within a small buffer window, absorbing brief network blips. For every online member, the system spins up one lightweight Akka actor that schedules a delayed offline-check timer, giving proactive rather than purely reactive offline detection. The platform is built on Play Framework and Akka, backed by a distributed key-value store, scaling horizontally with each node handling about 1.8K QPS while propagating status changes in under 200ms at p99.