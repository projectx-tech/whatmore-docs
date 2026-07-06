# Android SDK (Kotlin)

{% hint style="warning" %}
**Planned.** The native Android (Kotlin) SDK is on the roadmap. This page documents the
intended interface so Android teams can plan; it is confirmed and updated when the SDK
ships.
{% endhint %}

## What to expect

The Android SDK will mirror the [iOS SDK](sdk-ios.md) model so the integration is consistent
across platforms:

- The same **surfaces** — Reel (full-screen swipe), Feed (creator page), Carousel
  (autoplaying rail).
- A single **configuration** object (your Whatmore store id + theme), built once and reused.
- A single **listener / delegate** for events (add-to-cart, product tap, view-all-products,
  CTA, like/save/share) — the SDK stays commerce-agnostic; **your app owns the cart,
  checkout, and navigation**.
- The same **models** for product and event data.

Intended shape (subject to change):

```kotlin
// Illustrative — final Kotlin API confirmed at release
val config = WhatmoreReelsConfiguration(storeId = "STRNZFBL8TQ")

WhatmoreReelsView(context).apply {
    configure(config)
    listener = object : WhatmoreReelsListener {
        override fun onTapAddToCart(product: WhatmoreProduct, event: WhatmoreEvent) { /* your cart */ }
        override fun onTapProduct(product: WhatmoreProduct, event: WhatmoreEvent) { /* your PDP */ }
        // …like / save / share / CTA / viewAllProducts
    }
}
```

For attribution, capture the product / event from the listener callbacks and include them on
your [Order Tracking](order-tracking.md) call at checkout — same as iOS and React Native.

Until this ships, Android apps can integrate via the
[React Native SDK](sdk-react-native.md) where a React Native layer is available.
