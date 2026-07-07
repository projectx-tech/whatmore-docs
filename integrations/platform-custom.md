---
title: "Custom / Headless"
timestamp: false
---

For a custom or headless storefront (any stack), you connect Whatmore in three steps:
provide a product API, embed the widget snippet, and report orders. This is the most direct
integration and uses the [Core APIs](/integrations/authentication) plus the dashboard.

## 1. Create your store in the dashboard

Sign in at [dashboard.whatmore.live](https://dashboard.whatmore.live/), choose **Shoppable
Videos**, and select **Custom** as the store type.

## 2. Connect your catalog

Give Whatmore a **product API** — an endpoint that returns a single product's detail — plus
any auth it needs:

```bash
curl https://yourstore.com/products/PRODUCT_IDENTIFIER \
  -u YOUR_CONSUMER_KEY:YOUR_CONSUMER_SECRET
```

Whatmore fetches a sample response and you **map fields** to Whatmore's in the dashboard
(see [Catalog API → Connect in the dashboard](/integrations/catalog-api#pull-initial-load-and-refresh)):

| Whatmore field | Your API field (example) |
| -------------- | ------------------------ |
| Product title | `name` |
| Product ID / SKU | `id` |
| Price | `price` |
| Compare-at / MRP | `regular_price` |
| Product URL | `permalink` |
| Product image | `images[0].src` |
| Currency | set manually |

Whatmore then pulls product data itself. *(Prefer to push? Use the
[Catalog API](/integrations/catalog-api#push-real-time-updates) instead.)*

## 3. Embed the widget

In the dashboard, open a surface (e.g. **Homepage → Homepage Video Carousel → Setup**),
choose a template, review the live preview, and **copy the generated snippet**. Paste it
into your site where you want the surface. For mobile apps, use the [App SDK](/integrations/app-sdk)
instead of a web snippet.

## 4. Authentication

You receive a **`store_id`** and get a bearer **token** from
`GET /auth/access-token?store_id=<store_id>`. See [Authentication](/integrations/authentication).

## 5. Order tracking

Add the order-tracking call to your order-confirmation page. The widget stores video-view /
add-to-cart signals in `localStorage`, so the
[ready-to-use snippet](/integrations/order-tracking#ready-to-use-snippet-web) picks them up:

```javascript
async function sendOrderTrackingRequest({ orderId, orderItems, storeId, token }) {
  const url = `https://api.whatmore.live/external-shop-order-tracking/private?store_id=${encodeURIComponent(storeId)}`;
  const payload = {
    order_id: orderId,
    order_items: orderItems,
    whatmore_video_view: localStorage._whatmore_viewed_products || "[]",
    whatmore_add_to_cart: localStorage._whatmore_add_to_cart_products || "[]",
  };
  const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json", Authorization: `Bearer ${token}` },
    body: JSON.stringify(payload),
  });
  if (!res.ok) throw new Error(`Order tracking failed: ${res.status} ${res.statusText}`);
  return res;
}
```

<Warning>
The `product_id` in `order_items[]` must be the **same identifier your product API returns**
for that product, so items can be matched to the video signals. See
[Order Tracking](/integrations/order-tracking).
</Warning>

## Verify

- Products appear in your Whatmore dashboard catalog
- Widget renders from the pasted snippet
- Order tracking fires on confirmation and attribution shows in the dashboard
