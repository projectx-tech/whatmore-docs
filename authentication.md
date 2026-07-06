# Part 2 — Authentication

Because traffic flows in **both directions**, both teams share auth keys. Whatmore calls
your Product Details API, and your backend calls Whatmore's webhook endpoints — each
direction is authenticated with a key issued by the side that **receives** the call.

{% hint style="warning" %}
Header names, schemes, and signing details below are **illustrative** and confirmed
jointly. The **Open points** at the bottom are not yet agreed.
{% endhint %}

## Key directory

| Direction | Purpose | Key provided by |
| --------- | ------- | --------------- |
| Whatmore → You | Calling your **Product Details API** | **You** (the merchant) |
| You → Whatmore | Posting to **webhooks** (product updates, orders, cart) | **Whatmore** |

## Whatmore → You (Product Details API)

Whatmore sends the key you issue on every request to your Product Details API:

```http
GET /api/products/{productId} HTTP/1.1
Host: api.yourstore.com
Authorization: Bearer <merchant_api_token>
```

## You → Whatmore (Webhooks)

Your backend sends the key Whatmore issues when posting webhooks:

```http
POST /webhooks/{brand}/orders HTTP/1.1
Host: api.whatmore.ai
Authorization: Bearer <whatmore_api_token>
X-Whatmore-Signature: <hmac_of_body>
```

## Webhook payload signing

To let each side verify that a webhook genuinely came from the other, payloads are signed
with a shared secret (e.g. HMAC-SHA256 over the raw request body), sent in a signature
header. The receiver recomputes the HMAC and rejects on mismatch.

> **Open point:** exact signature scheme (algorithm, header name, what is signed) to be
> agreed jointly.

## Environment separation

Test and production use **separate keys** and separate endpoints, so integration testing
never touches live data.

> **Open point:** confirm staging endpoints and whether a sandbox brand is issued.

## Open points to align on

- **Key rotation** — how to rotate credentials without service interruption (see
  [FAQ → Security](faq.md#8-security))
- **Signing / verification** of webhook payloads (scheme above)
- **Environment separation** — test vs. production keys and URLs
