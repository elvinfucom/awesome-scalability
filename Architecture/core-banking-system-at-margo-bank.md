# Core Banking System at Margo Bank

**Source:** https://medium.com/margobank/choosing-an-architecture-85750e1e5a03

> **Note:** The source appears to actually be 'Memo Bank', not 'Margo Bank' (likely a title typo in the source list); summary compiled from search-engine snippets since Medium blocked direct fetch.

## Problem
Building a bank from scratch, the engineering team needed an architecture meeting strict requirements for a core banking system: strong maintainability, straightforward testability of business logic, and the ability to serve very different read models (e.g. customer-facing API vs. internal accounting/general-ledger API) from the same source of truth without those models becoming tangled.

## Solution
The bank adopted a CQRS and Event Sourcing architecture: events are the sole source of truth, and independent 'projections' on the read side rebuild their own representations by consuming the event stream. Multiple decoupled projections can coexist (client API vs. accountant API), and new projections can be introduced by replaying the full event log into a new database before cutting over â€” harder to set up than CRUD but more maintainable since most logic ends up as pure, testable functions.