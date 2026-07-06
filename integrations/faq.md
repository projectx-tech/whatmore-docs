---
title: "Backend Questions & Clarifications"
---

Answers to the questions backend teams most commonly raise.

## 1. How does Whatmore get my product data?

Two ways (see [Catalog API](/integrations/catalog-api)):

- **Dashboard connect (recommended):** you give Whatmore your product API endpoint +
  credentials and map fields in the dashboard; Whatmore **pulls** product data for you.
- **Catalog API (automation):** you **push** products with `POST /product` /
  `POST /product/upload/bulk` and keep them fresh with `PUT /v1/product`.

Either way Whatmore stores the data and serves it to the surfaces.

**What fields does a product have?** `client_product_id`, `product_link`, `title`,
`description`, `price`, `compare_price`, `currency`, `thumbnail_image`, `product_status`,
plus `product_metadata` for `sku` / `variant_id`. See the
[Catalog API response](/integrations/catalog-api#fetch-a-product).

## 2. Catalog synchronization

- **Connect / initial load:** point Whatmore at your product API in the dashboard, or push
  with `POST /product` / bulk-by-URL
  [`POST /product/upload/bulk`](/integrations/catalog-api#bulk-import-large-catalogs).
- **Incremental updates:** on the dashboard connect Whatmore re-reads your API; on the push
  path, send `PUT /v1/product` by `client_product_id` when price or inventory changes.
- **Fetch / verify:** `GET /events/product/{client_product_id}`, or list with
  `GET /brand/{store_id}/products`.

Because you reference products by *your own* `client_product_id` (commonly the URL), there
is no separate id-mapping to maintain. **Video and media are managed in the dashboard — no
upload API to build.**

## 3. Bulk vs single fetch

Single-product fetch (`GET /events/product/{client_product_id}`) and a per-store list
(`GET /brand/{store_id}/products`) are available. For large catalogs, bulk import takes a
list of product URLs — see
[Bulk import](/integrations/catalog-api#bulk-import-large-catalogs).

## 4. Order tracking / "webhooks"

Order data is **pushed by you** to `POST /external-shop-order-tracking/private` on order
completion — see [Order Tracking](/integrations/order-tracking). Key semantics:

- **Idempotency:** orders are de-duplicated by `order_id`; a repeat is rejected
  (`Order Id already exists`), never double-counted — so retries are safe.
- **Timeout / retry:** send one call per order; on network failure, re-send the same
  `order_id`. Recommended retry cadence is confirmed at onboarding.

## 5. API Contracts

Concrete request/response examples for every endpoint are on the
[Catalog API](/integrations/catalog-api) and [Order Tracking](/integrations/order-tracking) pages. A formal
OpenAPI/Swagger export can be provided on request.

## 6. Security

- **Authentication:** all calls use a bearer access token obtained from
  `GET /auth/access-token` with your `store_id` — see [Authentication](/integrations/authentication).
- **Environment separation:** production and staging issue separate `store_id`s and tokens.
- **Token handling:** keep the token server-side; the App SDK uses only the public Brand ID.

## 7. Performance

> Confirmed jointly at onboarding: API rate limits, expected response times, and any
> recommendations (token caching, batching catalog updates). As a baseline, cache the
> access token and send catalog updates only on change rather than on a schedule.
