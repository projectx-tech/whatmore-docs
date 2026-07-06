# Part 1 — App SDK

The Whatmore App SDK renders Whatmore's shoppable-video surfaces inside your app or site.
For native mobile apps this is a **`react-native-sdk`** integration; web/headless
storefronts use the Web SDK. The surface list below is a baseline — scope and theming are
refined per partner.

## Baseline surfaces

- **Carousel** — a horizontal row of shoppable video cards
- **PDP Carousel** — carousel scoped to a product-detail page
- **PDP Floating Card** — a floating video card on the PDP
- **Celebrity Pages** — creator/celebrity-led video pages
- *(and similar surfaces as needed)*

## The SDK powers attribution

This is the most important part of the SDK track. As shoppers interact with video, the SDK
records two kinds of signal, each tied to a **product**:

- **video-view** — the product was shown/watched in a video
- **add-to-cart** — the product was added to cart from a video

Your app collects these signals and hands them to your order backend, which includes them
on the [Order Tracking](order-tracking.md) call at checkout. That is how Whatmore knows a
purchase came from a video.

Each signal is an object of the form:

```json
{ "product_id": "9268", "widget_info": { "…": "widget attribution payload" } }
```

- `product_id` — matches the product's [`client_product_id`](catalog-api.md) so line items
  can be reconciled.
- `widget_info` — the SDK-generated attribution payload identifying which surface/video
  drove the interaction.

```
  App SDK                         Your checkout                 Order tracking → Whatmore
  ───────                         ─────────────                 ─────────────────────────
  product watched in video  ──►   collect signals per   ──►     whatmore_video_view:  [ {product_id, widget_info}, … ]
  product added to cart           tagged product                whatmore_add_to_cart: [ {product_id, widget_info}, … ]
```

## Web SDK (headless / non-mobile)

```html
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

```js
WhatmoreSDK.init({ brandId: 'YOUR_BRAND_ID' });
WhatmoreSDK.render('#whatmore-carousel', { type: 'carousel' });
```

{% hint style="info" %}
Exact SDK package names, init signatures, and the surface list are confirmed during
integration; theming and placement are tailored to your app.
{% endhint %}
