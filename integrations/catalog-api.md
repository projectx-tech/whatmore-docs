---
title: "Catalog API"
timestamp: false
mode: "wide"
---

Whatmore keeps its own copy of your product data (title, price, stock, image) so videos can
be made shoppable. You keep that copy in sync **two complementary ways — you use both**:

- **[Pull — initial load & refresh](#pull-initial-load-and-refresh):** connect your product
  API once in the dashboard; Whatmore reads your catalog for the initial sync and whenever
  you refresh.
- **[Push — event-based updates](#push-event-based-updates):** when a product changes
  (price, stock, quantity, images), push the change so tagged products stay accurate in real
  time — no waiting for a refresh.

Video upload and product tagging happen in the
[Whatmore dashboard](https://dashboard.whatmore.live/).

<Info>
**Video & media are managed in the dashboard — there is no upload API to integrate.** Your
only catalog job is keeping product *data* in sync (pull + push below). Less to build on
your side, faster go-live.
</Info>

## Pull: initial load and refresh

In [dashboard.whatmore.live](https://dashboard.whatmore.live/), choose your platform and
provide your **product API** — an endpoint that returns a single product's detail, plus any
auth it needs. Whatmore fetches a sample response and you **map your fields**:

| Whatmore field | Typical source field |
| -------------- | -------------------- |
| Product title | `name` / `title` |
| Product ID / SKU (`client_product_id`) | `id` |
| Price | `price` |
| Compare-at / MRP (`compare_price`) | `regular_price` |
| Product URL (`product_link`) | `permalink` |
| Product image (`thumbnail_image`) | `images[0].src` |
| Currency | set manually |

Whatmore uses this mapping to pull your catalog for the **initial sync** and on any
**refresh**. See your platform guide for the exact endpoint and credentials:
[WooCommerce](/integrations/platform-woocommerce) · [Custom / headless](/integrations/platform-custom) ·
[Magento](/integrations/platform-magento) · [SFCC](/integrations/platform-sfcc) ·
[BigCommerce](/integrations/platform-bigcommerce).

## Product identity

- **`client_product_id`** — *your* product identifier, and the key you use on every call and
  in [order tracking](/integrations/order-tracking). It is commonly the **product URL**,
  which keeps Whatmore aligned with the same URL used to tag products to videos (no separate
  mapping layer).
- **`product_link`** — the product's URL.
- Whatmore also assigns its own internal numeric `product_id`, returned in responses.

## Push: event-based updates

When individual products change, push the change so Whatmore's copy stays current without
waiting for the next pull/refresh. All calls use a
[bearer token](/integrations/authentication); the base URL is `https://api.whatmore.live`.

### Update price and stock

The most common event-based push — send it whenever price, inventory, or status changes,
keyed by your own `client_product_id`:

```http
PUT /v1/product
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "client_product_id": "9268",
  "price": "11.000",
  "currency": "KWD",
  "inventory": 0,
  "thumbnail_image": "https://cdn.yourstore.com/9268-v2.jpg",
  "product_status": "active"
}
```

Setting `inventory` to `0` marks the product out of stock.

### Add a product

To add a product outside the pull (e.g. a brand-new SKU that should go live immediately):

```http
POST /product
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "store_id": "<store_id>",
  "product_link": "https://www.yourstore.com/en/rose-hair-mist-75ml/p/9268",
  "client_product_id": "9268",
  "title": "Mid Night Rose Hair Mist 75ml",
  "price": "12.500",
  "compare_price": "15.000",
  "currency": "KWD",
  "thumbnail_image": "https://cdn.yourstore.com/9268.jpg",
  "product_metadata": { "sku": "RHM-75", "variant_id": "9268-1" },
  "product_status": "active"
}
```

`product_link` is required; `price` / `compare_price` are strings; `product_metadata` is a
free-form object for `sku`, `variant_id`, etc.

### Bulk import (large catalogs)

To push many products at once, hand Whatmore a list of product **URLs** and it ingests them:

```http
POST /product/upload/bulk
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "store_id": "<store_id>",
  "url_list": [
    "https://www.yourstore.com/en/rose-hair-mist-75ml/p/9268",
    "https://www.yourstore.com/en/velvet-lipstick/p/5521"
  ]
}
```

Use `POST /product-variants/upload/bulk` (same body) to bulk-import variants.

### Fetch a product

```http
GET /events/product/{client_product_id}
Authorization: Bearer <access_token>
```

```json
{
  "product_id": 41,
  "brand": "yourstore",
  "client_product_id": "9268",
  "product_link": "https://www.yourstore.com/en/rose-hair-mist-75ml/p/9268",
  "title": "Mid Night Rose Hair Mist 75ml",
  "price": "11.000",
  "compare_price": "15.000",
  "currency": "KWD",
  "thumbnail_image": "https://cdn.yourstore.com/9268.jpg",
  "product_status": "active"
}
```

To list all products for a store: `GET /brand/{store_id}/products`.
