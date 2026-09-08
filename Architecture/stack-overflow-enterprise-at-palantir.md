# Stack Overflow Enterprise at Palantir

**Source:** https://medium.com/@palantir/terraforming-stack-overflow-enterprise-in-aws-47ee431e6be7

> **Note:** Direct fetch returned HTTP 403; summary compiled from search-result snippets.

## Problem
Palantir wanted to run Stack Overflow Enterprise, a complex Windows/.NET/SQL Server on-premises application, as an internal knowledge-sharing tool, but needed it to be self-healing, highly available, and manageable via modern infrastructure-as-code practices rather than manual server administration.

## Solution
Palantir migrated Stack Overflow Enterprise to an all-AWS architecture using HashiCorp's Packer to build machine images and Terraform to declaratively provision infrastructure. They isolated communication routes with front-end/back-end subnet separation and bastion hosts, working toward a reusable Terraform module so additional instances could be spun up simply by branching in Git.