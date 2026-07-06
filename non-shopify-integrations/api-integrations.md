# Part 3 — API Integrations

Two APIs make up the backend integration. Both are **called by your backend** and
authenticated with a [bearer token](authentication.md) — there is no Whatmore-hosted
service you need to expose an endpoint for.

| API | You call it to… | Page |
| --- | --------------- | ---- |
| **Catalog API** | Sync your products to Whatmore and keep price/stock current | [Catalog API](catalog-api.md) |
| **Order Tracking** | Report completed orders (with SDK signals) for attribution | [Order Tracking](order-tracking.md) |

## At a glance

```
  ┌──────────────┐   POST /product          ┌──────────────────────┐
  │ Your backend │   PUT  /v1/product        │  Whatmore            │
  │              │ ─────────────────────────►│  Product catalog     │
  │              │                           └──────────────────────┘
  │              │
  │              │   POST /external-shop-    ┌──────────────────────┐
  │ Your backend │        order-tracking     │  Whatmore            │
  │              │ ─────────────────────────►│  Attribution         │
  └──────────────┘                           └──────────────────────┘
```

Continue to:

* [Catalog API](catalog-api.md) — add, update, and fetch products
* [Order Tracking](order-tracking.md) — report purchases and attribute them to videos
