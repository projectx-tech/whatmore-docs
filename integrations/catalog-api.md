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
product page URLs. For each URL, Whatmore extracts the product id, calls your product API for
that product, and stores the returned JSON. This is how the **initial load**, every **new
product**, and each **refresh** work.

### How the product id is read from a URL

Whatmore turns each product URL into a **product id** — the value it uses to query your API,
and which it stores as your [`client_product_id`](#product-identity).

- **Give us a regex to extract it exactly (recommended).** For a Nike-style URL
  `https://www.nike.com/t/air-max-90-shoes/CN8490-002`, a rule such as `([^/]+)$` (the last
  path segment) yields `CN8490-002`. A regex keeps extraction deterministic across all your
  URL shapes.
- **Default behaviour today:** with no rule set, Whatmore takes the trailing token of the URL
  — it splits on `-` and uses the last segment. That works when the URL ends in the id, but is
  brittle for other URL patterns, so providing a regex is strongly recommended.

### Expected product JSON

Your product API returns a **single product's** detail as JSON. The shape is flexible — you
map fields to Whatmore's in the dashboard, and nested keys are supported (expand the tree view
to pick them). A representative response:

```json
{
  "id": "CN8490-002",
  "name": "Nike Air Max 90",
  "permalink": "https://www.nike.com/t/air-max-90-shoes/CN8490-002",
  "price": "130.00",
  "regular_price": "150.00",
  "sale_price": "130.00",
  "currency": "USD",
  "description": "The Air Max 90 stays true to its running roots with the iconic Waffle sole.",
  "sku": "CN8490-002",
  "stock_status": "instock",
  "stock_quantity": 42,
  "images": [
    { "src": "https://static.nike.com/air-max-90/CN8490-002.jpg" },
    { "src": "https://static.nike.com/air-max-90/CN8490-002-alt.jpg" }
  ]
}
```

Every field, its type, and where it maps in Whatmore:

| Field | Type | Maps to → | Required | Notes |
| ----- | ---- | --------- | -------- | ----- |
| `id` | string or integer | `client_product_id` | **Yes** | Stable, unique product id. Normally equals the id in the URL; it's the key you reuse in [order tracking](/integrations/order-tracking). |
| `name` | string | Product title | **Yes** | Display title. |
| `permalink` | string (URL) | `product_link` | **Yes** | Canonical product page URL. |
| `price` | string | `price` | **Yes** | Current selling price as a decimal **string** (`"130.00"`), not a number. |
| `sale_price` | string | `price` | Optional | Use in place of `price` when the product is on sale. |
| `regular_price` | string | `compare_price` (MRP) | Recommended | Struck-through / original price. |
| `currency` | string (ISO 4217) | Currency | Recommended | e.g. `"USD"`. If your API omits it, set it once in the dashboard. |
| `images` | array of `{ "src": string }` | `thumbnail_image` | **Yes** | First entry (`images[0].src`) becomes the thumbnail. |
| `description` | string | Description | Optional | May contain HTML. |
| `sku` | string | `product_metadata.sku` | Optional | Stock-keeping unit. |
| `stock_status` | string (enum) | Availability | Optional | One of `"instock"`, `"outofstock"`, `"onbackorder"`. |
| `stock_quantity` | integer or `null` | Availability | Optional | Units in stock; `null` when inventory isn't tracked. |

Field names above are illustrative — your API can use any names, and you map them in the
dashboard (nested keys included). Only the **id**, **title**, **price**, **product URL**, and
**first image** are strictly required; the rest are recommended or optional.

See your platform guide for the exact endpoint and credentials:
[WooCommerce](/integrations/platform-woocommerce) · [Custom / headless](/integrations/platform-custom) ·
[Magento](/integrations/platform-magento) · [SFCC](/integrations/platform-sfcc) ·
[BigCommerce](/integrations/platform-bigcommerce).

## Product identity

- **`client_product_id`** — *your* product id, **extracted from the product URL** (see
  [above](#how-the-product-id-is-read-from-a-url)) and normally identical to the `id` your API
  returns. It is the key you reuse on every push and in
  [order tracking](/integrations/order-tracking): whatever value ends up here **must be the
  exact same value you send in `order_items[].product_id`**, or the item can't be attributed.
- **`product_link`** — the product's URL.
- Whatmore also assigns its own internal numeric `product_id` for its records.

## Push: real-time updates

New products flow in through the pull above. Push is **optional** — use it only when you want a
price or availability change to reflect **immediately**, without waiting for the next refresh.
It uses a [bearer token](/integrations/authentication); the base URL is
`https://api.whatmore.live`. Status codes and response conventions are on
[Errors & Conventions](/integrations/errors).

`POST /v2/product` **upserts** a product by `client_product_id` — creating it if it's new and
updating it otherwise. The body is the product in Whatmore's field names, mirroring the
[product JSON](#expected-product-json) above:

```http
POST /v2/product
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "client_product_id": "CN8490-002",
  "product_link": "https://www.nike.com/t/air-max-90-shoes/CN8490-002",
  "title": "Nike Air Max 90",
  "price": "130.00",
  "compare_price": "150.00",
  "currency": "USD",
  "description": "The Air Max 90 stays true to its running roots.",
  "thumbnail_image": "https://static.nike.com/air-max-90/CN8490-002.jpg",
  "sku": "CN8490-002",
  "stock_status": "instock",
  "stock_quantity": 42,
  "product_status": "active"
}
```

| Field | Type | Required | Notes |
| ----- | ---- | -------- | ----- |
| `client_product_id` | string | **Yes** | Identifies the product to upsert; must match your catalog + [order tracking](/integrations/order-tracking). |
| `product_link` | string (URL) | **Yes** on create | Canonical product URL. |
| `title` | string | **Yes** on create | Display title. |
| `price` | string | **Yes** on create | Selling price as a decimal string. |
| `compare_price` | string | Recommended | MRP / struck-through price. |
| `currency` | string (ISO 4217) | Recommended | e.g. `"USD"`. |
| `thumbnail_image` | string (URL) | **Yes** on create | Product image. |
| `description` | string | Optional | May contain HTML. |
| `sku` | string | Optional | Stored under `product_metadata`. |
| `stock_status` | string (enum) | Optional | `"instock"`, `"outofstock"`, or `"onbackorder"` — inventory availability. |
| `stock_quantity` | integer or `null` | Optional | Units in stock. |
| `product_status` | string (enum) | Optional | `"active"` (shoppable) or `"inactive"` (taken down on your surfaces). Whether an out-of-stock product is hidden or shown as OOS is configured per store at onboarding. |

On **update**, only the fields you send are changed — omit the rest. New products still flow
in automatically through the pull; use `POST /v2/product` only when you need an immediate
update.
