# Part 1 — App SDK

The Whatmore App SDK renders Whatmore's shoppable-video surfaces **natively** inside your
app. It is **commerce-agnostic**: the SDK renders the experience and emits events through a
single delegate / callbacks object — **your app owns the cart, checkout, navigation, and
analytics**. This keeps the integration small and is why most of the work stays in the
[Whatmore dashboard](README.md), not in your codebase.

{% hint style="info" %}
These pages document the **current** SDK interfaces. The native SDKs are being rebuilt
(Swift / Kotlin / React Native); treat the signatures as a reference — they may evolve, and
the docs will be updated to match.
{% endhint %}

## Surfaces

The SDK ships ready-to-embed templates that share the same store feed, player, products,
and event model:

| Surface | What it is | Typical placement |
| ------- | ---------- | ----------------- |
| **Reel** | Full-screen vertical swipe (Instagram-Reels style) | a "TV" / "Videos" tab |
| **Feed** | Scrolling post feed | a creator / celebrity page |
| **Carousel** | Autoplaying horizontal rail that opens the Reel | home, category, any screen |

## Platforms

| Platform | Package | Status |
| -------- | ------- | ------ |
| **[iOS (Swift)](sdk-ios.md)** | `WhatmoreReels` (Swift Package) | Available |
| **[React Native](sdk-react-native.md)** | `@whatmore-repo/whatmore-reactnative-sdk` | Available |
| **[Android (Kotlin)](sdk-android.md)** | — | Planned |

## The integration model

Every surface takes the **same configuration** (your Whatmore store id + theme) and reports
user actions through the **same event hooks**. You wire those hooks once and reuse them
across surfaces.

```
  Whatmore SDK (renders)            Your app (owns commerce)
  ──────────────────────            ────────────────────────
  video surface                     ── add to cart  ─────►  your cart
  product tile / CTA / like    ──►  ── open product ─────►  your PDP / router
  (emits events)                    ── on purchase  ─────►  Order Tracking → Whatmore
```

- **Configure once** — a store id, optional theme, and a product provider.
- **Handle events** — add-to-cart, product tap, CTA, like/save/share. The SDK never touches
  a cart, so you decide what each event does.
- **Attribute purchases** — capture the products surfaced by the SDK and include them on the
  [Order Tracking](order-tracking.md) call at checkout, so Whatmore can credit the video.

Pick your platform to see the exact interface: **[iOS](sdk-ios.md)** ·
**[React Native](sdk-react-native.md)** · **[Android](sdk-android.md)**.
