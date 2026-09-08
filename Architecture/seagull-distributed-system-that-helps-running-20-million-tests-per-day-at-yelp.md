# Seagull: Distributed System that Helps Running > 20 Million Tests Per Day at Yelp

**Source:** https://engineeringblog.yelp.com/2017/04/how-yelp-runs-millions-of-tests-every-day.html

## Problem
Yelp's monolithic codebase had grown to nearly 100,000 tests, and running them sequentially took roughly 2 days per deployment cycle. With over 300 test runs triggered daily and 30-40 simultaneous runs during peak hours, sequential execution was a severe bottleneck on developer productivity.

## Solution
Yelp built Seagull, an in-house distributed testing system that parallelizes test execution across a large cluster using Apache Mesos, Docker, and AWS. Seagull bin-packs tests into roughly 10-minute bundles using a greedy algorithm and linear programming for dependency handling. At peak the system launches more than 2 million Docker containers per day, cutting test turnaround time from 2 days to about 30 minutes, with a companion auto-scaler (FleetMiser) cutting infrastructure costs by about 80%.