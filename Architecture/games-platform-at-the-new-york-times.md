# Games Platform at The New York Times

**Source:** https://open.nytimes.com/play-by-play-moving-the-nyt-games-platform-to-gcp-with-zero-downtime-cf425898d569

> **Note:** Direct fetch failed across multiple attempts; summary is best-effort based on the article title and related secondary references (e.g. a GCP Podcast episode discussing the migration) rather than the full article text.

## Problem
The New York Times needed to migrate its Games platform (including high-traffic puzzles like Crossword) from its existing infrastructure to Google Cloud Platform, without any downtime or disruption to the live, actively-used product during the cutover.

## Solution
The NYT team executed the migration using a careful, incremental cutover strategy onto GCP (leveraging Kubernetes/GKE-based infrastructure) so traffic could be shifted to the new environment gradually and safely, keeping the games platform fully available throughout the migration.