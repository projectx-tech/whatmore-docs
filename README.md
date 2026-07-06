# Whatmore Integrations

Technical integration documentation for connecting a storefront to Whatmore's
shoppable-video and live-streaming platform **without** the native Shopify app — i.e.
Magento / Adobe Commerce, Salesforce Commerce Cloud, and custom / headless apps built on
the Whatmore App SDKs.

{% hint style="warning" %}
All HTTP/JSON in this documentation is **sample / illustrative only** — actual production
APIs may differ. Final shapes are confirmed jointly with each partner during integration.
Items marked **Open point** are still to be agreed between the two teams.
{% endhint %}

## The three integration tracks

Every non-Shopify integration is made up of the same three tracks. They can be worked on
in parallel.

| Track | What it covers | Owner of the surface |
| ----- | -------------- | -------------------- |
| **[Part 1 — App SDK](app-sdk.md)** | Rendering Whatmore's shoppable-video surfaces (carousel, PDP carousel, PDP floating card, celebrity pages) inside your app/site | Whatmore SDK, embedded by you |
| **[Part 2 — Authentication](authentication.md)** | The auth keys the two teams share, in both directions | Jointly agreed |
| **[Part 3 — API Integrations](api-integrations.md)** | The API + webhook contracts: Product Details API (you host) and webhooks (you post to us) | Split — see below |

## How the data flows

The important thing to understand up front: **traffic flows in both directions.**

```
        Whatmore  ── GET /api/products/{id} ──►  Your backend      (you host this API)
        (pulls product detail on demand)         (Product Details API)

        Your backend  ── POST webhook ──►  Whatmore                (we host these)
        (product updates, orders, cart)    (api.whatmore.ai/webhooks/...)
```

- **Whatmore → You:** Whatmore calls **your** Product Details API to fetch live price /
  stock / variant data for a product, keyed by `productId`. See
  [Product Details API](product-details-api.md).
- **You → Whatmore:** Your backend posts **webhooks** to Whatmore for product changes,
  completed orders (for attribution), and optionally cart events. See
  [Webhooks](webhooks.md).

## No bulk catalog upload required

A common first question is *"how do we sync our whole catalog to Whatmore?"* — you
generally **don't need to**. Whatmore's model is **pull-on-demand**:

- Products are tagged to videos using the **product URL**.
- `productId` is a **substring of that same URL**, so there is no separate mapping layer
  to maintain.
- Whatmore fetches full product detail on demand via your Product Details API, and stays
  fresh via the **product-updates webhook** — no polling, no nightly full-catalog export.

This is why there is no "catalog push" endpoint in this documentation. See the
[Backend Questions & Clarifications](faq.md) for the long-form answer.

## Start here

1. **[Getting Started](getting-started.md)** — access, keys, environments, checklist
2. **[Part 1 — App SDK](app-sdk.md)**
3. **[Part 2 — Authentication](authentication.md)**
4. **[Part 3 — API Integrations](api-integrations.md)** → [Product Details API](product-details-api.md) · [Webhooks](webhooks.md)
5. Your platform: **[Magento](magento.md)** · **[Salesforce Commerce Cloud](salesforce-commerce-cloud.md)**

{% hint style="info" %}
These pages are maintained in the Whatmore backend repository under `docs/integrations/`
and published to GitBook automatically (one-way). Do not edit them in the GitBook UI —
changes there are overwritten on the next sync.
{% endhint %}
