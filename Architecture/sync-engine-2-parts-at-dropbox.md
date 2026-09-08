# Sync Engine (2 parts) at Dropbox

**Source:** https://dropbox.tech/infrastructure/-testing-our-new-sync-engine

## Problem
Dropbox's legacy sync engine had architectural flaws that made it nearly impossible to test reliably: it allowed invalid intermediate states, used path-based identifiers so a folder rename became an O(n) operation for every descendant, and relied on freely-spawned threads, global locks, and hard-coded timeouts, making bugs difficult to reproduce.

## Solution
Dropbox rewrote the engine as 'Nucleus,' modeling filesystem state as three trees (Remote, Local, Synced) using unique node IDs instead of paths so moves become atomic. Concurrency is simplified to a single control thread with dedicated background thread pools, enabling full serialization during tests. They built two randomized, seed-reproducible testing frameworks â€” CanopyCheck (fuzzing the sync-planning algorithm) and Trinity (end-to-end concurrency testing with chaos injection) â€” running tens of millions of randomized test runs nightly.