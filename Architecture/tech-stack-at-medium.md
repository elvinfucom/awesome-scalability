# Tech Stack at Medium

**Source:** https://medium.engineering/the-stack-that-helped-medium-drive-2-6-millennia-of-reading-time-e56801f7c492

> **Note:** Direct fetch returned HTTP 403 (Forbidden) from medium.engineering. Summary reconstructed from search-result snippets and secondary sources describing the same post.

## Problem
Medium needed an infrastructure capable of serving over 25 million unique readers a month and tens of thousands of new posts a week, while letting a relatively small engineering team iterate quickly on a publishing and social platform. They needed to balance developer productivity (shared code between server and client) against the operational complexity of running many services reliably at growing scale.

## Solution
Medium's main app servers were built in Node.js so code (views, models) could be shared between server and client, with auxiliary internal services written in Go for easy building, packaging, and deployment. Static assets were served from S3 behind CloudFront as a CDN, with nginx as a reverse proxy in front of app servers; email went through SES and background/async work was processed via SQS-backed workers. They migrated their datastore to DynamoDB ahead of public launch for scalability, and used Datadog for monitoring and PagerDuty for alerting operational issues.