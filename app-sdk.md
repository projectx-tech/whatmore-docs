# Part 1 — App SDK

The Whatmore App SDK brings Whatmore's shoppable-video surfaces into your app or site. For
native mobile apps this is a **`react-native-sdk`** integration; web/headless storefronts
use the Web SDK. The surface list below is a **baseline** — scope and behaviour are refined
per partner.

{% hint style="warning" %}
Snippets are **illustrative only**. Final SDK package names, init signatures, and event
names are confirmed during integration.
{% endhint %}

## Baseline surfaces

- **Carousel** — a horizontal row of shoppable video cards
- **PDP Carousel** — carousel scoped to a product-detail page
- **PDP Floating Card** — a floating video card on the PDP
- **Celebrity Pages** — creator/celebrity-led video pages
- *(and similar surfaces as needed)*

## Session identity & attribution

This is the most important part of the SDK track, because it powers order attribution.

- The SDK issues and carries two identifiers:
  - `whatmore_user_id` — the Whatmore user identity
  - `whatmore_session_id` — the Whatmore viewing session
- These IDs **originate from the App SDK** and must be **carried through your checkout** so
  they can be included on the [Order Tracking Webhook](webhooks.md#order-tracking-webhook).
- Attribution is tracked **per order item, not per order** — a single order may contain
  items discovered through different videos/sessions (or none).
- Only items where a video was actually watched should carry these IDs.

```
  App SDK                         Your checkout                 Order webhook → Whatmore
  ───────                         ─────────────                 ────────────────────────
  video watched                   cart line item                order item:
  → whatmore_user_id      ─────►  remembers the IDs   ─────►      productId
  → whatmore_session_id           for tagged items                whatmore_user_id
                                                                  whatmore_session_id
```

## Web SDK (headless / non-mobile)

```html
<!-- Illustrative -->
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

```js
WhatmoreSDK.init({ brandId: 'YOUR_BRAND_ID' });
WhatmoreSDK.render('#whatmore-carousel', { type: 'carousel' });
WhatmoreSDK.on('addToCart', ({ productId, qty }) => { /* your cart */ });
```

## Open for discussion

The App SDK track is intentionally kept open and evolves with your requirements. Points to
align on:

- **Theming / design tokens** to match your app's look and feel
- **Placement and navigation behaviour** for each surface
- **Session identity handling** (`whatmore_user_id`, `whatmore_session_id`) and how it
  flows through to order attribution
- **Analytics events and depth of tracking**
