---
title: "Catalog API"
timestamp: false
mode: "wide"
---

Whatmore keeps its own copy of your product data (title, price, image, availability) so
videos can be made shoppable. You keep that copy in sync two ways:

- **[Pull — how products get in](#pull-initial-load-and-refresh) (primary):** connect your
  product API once in the dashboard and add your product URLs. Whatmore reads each product
  from your API and stores it — for the initial load, for every new product you add, and on
  refresh. **This is all most integrations need.**
- **[Push — real-time updates](#push-real-time-updates) (optional):** when price or
  availability changes, push the change via API so tagged products update immediately, without
  waiting for a refresh.

Video upload and product tagging happen in the
[Whatmore dashboard](https://dashboard.whatmore.live/).

<Info>
**Video & media are managed in the dashboard — there is no upload API to integrate.** Your
only catalog job is exposing a product API Whatmore can read (pull); the real-time push is
optional. Less to build on your side, faster go-live.
</Info>

## Pull: initial load and refresh

**You add products by their URL — there's no product-creation API to call.** In
[dashboard.whatmore.live](https://dashboard.whatmore.live/) you connect your **product API**
(an endpoint that returns a single product's detail, plus any auth it needs) and add your
product page URLs. For each URL, Whatmore extracts the product identifier, calls your product
API for that product, and stores the returned JSON. This is how the **initial load**, every
**new product**, and each **refresh** work.

### Expected product JSON

Your product API returns a **single product's** detail as JSON. The shape is flexible — you
map fields to Whatmore's in the dashboard, and nested keys are supported (expand the tree view
to pick them). A representative response:

```json
{
  "id": 9268,
  "name": "Mid Night Rose Hair Mist 75ml",
  "permalink": "https://www.yourstore.com/product/rose-hair-mist-75ml",
  "price": "12.500",
  "regular_price": "15.000",
  "sale_price": "12.500",
  "currency": "KWD",
  "description": "A warm rose & oud hair mist, 75ml.",
  "sku": "RHM-75",
  "stock_status": "instock",
  "stock_quantity": 24,
  "images": [
    { "src": "https://cdn.yourstore.com/9268.jpg" },
    { "src": "https://cdn.yourstore.com/9268-alt.jpg" }
  ]
}
```

You map those fields to Whatmore's:

| Whatmore field | Source field (example above) | Required |
| -------------- | ---------------------------- | -------- |
| `client_product_id` | `id` | **Yes** — stable & unique; the key you reuse in [order tracking](/integrations/order-tracking) |
| Product title | `name` / `title` | **Yes** |
| `price` | `price` / `sale_price` | **Yes** |
| `product_link` (URL) | `permalink` | **Yes** |
| `thumbnail_image` | `images[0].src` | **Yes** |
| `compare_price` (MRP) | `regular_price` | Recommended |
| Currency | `currency`, or set manually in the dashboard | Recommended |
| Description | `description` | Optional |
| Availability | `stock_status` / `stock_quantity` | Optional |

See your platform guide for the exact endpoint and credentials:
[WooCommerce](/integrations/platform-woocommerce) · [Custom / headless](/integrations/platform-custom) ·
[Magento](/integrations/platform-magento) · [SFCC](/integrations/platform-sfcc) ·
[BigCommerce](/integrations/platform-bigcommerce).

## Product identity

- **`client_product_id`** — *your* product identifier (the `id` from your product API), and
  the key you reuse on every push and in [order tracking](/integrations/order-tracking).
  Whatever value you map here **must be the exact same value you send in
  `order_items[].product_id`** — otherwise the item can't be attributed.
- **`product_link`** — the product's URL.
- Whatmore also assigns its own internal numeric `product_id` for its records.

## Push: real-time updates

New products flow in through the pull above. Push is **optional** — use it only when you want
a price or availability change to reflect **immediately**, without waiting for the next
refresh. It uses a [bearer token](/integrations/authentication); the base URL is
`https://api.whatmore.live`. Status codes and response conventions are on
[Errors & Conventions](/integrations/errors).

Send it whenever price, availability, or images change, keyed by your own `client_product_id`:

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
  "thumbnail_image": "https://cdn.yourstore.com/9268-v2.jpg",
  "product_status": "active"
}
```

- **Availability is controlled by `product_status`** — `"active"` (shoppable) or `"inactive"`
  (taken down). Send `"inactive"` to remove a product from your surfaces. Whether an inactive
  product is hidden or shown as out-of-stock is configured per store during onboarding.
- **Update-only.** `PUT /v1/product` updates an **existing** product matched by
  `client_product_id`; an unknown id is a no-op. New products are added via the pull (add the
  URL in the dashboard) — there's no create-via-API step.
