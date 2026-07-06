# Getting Started

Prerequisites and a checklist for integrating a non-Shopify storefront with Whatmore.

## 1. Get access

Your Whatmore contact provisions your store and issues:

- A **`store_id`** — identifies your store; used to get an access token and on every API call.
- A **Brand ID** — used by the [App SDK](app-sdk.md) to render your surfaces.
- Access to the Whatmore **dashboard** for managing videos and tagging products.

You obtain a **bearer access token** yourself from `GET /auth/access-token?store_id=<store_id>`
and send it on every API call. See [Authentication](authentication.md).

## 2. Environments

The API base URL is `https://api.whatmore.live`; the dashboard is at
`https://dashboard.whatmore.live`. Production and staging issue **separate `store_id`s and
tokens**, so integration testing never touches live data.

## 3. Integration checklist

- [ ] `store_id` + Brand ID received; access token obtained ([Auth](authentication.md))
- [ ] Catalog connected — in the dashboard ([pull + field mapping](catalog-api.md#connect-in-the-dashboard-recommended)) or via the [Catalog API](catalog-api.md#catalog-api-automation)
- [ ] Widget embedded from the dashboard-generated snippet (or [App SDK](app-sdk.md) for apps)
- [ ] [Order tracking](order-tracking.md) called on order completion
- [ ] Attribution verified in the dashboard

## Mental model

- **The dashboard does the work.** You connect a catalog and report orders; video hosting,
  tagging, campaigns, and analytics live in the dashboard.
- **Reference products by your own `client_product_id`** (commonly the product URL) — no
  separate id mapping.
- **Attribution is per order item** — the widget's video-view / add-to-cart signals are
  matched to line items on the [order-tracking](order-tracking.md) call.
