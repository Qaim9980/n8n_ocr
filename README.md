# Temu & Daraz Product Comparison (n8n)

This workspace contains an n8n workflow that takes a user query (via Chat Trigger), searches Google (via SerpAPI) for Temu and Daraz results, filters by rating and satisfaction, generates an AI comparison, merges the analysis with product data, and appends the results to Google Sheets.

## Workflow
- File: scraper.json (active)
- Name: "Product Scraper - Temu & Daraz - Full Flow"
- Steps:
  1) Chat Trigger → accepts a free-text product query
  2) SerpAPI Search → Google search with `site:temu.com OR site:daraz.pk`
  3) Extract Results → normalize SerpAPI output
  4) Transform Fields → compute `rating`, `reviews`, `price`, `satisfaction` (≈ rating/5*100)
  5) Filter High Satisfaction → rating ≥ 4 and satisfaction ≥ 80
  6) Generate Comparison Data → builds a concise AI prompt
  7) AI Analysis → calls OpenAI Chat Completions (model configurable)
  8) Merge Analysis → merges AI output back into each item
  9) Append to Google Sheets → appends auto-mapped columns

## Required Secrets / Credentials
Configure these in n8n (Settings → Credentials/Secrets):

- SERP_API_KEY: Your SerpAPI key (used in `SerpAPI Search`)
- OPENAI_API_KEY: Your OpenAI API key (used in `AI Analysis`)
- GOOGLE_SHEETS_ID: Target Google Sheet ID (used in `Append to Google Sheets`)
- Google Sheets OAuth2 credential: Link it on the `Append to Google Sheets` node

## Google Sheet Setup
- Sheet name: `Sheet1` (or adjust node `range`)
- Auto-map is enabled. Recommended headers include: `timestamp, search_query, product_name, price, rating, reviews, satisfaction, platform, quality_score, url, ai_analysis, comparison_group`

## Changing Thresholds
- Node: `Filter High Satisfaction`
  - Rating threshold: 4 (>=)
  - Satisfaction threshold: 80% (>=)

## Running
- Trigger the workflow via the Chat Trigger with your product query (e.g., "wireless mouse").
- Results will be appended to your configured Google Sheet.

## Notes
- Satisfaction is computed as an approximate metric from rating (rating/5*100). If you need actual on-site satisfaction percentages, add an HTTP Request step per product and parse with an AI or HTML-extraction step.
- To target other Daraz regions, expand the site filter (e.g., `site:daraz.pk OR site:daraz.lk OR site:daraz.com.bd`).
