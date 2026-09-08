# Architecture of League of Legends Client Update

**Source:** https://technology.riotgames.com/news/architecture-league-client-update

## Problem
Riot's League of Legends client ran on Adobe AIR/RTMP dating back to 2008, an increasingly outdated platform. Players wanted persistent connectivity/background presence, but the AIR client consumed excessive resources when idle, and as Riot grew into a multi-game studio, independent feature teams kept colliding on the same shared monolithic codebase.

## Solution
Riot rebuilt the client iteratively, eventually introducing a C++ 'foundation' microservice that exposes RTMP functionality as REST endpoints and WebSocket events, running at roughly 20MB of memory when minimized. The final architecture layers a Chromium Embedded Framework (CEF)-based UI on top, organized as an explicit plugin system with semantic versioning and dependency graphs, letting independent teams ship features autonomously and letting players receive personalized plugin sets based on entitlements/region.