---
title: "Android SDK (Kotlin)"
mode: "wide"
---

`whatmore-storefront` — drop-in shoppable video for Android. One dependency, three ready-to-embed
templates (Reel, Feed, Carousel) that share the same configuration and listener. Available
for both **Views/Fragments** and **Jetpack Compose** hosts. It mirrors the
[iOS SDK](/integrations/sdk-ios) surface-for-surface.

## Requirements

| | Minimum |
| --- | --- |
| Android | API 24 (Android 7.0) |
| Kotlin | 1.9 |
| UI | AndroidX Views **or** Jetpack Compose |

## Install (Gradle)

```kotlin
// build.gradle.kts
dependencies {
    implementation("ai.whatmore:whatmore-storefront:1.0.0")
}
```

## Configure once

Build the config and your listener once, then reuse them across every surface:

```kotlin
import ai.whatmore.storefront.*

val config = WhatmoreStorefrontConfiguration(storeId = "STRNZFBL8TQ")
val whatmore = AppWhatmoreListener()   // your WhatmoreStorefrontListener
```

```kotlin
data class WhatmoreStorefrontConfiguration(
    val storeId: String,                                   // required — your Whatmore store id
    val statuses: List<String> = listOf("live", "upcoming"),
    val theme: WhatmoreStorefrontTheme = WhatmoreStorefrontTheme.Default,
    val productProvider: ProductProvider = MockProductProvider()
)
```

## Surfaces

### Reel — full-screen swipe (e.g. a "TV" tab)

```kotlin
// View
val reel = WhatmoreReelView(context).apply {
    configure(config, startIndex = 0)
    listener = whatmore
}

// Fragment
val fragment = WhatmoreReelFragment.newInstance(config, startIndex = 0).apply {
    listener = whatmore
}
```

```kotlin
// Jetpack Compose
WhatmoreReel(configuration = config, startIndex = 0, listener = whatmore)
```

### Feed — creator / celebrity page

```kotlin
// View / Fragment
WhatmoreFeedFragment.newInstance(config, celebrityName = celebrity.name).apply {
    listener = whatmore
}

// Compose
WhatmoreFeed(configuration = config, celebrityName = celebrity.name, listener = whatmore)
```

### Carousel — autoplaying rail

```kotlin
// View
WhatmoreCarouselView(context).apply {
    configure(config, title = "Trending Videos")   // title optional
    listener = whatmore
}

// Compose
WhatmoreCarousel(configuration = config, title = "Trending Videos", listener = whatmore)
```

Fully-visible cards autoplay muted; tapping opens the Reel at that video (the surface manages
its own full-screen presentation).

## Handle events — `WhatmoreStorefrontListener`

Implement once and attach to every surface. **Every method has a default no-op** — the SDK
never touches a cart, so you decide what each event does.

```kotlin
interface WhatmoreStorefrontListener {
    fun onTapAddToCart(product: WhatmoreProduct, event: WhatmoreEvent) {}
    fun onTapProduct(product: WhatmoreProduct, event: WhatmoreEvent) {}
    fun onTapViewAllProducts(event: WhatmoreEvent) {}
    fun onTapCTA(url: Uri, event: WhatmoreEvent) {}
    fun onToggleLike(isLiked: Boolean, event: WhatmoreEvent) {}
    fun onToggleSave(isSaved: Boolean, event: WhatmoreEvent) {}
    fun onTapShare(event: WhatmoreEvent) {}
}
```

| Method | Fires from | Meaning |
| ------ | ---------- | ------- |
| `onTapAddToCart` | Reel, Feed | "Add to cart" on a product tile |
| `onTapProduct` | Reel, Feed | product tile tapped (open PDP) |
| `onTapViewAllProducts` | Feed | "View All Products" tapped |
| `onTapCTA` | Reel | event call-to-action link tapped |
| `onToggleLike` | Reel, Feed | like toggled |
| `onToggleSave` | Feed | save / bookmark toggled |
| `onTapShare` | Reel, Feed | share tapped (SDK also presents a share sheet) |

```kotlin
class AppWhatmoreListener : WhatmoreStorefrontListener {
    override fun onTapAddToCart(product: WhatmoreProduct, event: WhatmoreEvent) {
        Cart.add(productId = product.id)
    }
    override fun onTapProduct(product: WhatmoreProduct, event: WhatmoreEvent) {
        Router.openPdp(product.id)
    }
}
```

For **attribution**, record `product.id` / `event.eventId` from these callbacks and include
them on your [Order Tracking](/integrations/order-tracking) call at checkout.

## Models

```kotlin
data class WhatmoreProduct(
    val id: String,
    val imageUrl: Uri?,
    val title: String,
    val price: BigDecimal,
    val comparePrice: BigDecimal?,     // struck-through / original price
    val currencyCode: String           // ISO 4217, e.g. "INR"
) {
    val priceText: String              // localized, e.g. "₹1,299"
    val comparePriceText: String?      // localized, null when no comparePrice
}

data class WhatmoreEvent(
    val eventId: Int,
    val brand: String?,
    val videoUrl: Uri?,
    val posterImageUrl: Uri?,
    val likeCount: Int,
    val shareCount: Int,
    val ctaUrl: Uri?,
    val pageId: Int
)
```

## Theme

```kotlin
data class WhatmoreStorefrontTheme(
    val accent: Color = Color.White,       // primary action tint
    val likeActive: Color = Color.Red      // liked-heart tint
) {
    companion object { val Default = WhatmoreStorefrontTheme() }
}
```

## Product data — `ProductProvider`

Products render in the bottom carousels via a provider. The SDK ships `MockProductProvider`
(the default) so all surfaces are demoable before product tagging is wired up. Supply your
own to render live catalog data — no UI changes required.

```kotlin
interface ProductProvider {
    suspend fun products(event: WhatmoreEvent): List<WhatmoreProduct>
}
```
