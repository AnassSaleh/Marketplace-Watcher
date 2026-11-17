# Marketplace Watcher (Carousell Listing Monitor)

An n8n workflow that checks Carousell Malaysia for new listings every 2 hours and sends an alert to Discord when a new one appears.

I built it to catch Xbox Series S / X listings, which sell fast. The keyword can be changed to track any product.

## How it works

```
Schedule Trigger (every 2 hours) ─┐
                                  ├→ Apify (scrape Carousell) → Edit Fields → If (keyword filter) → Remove Duplicates → Discord
Manual Trigger ───────────────────┘
```

1. **Schedule Trigger** starts the workflow every 2 hours (timezone: Asia/Kuala_Lumpur). A **Manual Trigger** is also included for testing.
2. **Apify** runs the [Carousell Listings Scraper](https://console.apify.com/actors/9XAXWuXpnFXAMmciQ) (`automation-lab/carousell-listings-scraper`) on the search page, sorted by newest first. It returns up to 5 listings per run.
3. **Edit Fields** keeps only the fields needed: title, price, and listing URL. The URL is cleaned by removing everything after `?`, so the same listing always has the same URL.
4. **If** keeps listings whose title contains `xbox series` (case-insensitive).
5. **Remove Duplicates** remembers every listing URL from past runs and only lets new ones through.
6. **Discord** sends the price, title, and link to a channel through a webhook.

## Problem I had to solve

Carousell protects its pages with Cloudflare. A normal HTTP Request from n8n (even from a home internet connection) gets a "Just a moment... Enable JavaScript" challenge page instead of listings, because the HTTP Request node cannot run JavaScript.

To get around this, the workflow uses an **Apify** scraper, which handles loading the page and returns the listings as clean data.

## Requirements

- [n8n](https://n8n.io/) (self-hosted or cloud)
- The Apify community node: `@apify/n8n-nodes-apify` (install from **Settings → Community Nodes**)
- An [Apify](https://apify.com/) account and API token
- A Discord channel with a webhook URL

## Setup

1. Import `Marketplace Watcher.json` into n8n.
2. In the Apify node, add your Apify API token as a credential.
3. In the Discord node, add your Discord webhook URL as a credential.
4. Run the workflow once manually to check that it works.
5. Publish (activate) the workflow so the schedule starts.

> Note: the first run sends every matching listing it finds, because it has not seen any of them before. After that, only new listings are sent.

> No API keys or webhook URLs are stored in the JSON file. The credential entries are only names that point to credentials saved in your own n8n, so you must add your own.

## Changing what it searches for

Edit two places:

1. **Apify node input**: change the search in the URL:
   ```json
   {
     "startUrls": [
       { "url": "https://www.carousell.com.my/search/YOUR%20KEYWORD?sort_by=3" }
     ],
     "maxItems": 5,
     "includeDetails": false,
     "proxyType": "datacenter"
   }
   ```
   `sort_by=3` sorts results by most recent.
2. **If node**: change the text the title must contain.

You can also change the Discord message text in the **Discord** node.

## Cost

Apify's free plan gives $5 of usage per month. Each run only asks for 5 listings (`maxItems: 5`) and uses a datacenter proxy to keep the cost low. Running more often means more runs, so the schedule should match the budget.

## Limitations

- If the PC running n8n is off or asleep, scheduled runs are skipped.
- If the Apify credit runs out, runs stop until the next month.
- Only the 5 newest listings are checked each run. If more than 5 new listings appear between runs, some can be missed.
- The keyword filter is simple, so accessories that mention the keyword (cases, storage cards) can also pass.

## Built with

- n8n
- Apify
- Discord webhooks
