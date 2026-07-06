# Backend Questions & Clarifications

Answers to the questions backend teams most commonly raise.

## 1. Product data — when is it read?

Whatmore stores your product data (you sync it via the [Catalog API](catalog-api.md)) and
serves it to the video surfaces. You do **not** host a product API for Whatmore to call.
Products render from Whatmore's stored copy, which you keep current with `PUT /v1/product`.

**What fields does a product have?** `client_product_id`, `product_link`, `title`,
`description`, `price`, `compare_price`, `currency`, `thumbnail_image`, `product_status`,
plus `product_metadata` for `sku` / `variant_id`. See the
[Catalog API response](catalog-api.md#fetch-a-product).

## 2. Catalog synchronization

**How do products sync?** You push them to Whatmore:

- **Initial load / add:** `POST /product` per product, or bulk-by-URL via
  [`POST /product/upload/bulk`](catalog-api.md#bulk-import-large-catalogs) for large catalogs.
- **Incremental updates:** `PUT /v1/product` by `client_product_id` — send it whenever
  price or inventory changes.
- **Fetch / verify:** `GET /events/product/{client_product_id}`, or list with
  `GET /brand/{store_id}/products`.

Because you reference products by *your own* `client_product_id` (commonly the URL), there
is no separate id-mapping to maintain. **Video and media are managed in the dashboard — no
upload API to build.**

## 3. Bulk vs single fetch

Single-product fetch (`GET /events/product/{client_product_id}`) and a per-store list
(`GET /brand/{store_id}/products`) are available. For large catalogs, bulk import takes a
list of product URLs — see
[Bulk import](catalog-api.md#bulk-import-large-catalogs).

## 4. Order tracking / "webhooks"

Order data is **pushed by you** to `POST /external-shop-order-tracking/private` on order
completion — see [Order Tracking](order-tracking.md). Key semantics:

- **Idempotency:** orders are de-duplicated by `order_id`; a repeat is rejected
  (`Order Id already exists`), never double-counted — so retries are safe.
- **Timeout / retry:** send one call per order; on network failure, re-send the same
  `order_id`. Recommended retry cadence is confirmed at onboarding.

## 5. API Contracts

Concrete request/response examples for every endpoint are on the
[Catalog API](catalog-api.md) and [Order Tracking](order-tracking.md) pages. A formal
OpenAPI/Swagger export can be provided on request.

## 6. Security

- **Authentication:** all calls use a bearer access token obtained from
  `GET /auth/access-token` with your `store_id` — see [Authentication](authentication.md).
- **Environment separation:** production and staging issue separate `store_id`s and tokens.
- **Token handling:** keep the token server-side; the App SDK uses only the public Brand ID.

## 7. Performance

> Confirmed jointly at onboarding: API rate limits, expected response times, and any
> recommendations (token caching, batching catalog updates). As a baseline, cache the
> access token and send catalog updates only on change rather than on a schedule.
