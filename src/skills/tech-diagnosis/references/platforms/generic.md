# Stack trace classification — Generic / Headless / Custom

Use this file when the platform isn't Shopify, Magento, WooCommerce, or BigCommerce — or when the store runs a headless or custom frontend where platform-specific path patterns don't apply.

## Classification approach

Without reliable path patterns, classify by what you can identify:

### Known vendor scripts (share with vendor)
Regardless of platform, these are always third-party and not fixable by the merchant:
- `static.klaviyo.com/` — Klaviyo
- `googletagmanager.com/` — Google Tag Manager
- `connect.facebook.net/` — Meta Pixel
- `static.ads-twitter.com/` — Twitter/X Ads
- `analytics.tiktok.com/` — TikTok Pixel
- `cdn.shopify.com/shopifycloud/` — Shopify platform (even in headless)
- Any recognisable analytics, marketing, or payments CDN

### ERR_TYPE as primary classifier
When stack trace origin is unclear:
- **JS error types** (TypeError, ReferenceError, RangeError, etc.) — likely code-level, treat as fixable candidate
- **HTTP error types** (403, 429, 404, 500) with no JS stack trace — likely infrastructure or platform, treat as not fixable by merchant code

### Frames on the store's own domain
If a frame points to the store's own domain or a CDN that clearly serves the merchant's app (e.g. Vercel deployment URLs for a known merchant project), treat as merchant code and fixable.

### When uncertain
If you can't confidently classify an error, surface it as a fixable candidate but note in the card description that the origin is unclear and the fix may require developer investigation.

## Editing this file
Add platform-specific patterns here as you identify them. This file is meant to be extended.
