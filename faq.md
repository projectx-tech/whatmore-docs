# Backend Questions & Clarifications

Answers to the questions backend teams most commonly raise. Where an answer is not yet
finalised it is marked **Open point** and confirmed jointly during integration.

{% hint style="warning" %}
Contracts referenced here are **illustrative** — see the linked pages for the sample
shapes.
{% endhint %}

## 1. Product Details API

**When exactly is `GET /api/products/{productId}` called?** On demand — primarily when a
shopper interacts with a video/surface that has the product tagged — with short-lived
caching. **Not** on every video open as an uncached call, **not** during video upload, and
**not** as a nightly full-catalog crawl. See
[Product Details API → When is this called](product-details-api.md#when-is-this-api-called).

**Expected response payload?** See the [sample response](product-details-api.md#response)
— `productId`, `url`, `title`, `price {amount, currency}`, `inStock`, `quantityAvailable`,
`variants`, `images`. As detailed as possible.

> **Open point:** exact cache TTL and call timing.

## 2. Product Catalog Synchronization

**How should products be synchronized with Whatmore?** In this model there is **no bulk
catalog upload**. Products are tagged to videos by URL; `productId` is a substring of that
URL; Whatmore pulls detail on demand via your Product Details API and stays fresh via the
[product-updates webhook](webhooks.md#product-details-webhook). This resolves the
"initial upload / bulk push / incremental / full re-sync" questions — none of those steps
are required because there is no catalog copy to keep in sync. See
[README → No bulk catalog upload](README.md#no-bulk-catalog-upload-required).

## 3. Product API — bulk vs single

Single-product `GET /api/products/{productId}` is the baseline.

> **Open point:** whether a bulk / multi-id fetch is added depends on volume — to be
> agreed. See [Product Details API → Bulk fetch](product-details-api.md#bulk-fetch).

## 4. Webhooks

Strategy details — **retry policy, timeout, idempotency, failure recovery** — are listed
as [delivery semantics](webhooks.md#delivery-semantics).

> **Open point:** the concrete numbers (retry counts/backoff, timeout seconds, idempotency
> key, backfill mechanism) are agreed jointly.

## 5. API Contracts

Sample request/response for every integration API is on the
[Product Details API](product-details-api.md) and [Webhooks](webhooks.md) pages.

> **Open point:** a formal OpenAPI/Swagger spec can be produced once shapes are frozen.

## 6. Catalog Synchronization Process

Because the model is pull-on-demand (see #2), there is no initial sync / incremental /
full re-sync lifecycle to operate. Product accuracy is maintained entirely by the
product-updates webhook plus on-demand fetch.

## 7. Performance

> **Open points**, agreed jointly:
> - **API rate limits** on your Product Details API (Whatmore-side call volume)
> - **Webhook rate limits** Whatmore accepts
> - **Expected response times** for the Product Details API
> - Performance recommendations (caching, payload size, keep-alive)

## 8. Security

**API key rotation** — supported via **overlapping keys**: issue a new key, accept both old
and new during a grace window, then retire the old key — so credentials rotate without
service interruption. Applies in both directions (see
[Authentication](authentication.md)).

> **Open point:** rotation cadence and the exact grace-window mechanism are agreed jointly.
