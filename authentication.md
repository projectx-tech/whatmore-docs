# Part 2 — Authentication

Whatmore's integration APIs are called by **your** backend and authenticated with a
**bearer token**. You obtain the token from Whatmore using your `store_id`, then send it on
every API call.

## Get an access token

```http
GET /auth/access-token?store_id=<store_id>
```

Returns a bearer token (JWT) scoped to your store. Request it from your server, cache it,
and reuse it across calls.

## Call the APIs with the token

Send the token as a bearer credential, and include `store_id` as a query parameter on the
private tracking endpoints:

```http
POST /external-shop-order-tracking/private?store_id=<store_id>
Authorization: Bearer <access_token>
Content-Type: application/json
```

A missing or invalid token is rejected with **HTTP 401**.

## Base URLs & environments

| Environment | Base URL | Purpose |
| ----------- | -------- | ------- |
| Production  | provided at onboarding | Live traffic |
| Staging     | provided at onboarding | Integration testing |

Each environment issues its own `store_id` / token, so integration testing never touches
live data.

## Credentials you receive

| Credential | Used for |
| ---------- | -------- |
| `store_id` | Identifies your store; used to fetch a token and on every call |
| `brand` / Brand ID | Used by the [App SDK](app-sdk.md) to render your surfaces |
| Access token | Bearer auth on Catalog + Order Tracking APIs |

{% hint style="info" %}
Keep the `store_id` and access token **server-side**. The App SDK uses only the public
Brand ID — never embed the token in client or mobile app code.
{% endhint %}
