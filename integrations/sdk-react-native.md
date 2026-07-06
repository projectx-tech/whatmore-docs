# React Native SDK

`@whatmore-repo/whatmore-reactnative-sdk` — shoppable video for React Native apps. Renders
**native** views (not a WebView); your app owns the cart and navigation via a callbacks
object.

{% hint style="info" %}
Current interface — the SDK is being rebuilt and ships without TypeScript types today, so
the signatures below are the reference shape and may change.
{% endhint %}

## Install

```bash
npm install @whatmore-repo/whatmore-reactnative-sdk
```

Native peer dependencies to link in your app: `react-native-video`, `react-native-svg`.

## Render a surface — `WhatmoreBase`

The SDK exposes a single component. Configuration is done via props (there is no separate
init/Provider call).

```tsx
import { WhatmoreBase } from '@whatmore-repo/whatmore-reactnative-sdk';

<WhatmoreBase
  shopId="STRNZFBL8TQ"              // your Whatmore store id
  domainContext="whatmore"
  template="template-swipe-a"
  modalUsed={true}
  activateMock={false}
  commandsObject={{
    navigate: ({ options }) => Router.openProduct(options.productHandle),
    addToCart: (variantId, quantity) => Cart.add(variantId, quantity),
  }}
/>
```

## Props

| Prop | Type | Notes |
| ---- | ---- | ----- |
| `shopId` | `string` | Your Whatmore store id |
| `domainContext` | `'appbrew' \| 'appmaker' \| 'plobalapps' \| 'shopify' \| 'whatmore'` | Host/partner context |
| `template` | `string` | Surface template, e.g. `'template-swipe-a'` |
| `modalUsed` | `boolean` | Modal vs inline swipe presentation |
| `activateMock` | `boolean` | Mock mode for demos (short-circuits commerce callbacks) |
| `commandsObject` | `object` | Host callbacks — see below |

## Commerce callbacks — `commandsObject`

The SDK calls these when the shopper acts; your app performs the commerce action.

```ts
type WhatmoreCommandsObject = {
  navigate: (navigateObject: {
    kind: 'screen';
    screenId: 'product';
    options: { productHandle: string };
  }) => void;                                    // product tapped → open your PDP
  addToCart: (variantId: string, quantity: number) => void;  // add-to-cart tapped
};
```

| Callback | Fired when | Payload |
| -------- | ---------- | ------- |
| `navigate` | a product is tapped | `{ kind: 'screen', screenId: 'product', options: { productHandle } }` |
| `addToCart` | add-to-cart is tapped | `(variantId, quantity)` |

For **attribution**, record the product handle / variant from these callbacks and include
them on your [Order Tracking](order-tracking.md) call at checkout.

{% hint style="info" %}
The SDK fetches its own event feed from Whatmore using `shopId`; you do not pass video data
in. Product detail is resolved from your store. Exact `template` values and additional
`domainContext` wiring are confirmed at onboarding.
{% endhint %}
