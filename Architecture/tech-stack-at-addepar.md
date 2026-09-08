# Tech Stack at Addepar

**Source:** https://medium.com/build-addepar/our-tech-stack-a4f55dab4b0d

> **Note:** Direct fetch returned HTTP 403; summary reconstructed from search-engine snippets/cached excerpts.

## Problem
Addepar, a wealth/portfolio management data platform managing over $750 billion in assets, needed a front-end and tooling stack capable of building rich, data-heavy financial visualizations and handling hundreds of gigabytes of data daily along with roughly 2.5 million API requests per day.

## Solution
Addepar built its client application on Ember.js as the core framework, using Handlebars/HTMLBars for templating, D3.js for complex financial data visualizations, and SASS for styling, running on Node LTS with Yarn for package management. The team also open-sourced several Ember-based libraries (Ember Charts, Ember Table, Ember Widgets) developed in-house.