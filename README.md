# Carousell Listing Monitor

An n8n workflow that checks Carousell Malaysia for new listings on a schedule and sends an alert to Discord when a new one appears.

I built it to catch Xbox Series S / X listings, which sell fast. The keyword can be changed to track any product.

![Workflow](workflow.png)

## How it works

```
Schedule Trigger → Apify (scrape Carousell) → Edit Fields → If (keyword filter) → Remove Duplicates → Discord
```

1. **Schedule Trigger** starts the workflow at a set interval.
2. **Apify** runs a Carousell scraper on the search page, sorted by newest first.
3. **Edit Fields** keeps only the fields needed: title and listing URL.
4. **If** keeps listings whose title contains the keyword (case-insensitive).
5. **Remove Duplicates** remembers every listing URL from past runs and only lets new ones through.
6. **Discord** sends the title and link to a channel through a webhook.

## Problem I had to solve

Carousell protects its pages with Cloudflare. A normal HTTP Request from n8n (even from a home internet connection) gets a "Just a moment... Enable JavaScript" challenge page instead of listings, because the HTTP Request node cannot run JavaScript.

To get around this, the workflow uses **Apify**, which loads the page in a real browser through a Malaysian residential proxy.

## Requirements

- [n8n](https://n8n.io/) (self-hosted or cloud)
- The Apify community node: `@apify/n8n-nodes-apify` (install from **Settings → Community Nodes**)
- An [Apify](https://apify.com/) account and API token
- A Discord channel with a webhook URL

## Setup

1. Import `carousell-listing-monitor.json` into n8n.
2. In the Apify node, add your Apify API token as a credential.
3. In the Discord node, add your Discord webhook URL as a credential.
4. Run the workflow once manually to check that it works.
5. Publish the workflow so the schedule starts.

> Note: the first run sends every listing it finds, because it has not seen any of them before. After that, only new listings are sent.

## Changing what it searches for

Edit two places:

1. **Apify node input**: change the search in the URL:
   ```
   https://www.carousell.com.my/search/YOUR%20KEYWORD?sort_by=3
   ```
   `sort_by=3` sorts results by most recent.
2. **If node**: change the text the title must contain.

## Cost

Apify's free plan gives $5 of usage per month. Each run loads one page (`maxRequestsPerCrawl: 1`) to keep the cost low. Running more often means more runs, so the schedule should match the budget.

## Limitations

- If the PC running n8n is off or asleep, scheduled runs are skipped.
- If the Apify credit runs out, runs stop until the next month.
- The keyword filter is simple, so accessories that mention the keyword (cases, storage cards) can also pass.

## Built with

- n8n
- Apify
- Discord webhooks
