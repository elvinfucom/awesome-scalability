# Settings Platform at LinkedIn

**Source:** https://engineering.linkedin.com/blog/2019/05/building-member-trust-through-a-centralized-and-scalable-setting

## Problem
LinkedIn's member settings were scattered across many independently-built, custom services, forcing duplicated development effort and inconsistent behavior. The legacy model only supported simple boolean/enum values, which could not handle emerging needs like hierarchical or multi-keyed settings. Adding a single new setting required manual code changes and took an average of about five weeks, a major bottleneck given roughly 50 new settings were added per year.

## Solution
LinkedIn built a centralized Settings Platform composed of a Settings Mid Tier Service (a Rest.li microservice), a Setting Values Service computing hierarchical/effective setting values persisted in Espresso, and a Setting Metadata Service storing each setting's schema in an Oracle database with a DRAFT/ACTIVE/DEPRECATED versioning workflow. A self-service developer tool lets teams define new settings via metadata rather than code deployments, and the platform was designed to support over 600,000 queries per second.