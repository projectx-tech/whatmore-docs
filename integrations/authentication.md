---
title: "Authentication"
timestamp: false
---

Whatmore's integration APIs are called by **your** backend and authenticated with a
**bearer token**. You obtain the token from Whatmore using your `store_id`, then send it on
every API call.

## Get an access token

```http
GET /auth/access-token?store_id=<store_id>
```

Returns a bearer token (JWT) scoped to your store. The token is **long-lived — it does not
expire** — so request it once from your server, cache it, and reuse it across all calls;
there's no refresh flow to build.

## Call the APIs with the token

Send the token as a bearer credential, and include `store_id` as a query parameter on the
private tracking endpoints:

```http
POST /external-shop-order-tracking/private?store_id=<store_id>
Authorization: Bearer <access_token>
Content-Type: application/json
```

A missing or invalid token is rejected with **HTTP 401**. See
[Errors & Conventions](/integrations/errors) for all status codes and response shapes.

## Base URL

The API base URL is **`https://api.whatmore.live`**. You manage your store, videos, and
integration settings in the dashboard at **[dashboard.whatmore.live](https://dashboard.whatmore.live/)**.

Each environment issues its own `store_id` / token, so integration testing never touches
live data.

## Credentials you receive

| Credential | Used for |
| ---------- | -------- |
| `store_id` | Identifies your store; used to fetch a token and on every call |
| `brand` / Brand ID | Used by the [App SDK](/integrations/app-sdk) to render your surfaces |
| Access token | Bearer auth on Catalog + Order Tracking APIs |

<Info>
Keep the `store_id` and access token **server-side**. The App SDK uses only the public
Brand ID — never embed the token in client or mobile app code.
</Info>
