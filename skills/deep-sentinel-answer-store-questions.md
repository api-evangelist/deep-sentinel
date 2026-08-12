---
name: Answer Deep Sentinel store and policy questions
description: Answer buyer questions about Deep Sentinel's return, shipping, warranty and
  contact policies from the store's own storefront MCP endpoint instead of guessing.
api: https://shop.deepsentinel.com/api/mcp
generated: '2026-08-12'
method: generated
source: mcp/deep-sentinel-mcp-tools-list.json
operations:
  - search_shop_policies_and_faqs
---

# Answer Deep Sentinel store and policy questions

Deep Sentinel serves a second, separate MCP endpoint alongside its commerce one. It
exposes exactly one tool, read from a live `tools/list` on 2026-08-12 and saved in
`mcp/deep-sentinel-mcp-tools-list.json`.

## Before you start

- **Endpoint** — `POST https://shop.deepsentinel.com/api/mcp`, `Content-Type: application/json`.
- **No credentials required.** The endpoint answered anonymously.
- This endpoint is **not** the commerce endpoint. Catalog, cart and checkout live at
  `/api/ucp/mcp` — see `deep-sentinel-buy-camera-pack.md`.

## Steps

1. Call `search_shop_policies_and_faqs` with:
   - `query` (**required**) — the buyer's question in natural language, e.g. "What is
     your return policy?", "Do you ship to Canada?", "What is your phone number?"
   - `context` (optional) — anything about the buyer that should shape the answer
     (location, urgency, whether they are an existing customer).
2. Answer from what the tool returns. Do not fill gaps from memory — Deep Sentinel's
   contract terms are specific and getting them wrong is costly for the buyer.
3. When the tool has no answer, fall back to the published pages rather than inventing
   one:
   - Return policy — <https://www.deepsentinel.com/return-policy/>
   - End User Terms of Service — <https://www.deepsentinel.com/end-user-terms-of-service/>
   - Privacy Policy — <https://www.deepsentinel.com/privacy-policy/>
   - Support / knowledge base — <https://support.deepsentinel.com/>

## Facts worth stating unprompted

These come from Deep Sentinel's published pricing page and are the terms buyers most
often miss:

- Live Guard monitoring requires a **1-year minimum service contract**.
- There is a **30-day money-back guarantee**.
- Service is available for **United States properties only**.
- Monitoring is **$55/month per camera**, separate from the hardware price.
- Each camera needs **1.5 Mbps of upload**, on a 5 Mbps base.
