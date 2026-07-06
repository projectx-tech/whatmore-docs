# Product Details API

**You host this API. Whatmore calls it.** It returns full, live detail for a single product
by `productId`, so tagged products stay accurate (price, stock, variants) at the moment a
shopper views a video.

{% hint style="warning" %}
The request/response below is a **sample for example purposes only** — production may
differ. Final shape confirmed jointly.
{% endhint %}

## `productId` comes from the product URL

`productId` should **preferably be a substring of the product URL**, because the *same URL
is used to tag products to videos* — keeping them aligned avoids a separate mapping layer.

```
https://www.yourstore.com/en/women/mid-night-rose-hair-mist-75ml/p/cid/9268/
                                                                          ────
                                                                          productId = 9268
```

## Request

```http
GET /api/products/{productId}
Authorization: Bearer <merchant_api_token>
```

## Response

Should be **as detailed as possible**: in-stock / out-of-stock state, available quantity,
price (and currency), variants, images, etc.

```json
{
  "productId": "9268",
  "url": "https://www.yourstore.com/en/women/mid-night-rose-hair-mist-75ml/p/cid/9268/",
  "title": "Mid Night Rose Hair Mist 75ml",
  "price": { "amount": 12.500, "currency": "KWD" },
  "inStock": true,
  "quantityAvailable": 42,
  "variants": [],
  "images": []
}
```

## When is this API called?

{% hint style="info" %}
This is one of the most-asked clarifications — see also
[FAQ → Product Details API](faq.md#1-product-details-api).
{% endhint %}

Whatmore fetches product detail **on demand** — primarily when a shopper interacts with a
video/surface that has the product tagged — with short-lived caching, and is kept fresh by
the [product-updates webhook](webhooks.md#product-details-webhook). It is **not** called on
every keystroke, nor as a nightly full-catalog crawl.

> **Open point:** exact call timing, cache TTL, and whether Whatmore batches lookups are
> confirmed jointly.

## Bulk fetch

> **Open point:** whether the API must support single-product retrieval only, or also a
> bulk/multi-id fetch, is to be agreed based on volume. Single-product `GET` is the
> baseline.
