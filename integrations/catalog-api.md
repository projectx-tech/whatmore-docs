---
title: "Catalog API"
---

Whatmore needs your product data (title, price, stock, image) to make videos shoppable.
There are two ways to provide it:

- **[Connect in the dashboard](#connect-in-the-dashboard-recommended)** *(recommended, no
  code)* — point Whatmore at your product API and map fields; Whatmore pulls the data for you.
- **[Catalog API](#catalog-api-automation)** *(automation)* — push products directly with the
  endpoints below.

Either way, Whatmore stores the data and keeps it fresh. Video upload and product tagging
happen in the dashboard.

## Connect in the dashboard (recommended)

In [dashboard.whatmore.live](https://dashboard.whatmore.live/), choose your platform and
provide your **product API** — an endpoint that returns a single product's detail, plus any
auth it needs. Whatmore fetches a sample response and you **map your fields** to Whatmore's:

| Whatmore field | Typical source field |
| -------------- | -------------------- |
| Product title | `name` / `title` |
| Product ID / SKU (`client_product_id`) | `id` |
| Price | `price` |
| Compare-at / MRP (`compare_price`) | `regular_price` |
| Product URL (`product_link`) | `permalink` |
| Product image (`thumbnail_image`) | `images[0].src` |
| Currency | set manually |

Whatmore then pulls product data using this mapping — no code to write. See your platform
guide for the exact endpoint and credentials:
[WooCommerce](/integrations/platform-woocommerce) · [Custom / headless](/integrations/platform-custom) ·
[Magento](/integrations/platform-magento) · [SFCC](/integrations/platform-sfcc) · [BigCommerce](/integrations/platform-bigcommerce).

## Product identity

- **`client_product_id`** — *your* product identifier, and the key you use on every call and
  in [order tracking](/integrations/order-tracking). It is commonly the **product URL**, which keeps
  Whatmore aligned with the same URL used to tag products to videos (no separate mapping
  layer).
- **`product_link`** — the product's URL.
- Whatmore also assigns its own internal numeric `product_id`, returned in responses.

<Info>
**Video & media are managed in the [Whatmore dashboard](https://dashboard.whatmore.live/) — there is no upload API to
integrate.** You upload, trim, and tag videos in the dashboard; your only catalog job is
making product data available. This is deliberate: less to build on your side, faster
go-live.
</Info>

## Catalog API (automation)

Prefer to push products yourself instead of the dashboard connect? Use these authenticated
endpoints. All calls use a [bearer token](/integrations/authentication); the base URL is
`https://api.whatmore.live`.

### Add a product

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
  "description": "…",
  "price": "12.500",
  "compare_price": "15.000",
  "currency": "KWD",
  "thumbnail_image": "https://cdn.yourstore.com/9268.jpg",
  "product_metadata": { "sku": "RHM-75", "variant_id": "9268-1" },
  "product_status": "active"
}
```

- `product_link` is required.
- `price` and `compare_price` are strings; `compare_price` is the strike-through / MRP.
- `product_metadata` is a free-form object for `sku`, `variant_id`, etc.

### Bulk import (large catalogs)

For a large catalog you don't build a payload per product — just hand Whatmore a list of
product **URLs** and it ingests them:

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

Whatmore fetches and stores each product from its URL. Use `POST /product-variants/upload/bulk`
(same body) to bulk-import variants.

### Update price and stock

Keep products fresh by updating them by your own `client_product_id` — no need to know
Whatmore's internal id:

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
  "product_status": "active"
}
```

Send this whenever price or inventory changes so tagged products stay accurate. Setting
`inventory` to `0` marks the product out of stock.

### Fetch a product

```http
GET /events/product/{client_product_id}
Authorization: Bearer <access_token>
```

Returns the stored product:

```json
{
  "product_id": 41,
  "brand": "yourstore",
  "client_product_id": "9268",
  "product_link": "https://www.yourstore.com/en/rose-hair-mist-75ml/p/9268",
  "title": "Mid Night Rose Hair Mist 75ml",
  "description": "…",
  "price": "11.000",
  "compare_price": "15.000",
  "currency": "KWD",
  "thumbnail_image": "https://cdn.yourstore.com/9268.jpg",
  "product_status": "active"
}
```

To list all products for a store: `GET /brand/{store_id}/products`.
