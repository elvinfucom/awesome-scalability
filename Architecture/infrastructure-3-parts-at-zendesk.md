# Infrastructure (3 parts) at Zendesk

**Source:** https://medium.com/zendesk-engineering/the-history-of-infrastructure-at-zendesk-part-3-foundation-team-forming-and-evolving-9859e40f5390

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from web search result snippets referencing the article's content.

## Problem
By early 2018, Zendesk had a traditional organizational split where infrastructure/operations was completely separate from product engineering, even as the company raced to migrate from physical data centers to AWS. Leadership anticipated engineering headcount would more than double and reliability requirements would rise sharply, risking the old org structure becoming a bottleneck.

## Solution
Zendesk used the AWS migration as an opportunity to redesign its organization to match its target cloud architecture, forming a dedicated 'Foundation' team responsible for the reliable platform underpinning all product engineering. To design this, leaders researched how larger cloud-native companies structured infrastructure teams, consulting infrastructure leaders at companies like Facebook and Stripe, resulting in an evolved team structure intended to reduce cross-team bottlenecks as Zendesk scaled on AWS.