# Catalog API

You sync your products **to** Whatmore so videos can be made shoppable. You add products
once, then keep price and stock current with lightweight update calls. All calls use a
[bearer token](authentication.md).

## Product identity

- **`client_product_id`** — *your* product identifier, and the key you use on every call.
  It is commonly the **product URL**, which keeps Whatmore aligned with the same URL used
  to tag products to videos (no separate mapping layer).
- **`product_link`** — the product's URL.
- Whatmore also assigns its own internal numeric `product_id`, returned in responses.

## Add a product

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

## Bulk import (large catalogs)

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
(same body) to bulk-import variants. This is the fastest path to a live catalog.

{% hint style="info" %}
**Video & media are managed in the Whatmore dashboard — there is no upload API to
integrate.** You upload, trim, and tag videos in the dashboard; your only catalog job is
making product data available (single, update, or bulk-by-URL above). This is deliberate:
less to build on your side, faster go-live.
{% endhint %}

## Update price and stock

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

## Fetch a product

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

{% hint style="info" %}
Variant- and SKU-level data is carried in `product_metadata`. If you need bulk product
import for a large catalog, that is available — the exact bulk format is confirmed during
onboarding.
{% endhint %}
