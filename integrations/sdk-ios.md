---
title: "iOS SDK (Swift)"
mode: "wide"
---

`WhatmoreStorefront` — drop-in shoppable video for iOS. One Swift Package, three ready-to-embed
templates (Reel, Feed, Carousel) that share the same configuration and delegate. SwiftUI +
AVFoundation, no third-party dependencies; works in both UIKit and SwiftUI hosts.

## Requirements

| | Minimum |
| --- | --- |
| iOS | 17.0 |
| Swift tools | 5.9 |
| Xcode | 15+ |

## Install (Swift Package Manager)

```swift
dependencies: [
    .package(url: "<git-url>", from: "1.0.0")
],
targets: [
    .target(name: "YourApp", dependencies: [
        .product(name: "WhatmoreStorefront", package: "WhatmoreStorefront")
    ])
]
```

## Configure once

Build the config and your delegate once, reuse them across all surfaces:

```swift
import WhatmoreStorefront

let config = WhatmoreStorefrontConfiguration(storeID: "STRNZFBL8TQ")
let whatmore = AppWhatmoreHandler()   // your WhatmoreStorefrontDelegate
```

```swift
public struct WhatmoreStorefrontConfiguration {
    public init(
        storeID: String,                                   // required — your Whatmore store id
        statuses: [String] = ["live", "upcoming"],         // events to fetch
        theme: WhatmoreStorefrontTheme = .default,
        productProvider: ProductProviding = MockProductProvider()
    )
}
```

## Surfaces

### Reel — full-screen swipe (e.g. a "TV" tab)

```swift
// UIKit
let tv = WhatmoreReelViewController(configuration: config, startIndex: 0, delegate: whatmore)

// SwiftUI
WhatmoreReelView(configuration: config, startIndex: 0, delegate: whatmore)
    .ignoresSafeArea()
```

### Feed — creator / celebrity page

```swift
// SwiftUI
WhatmoreFeedView(configuration: config, celebrityName: celebrity.name, delegate: whatmore)

// UIKit
WhatmoreFeedViewController(configuration: config, celebrityName: celebrity.name, delegate: whatmore)
```

### Carousel — autoplaying rail (SwiftUI only)

```swift
WhatmoreCarouselView(configuration: config, title: "Trending Videos", delegate: whatmore)
```

Fully-visible cards autoplay muted; tapping opens the Reel at that video (the view manages
its own full-screen presentation).

## Handle events — `WhatmoreStorefrontDelegate`

Implement once and pass to every surface. **Every method is optional** (default no-ops) —
the SDK never touches a cart, so you decide what each event does.

```swift
public protocol WhatmoreStorefrontDelegate: AnyObject {
    func reelsDidTapAddToCart(_ product: WhatmoreProduct, in event: WhatmoreEvent)
    func reelsDidTapProduct(_ product: WhatmoreProduct, in event: WhatmoreEvent)
    func reelsDidTapViewAllProducts(in event: WhatmoreEvent)
    func reelsDidTapCTA(_ url: URL, in event: WhatmoreEvent)
    func reelsDidToggleLike(_ isLiked: Bool, in event: WhatmoreEvent)
    func reelsDidToggleSave(_ isSaved: Bool, in event: WhatmoreEvent)
    func reelsDidTapShare(_ event: WhatmoreEvent)
}
```

| Method | Fires from | Meaning |
| ------ | ---------- | ------- |
| `reelsDidTapAddToCart(_:in:)` | Reel, Feed | "Add to cart" on a product tile |
| `reelsDidTapProduct(_:in:)` | Reel, Feed | product tile tapped (open PDP) |
| `reelsDidTapViewAllProducts(in:)` | Feed | "View All Products" tapped |
| `reelsDidTapCTA(_:in:)` | Reel | event call-to-action link tapped |
| `reelsDidToggleLike(_:in:)` | Reel, Feed | like toggled |
| `reelsDidToggleSave(_:in:)` | Feed | save / bookmark toggled |
| `reelsDidTapShare(_:)` | Reel, Feed | share tapped (SDK also presents a share sheet) |

```swift
final class AppWhatmoreHandler: WhatmoreStorefrontDelegate {
    func reelsDidTapAddToCart(_ product: WhatmoreProduct, in event: WhatmoreEvent) {
        Cart.shared.add(productID: product.id)
    }
    func reelsDidTapProduct(_ product: WhatmoreProduct, in event: WhatmoreEvent) {
        Router.openPDP(product.id)
    }
}
```

For **attribution**, record the `product.id` / `event.eventID` from these callbacks and
include them on your [Order Tracking](/integrations/order-tracking) call at checkout.

## Models

```swift
public struct WhatmoreProduct: Identifiable, Hashable {
    public let id: String
    public let imageURL: URL?
    public let title: String
    public let price: Decimal
    public let comparePrice: Decimal?      // struck-through / original price
    public let currencyCode: String        // ISO 4217, e.g. "INR"
    // computed:
    public var priceText: String           // localized, e.g. "₹1,299"
    public var comparePriceText: String?   // localized, nil when no comparePrice
}

public struct WhatmoreEvent: Identifiable, Hashable {
    public let eventID: Int
    public let brand: String?
    public let videoURL: URL?
    public let posterImageURL: URL?
    public let likeCount: Int
    public let shareCount: Int
    public let ctaURL: URL?
    public let pageID: Int
}
```

## Theme

```swift
public struct WhatmoreStorefrontTheme {
    public init(accent: Color = .white, likeActive: Color = .red)
    public static let `default` = WhatmoreStorefrontTheme()
}
```

## Product data — `ProductProviding`

Products render in the bottom carousels via a provider. The SDK ships `MockProductProvider`
(the default) so all surfaces are demoable before product tagging is wired up. Supply your
own to render live catalog data — no UI changes required.

```swift
public protocol ProductProviding {
    func products(for event: WhatmoreEvent) async -> [WhatmoreProduct]
}
```
