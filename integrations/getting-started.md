---
title: "Getting Started"
---

Prerequisites and a checklist for integrating a non-Shopify storefront with Whatmore.

## 1. Get access

Your Whatmore contact provisions your store and issues:

- A **`store_id`** — identifies your store; used to get an access token and on every API call.
- A **Brand ID** — used by the [App SDK](/integrations/app-sdk) to render your surfaces.
- Access to the Whatmore **dashboard** for managing videos and tagging products.

You obtain a **bearer access token** yourself from `GET /auth/access-token?store_id=<store_id>`
and send it on every API call. See [Authentication](/integrations/authentication).

## 2. Environments

The API base URL is `https://api.whatmore.live`; the dashboard is at
[dashboard.whatmore.live](https://dashboard.whatmore.live/). Production and staging issue **separate `store_id`s and
tokens**, so integration testing never touches live data.

## 3. Integration checklist

- [ ] `store_id` + Brand ID received; access token obtained ([Auth](/integrations/authentication))
- [ ] Catalog [pull connected](/integrations/catalog-api#pull-initial-load-and-refresh) (product API + field mapping) **and** [push updates](/integrations/catalog-api#push-event-based-updates) wired (price / stock / images)
- [ ] Widget embedded from the dashboard-generated snippet (or [App SDK](/integrations/app-sdk) for apps)
- [ ] [Order tracking](/integrations/order-tracking) called on order completion
- [ ] Attribution verified in the dashboard

## Mental model

- **The dashboard does the work.** You connect a catalog and report orders; video hosting,
  tagging, campaigns, and analytics live in the dashboard.
- **Reference products by your own `client_product_id`** (commonly the product URL) — no
  separate id mapping.
- **Attribution is per order item** — the widget's video-view / add-to-cart signals are
  matched to line items on the [order-tracking](/integrations/order-tracking) call.
