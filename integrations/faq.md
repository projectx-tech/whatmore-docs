---
title: "Backend Questions & Clarifications"
timestamp: false
---

Answers to the questions backend teams most commonly raise.

## 1. How does Whatmore get my product data?

Two mechanisms (see [Catalog API](/integrations/catalog-api)):

- **[Pull](/integrations/catalog-api#pull-initial-load-and-refresh) (primary):** you give
  Whatmore your product API endpoint + credentials and add product URLs in the dashboard;
  Whatmore **pulls** each product from your API — for the initial load, every new product, and
  on refresh. There is no product-creation API for you to call.
- **[Push](/integrations/catalog-api#push-real-time-updates) (optional):** when price or
  availability changes, you **push** `PUT /v1/product` so the change reflects immediately.

Whatmore stores the data and serves it to the surfaces.

**What fields does a product have?** Your product API returns them and you map them in the
dashboard — see [Expected product JSON](/integrations/catalog-api#expected-product-json).

## 2. Catalog synchronization

- **Initial load & refresh (pull):** point Whatmore at your product API in the dashboard and
  add product URLs; Whatmore reads each product on connect and on refresh.
- **Real-time updates (push, optional):** when price or availability changes, send
  `PUT /v1/product` by `client_product_id` so it reflects immediately.

Because you reference products by *your own* `client_product_id` (the `id` from your product
API), there is no separate id-mapping to maintain. **Video and media are managed in the
dashboard — no upload API to build.**

## 3. How do I add or remove a product?

**Add:** put its product page **URL** in the dashboard — Whatmore pulls it from your product
API and stores it (new products are also picked up on refresh). There's no product-create or
bulk API for you to call. **Remove / take down:** drop it from your catalog (reflected on
refresh) or push `PUT /v1/product` with `product_status: "inactive"`.

## 4. Order tracking / "webhooks"

Order data is **pushed by you** to `POST /external-shop-order-tracking/private` on order
completion — see [Order Tracking](/integrations/order-tracking). Key semantics:

- **Idempotency:** orders are de-duplicated by `order_id`; a repeat is rejected
  (`Order Id already exists.`), never double-counted — so retries are safe.
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

The default rate limit is **1,000 requests per minute per store**, and can be increased per
client on request. Cache the long-lived access token rather than re-fetching it, and push
catalog updates **on change** rather than on a schedule. See
[Errors & Conventions](/integrations/errors) for status codes, response shapes, and limits.
