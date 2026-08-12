---
name: Buy a Deep Sentinel camera pack
description: Search the Deep Sentinel store catalog, build a cart, and take a buyer
  through checkout using the store's Universal Commerce Protocol MCP endpoint.
api: https://shop.deepsentinel.com/api/ucp/mcp
protocol: UCP 2026-04-08 over MCP
generated: '2026-08-12'
method: generated
source: mcp/deep-sentinel-ucp-mcp-tools-list.json
operations:
  - search_catalog
  - lookup_catalog
  - get_product
  - create_cart
  - update_cart
  - get_cart
  - create_checkout
  - update_checkout
  - get_checkout
  - complete_checkout
  - cancel_checkout
  - get_order
---

# Buy a Deep Sentinel camera pack

Deep Sentinel sells hardware camera packs bundled with a recurring Live Guard monitoring
subscription. The store exposes a Universal Commerce Protocol surface over MCP, so an
agent can complete the whole purchase without screen-scraping.

Every tool name below was read from a live `tools/list` response on 2026-08-12 and is
saved verbatim in `mcp/deep-sentinel-ucp-mcp-tools-list.json`, including each tool's
full JSON Schema 2020-12 `inputSchema`. Read that file for the authoritative input
contract before calling anything.

## Before you start

- **Endpoint** — `POST https://shop.deepsentinel.com/api/ucp/mcp`, `Content-Type: application/json`.
- **No credentials are required** for catalog, cart and checkout discovery. The endpoint
  answered anonymously.
- **Confirm the protocol version first** — `GET https://shop.deepsentinel.com/.well-known/ucp`.
  The store currently advertises `2026-04-08` (latest stable) and still supports
  `2026-01-23`.
- **Agent identity is mandatory.** Every tool requires a `meta` object containing
  `ucp-agent.profile`, a URI identifying your agent. A call without it will not validate.
- **US only.** Deep Sentinel service is limited to properties in the United States, and
  monitoring requires a 1-year minimum contract. Say this to the buyer before checkout,
  not after.

## Steps

1. **Find the right pack.** Call `search_catalog` with the buyer's intent (number of
   cameras, indoor vs outdoor, wireless vs wired). Use `lookup_catalog` when you already
   have an identifier, and `get_product` for the full detail of one item.
2. **Quote the real total.** Prices come back as integer ISO 4217 minor units paired with
   a currency code — `{"amount": 49900, "currency": "USD"}` is `$499.00`. Divide by 100
   for two-decimal currencies before you say a number to the buyer. Also tell them the
   recurring monitoring cost: monitoring is billed at $55/month per camera on top of the
   hardware price (see `plans/deep-sentinel-plans-pricing.yml`).
3. **Build the cart.** `create_cart` with the selected items, then `update_cart` to change
   quantities and `get_cart` to re-read state. `cancel_cart` abandons it.
4. **Open a checkout.** `create_checkout` returns line items, totals, discounts and taxes.
   Cart and checkout IDs are Shopify global IDs in the form `gid://shopify/Checkout/abc123`
   — pass them back exactly as received; do not reconstruct them.
5. **Set shipping and any discount.** `update_checkout` carries the shipping address and
   method. The store's fulfillment capability declares
   `allows_multi_destination.shipping: false`, so a single checkout ships to one address.
6. **Complete only with buyer approval.** `complete_checkout` finalizes the purchase and
   charges the payment instrument. The store's own agent instructions state the buyer
   must approve payment. Never call this tool on your own initiative — get explicit
   confirmation of the final total, the address and the recurring monthly charge first.
7. **Confirm.** `get_order` returns the placed order. `cancel_checkout` backs out before
   completion.

## Retry and failure rules

- **There is no idempotency contract.** None of the 14 tool schemas accepts an
  idempotency key, and the provider documents none. Do **not** blind-retry
  `complete_checkout` on a timeout or an ambiguous response — re-read state with
  `get_checkout` (and `get_order`) and decide from the observed state. A naive retry can
  produce a duplicate purchase.
- Errors arrive as JSON-RPC 2.0 error objects. Deep Sentinel publishes no error catalog,
  so surface the raw message to the buyer rather than guessing a remediation.
- No rate limits are documented and no `RateLimit-*` headers were observed. Back off
  politely on any non-200.

## Payment

The merchant profile declares two payment handlers: `com.google.pay` and
`dev.shopify.card`. Do not handle raw card data yourself — use the declared handler flow.
