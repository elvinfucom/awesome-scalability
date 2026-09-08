# Handles Millions of Digital Transactions Safely Everyday at Paytm

**Source:** https://paytm.com/blog/engineering/how-paytm-handles-millions-of-digital-transactions-safely-everyday/

## Problem
Paytm needed a database architecture capable of very high-throughput reads and writes for digital transactions while keeping data synchronized and consistent across database servers and clusters spread over multiple data centers, using primarily open-source technology.

## Solution
Paytm adopted the Paxos consensus algorithm with a single-leader approach to reach agreement across machines in multiple data centers, ensuring transactions are durably committed once acknowledged by a majority of replicas. Writes are first applied to logical databases before being mapped to physical shards, applying Log-Structured Merge (LSM) tree concepts to improve write throughput, built on the open-source InnoDB storage engine with configurable consistency levels (strong, weak, medium) calibrated per use case.