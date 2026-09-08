# Ads Pacing Service at Twitter

**Source:** https://blog.twitter.com/engineering/en_us/topics/infrastructure/2021/how-we-built-twitter-s-highly-reliable-ads-pacing-service

> **Note:** Direct fetch blocked (403); summary compiled from search-engine snippets and a third-party mirror.

## Problem
Twitter's ad budget-pacing logic originally lived as a 'Pacing Library' embedded inside the monolithic AdServer, tightly coupling budget-distribution logic to the ad-serving path, making it harder to evolve, test, and operate reliably as ad volume and campaign complexity grew.

## Solution
Twitter extracted pacing into a standalone Pacing Service, decoupled from the ad-serving stack. The service reads real-time spend data and budget data, then recomputes pacing parameters every 10 seconds, persisting results for the serving path to consume, forming a continuous feedback loop letting campaigns choose smooth or accelerated delivery.