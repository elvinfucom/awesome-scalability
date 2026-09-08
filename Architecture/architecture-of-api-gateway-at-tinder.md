# Architecture of API Gateway at Tinder

**Source:** https://medium.com/tinder/how-we-built-the-tinder-api-gateway-831c6ca5ceca

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from web search snippets and a secondary summary referencing the original article's content.

## Problem
Different Tinder application teams each used a different third-party API gateway solution, and none fully met Tinder's needs. Because each team's gateway ran on a different tech stack, managing, securing, and evolving them consistently became increasingly cumbersome, and Tinder needed centralized, strict authorization and abuse detection across all external-facing APIs given significant malicious traffic across 190 countries.

## Solution
Tinder built its own API Gateway, TAG (Tinder API Gateway), on top of Spring Cloud Gateway, a JVM-based framework within the Spring ecosystem. TAG centralizes all external-facing APIs and enforces strict, consistent authorization and security checks across teams, letting application teams spin up their own gateway instance simply by writing configuration rather than custom code, standardizing the tech stack across teams.