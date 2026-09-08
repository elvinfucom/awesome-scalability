# API Specification Workflow at WeWork

**Source:** https://engineering.wework.com/our-api-specification-workflow-9337448d6ee6

> **Note:** Direct fetch failed; summary compiled from a mirror of the article and corroborating search snippets.

## Problem
WeWork's API developers were duplicating effort across many artifacts for each API â€” Postman collections, mocks, contract tests, payload validation, and documentation â€” with no single source of truth, leading to inconsistent specifications and wasted engineering time.

## Solution
WeWork standardized on OpenAPI v3 as the canonical spec format, building a YAML registry tracking all API repositories and a central aggregator normalizing specs into unified OASv3 documents. From that single source of truth they generate documentation via ReDoc, enforce linting via Speccy in CI, run contract testing, spin up mock servers, and generate SDKs â€” driving up the number of documented APIs and substantially improving spec quality once linting was wired into CI/CD.