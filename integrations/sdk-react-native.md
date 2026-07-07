---
title: "React Native SDK"
timestamp: false
mode: "wide"
---

`@whatmore-repo/whatmore-storefront` — drop-in shoppable video for React Native. Renders
**native** views (not a WebView) and ships three ready-to-embed surfaces (Reel, Feed,
Carousel) that share the same configuration and event handlers. It mirrors the
[iOS SDK](/integrations/sdk-ios) surface-for-surface.

## Install

```bash
npm install @whatmore-repo/whatmore-storefront
```

Native peer dependencies to link in your app: `react-native-video`, `react-native-svg`.

## Configure once

Wrap your app in `WhatmoreStorefrontProvider` with a configuration and your event handlers; place the
surfaces anywhere below it.

```tsx
import {
  WhatmoreStorefrontProvider, WhatmoreReel, WhatmoreFeed, WhatmoreCarousel,
  type WhatmoreStorefrontConfiguration, type WhatmoreStorefrontHandlers,
} from '@whatmore-repo/whatmore-storefront';

const config: WhatmoreStorefrontConfiguration = {
  storeId: 'STRNZFBL8TQ',                         // required — your Whatmore store id
  statuses: ['live', 'upcoming'],
  theme: { accent: '#FFFFFF', likeActive: '#FF3B30' },
  productProvider: async (event) => [ /* WhatmoreProduct[] */ ],
};

const handlers: WhatmoreStorefrontHandlers = {
  onTapAddToCart: (product, event) => Cart.add(product.id),
  onTapProduct:   (product, event) => Router.openPDP(product.id),
};

<WhatmoreStorefrontProvider configuration={config} handlers={handlers}>
  {/* app */}
</WhatmoreStorefrontProvider>
```

```ts
interface WhatmoreStorefrontConfiguration {
  storeId: string;                                         // required
  statuses?: string[];                                     // default ['live', 'upcoming']
  theme?: WhatmoreStorefrontTheme;
  productProvider?: (event: WhatmoreEvent) => Promise<WhatmoreProduct[]>;
}

interface WhatmoreStorefrontTheme {
  accent?: string;       // hex, default '#FFFFFF'
  likeActive?: string;   // hex, default '#FF3B30'
}
```

## Surfaces

### Reel — full-screen swipe (e.g. a "TV" tab)

```tsx
<WhatmoreReel startIndex={0} />
```

### Feed — creator / celebrity page

```tsx
<WhatmoreFeed celebrityName={celebrity.name} />
```

### Carousel — autoplaying rail

```tsx
<WhatmoreCarousel title="Trending Videos" />   {/* title optional */}
```

Fully-visible cards autoplay muted; tapping opens the Reel at that video (the surface manages
its own full-screen presentation). Any surface can also take its own `handlers` prop to
override the provider's for that instance.

## Handle events — `WhatmoreStorefrontHandlers`

Wire these once on the provider. **Every handler is optional** — the SDK never touches a
cart, so you decide what each event does.

```ts
interface WhatmoreStorefrontHandlers {
  onTapAddToCart?:      (product: WhatmoreProduct, event: WhatmoreEvent) => void;
  onTapProduct?:        (product: WhatmoreProduct, event: WhatmoreEvent) => void;
  onTapViewAllProducts?: (event: WhatmoreEvent) => void;
  onTapCTA?:            (url: string, event: WhatmoreEvent) => void;
  onToggleLike?:        (isLiked: boolean, event: WhatmoreEvent) => void;
  onToggleSave?:        (isSaved: boolean, event: WhatmoreEvent) => void;
  onTapShare?:          (event: WhatmoreEvent) => void;
}
```

| Handler | Fires from | Meaning |
| ------- | ---------- | ------- |
| `onTapAddToCart` | Reel, Feed | "Add to cart" on a product tile |
| `onTapProduct` | Reel, Feed | product tile tapped (open PDP) |
| `onTapViewAllProducts` | Feed | "View All Products" tapped |
| `onTapCTA` | Reel | event call-to-action link tapped |
| `onToggleLike` | Reel, Feed | like toggled |
| `onToggleSave` | Feed | save / bookmark toggled |
| `onTapShare` | Reel, Feed | share tapped (SDK also presents a share sheet) |

For **attribution**, record `product.id` / `event.eventID` from these handlers and include
them on your [Order Tracking](/integrations/order-tracking) call at checkout.

## Models

```ts
interface WhatmoreProduct {
  id: string;
  imageURL?: string;
  title: string;
  price: number;
  comparePrice?: number;     // struck-through / original price
  currencyCode: string;      // ISO 4217, e.g. "INR"
  priceText: string;         // localized, e.g. "₹1,299"
  comparePriceText?: string; // localized, undefined when no comparePrice
}

interface WhatmoreEvent {
  eventID: number;
  brand?: string;
  videoURL?: string;
  posterImageURL?: string;
  likeCount: number;
  shareCount: number;
  ctaURL?: string;
  pageID: number;
}
```

## Product data — `productProvider`

Products render in the bottom carousels via `configuration.productProvider`. The SDK ships a
mock provider (the default) so all surfaces are demoable before product tagging is wired up.
Supply your own to render live catalog data — no UI changes required.

```ts
productProvider: (event: WhatmoreEvent) => Promise<WhatmoreProduct[]>
```
