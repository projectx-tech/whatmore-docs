# Part 3 — API Integrations

This track defines the API and webhook **contracts** between your backend and Whatmore.
There are two halves:

| Contract | Direction | Who hosts it | Page |
| -------- | --------- | ------------ | ---- |
| **Product Details API** | Whatmore → You | **You** | [Product Details API](product-details-api.md) |
| **Product / Order / Cart Webhooks** | You → Whatmore | **Whatmore** | [Webhooks](webhooks.md) |

{% hint style="warning" %}
All request/response examples are **sample / illustrative only** — production shapes may
differ and are confirmed jointly.
{% endhint %}

## At a glance

```
  ┌─────────────┐   GET /api/products/{id}     ┌──────────────────────┐
  │  Whatmore   │ ───────────────────────────► │  Your Product        │
  │             │ ◄─────────────────────────── │  Details API         │
  │             │        product JSON          └──────────────────────┘
  │             │
  │             │   POST product.updated       ┌──────────────────────┐
  │  Whatmore   │ ◄─────────────────────────── │  Your backend        │
  │  webhooks   │   POST order.completed        │  (posts webhooks)    │
  │             │ ◄─────────────────────────── │                      │
  │             │   POST cart.* (optional)      │                      │
  └─────────────┘ ◄─────────────────────────── └──────────────────────┘
```

Continue to:

* [Product Details API](product-details-api.md) — the API you expose to Whatmore
* [Webhooks](webhooks.md) — product updates, order attribution, and cart events you send us
