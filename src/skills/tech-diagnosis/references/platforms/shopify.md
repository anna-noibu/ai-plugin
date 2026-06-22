# Stack trace classification — Shopify

## Merchant code (fixable)
Stack frames from these paths point to code the merchant owns and can edit:
- `cdn.shopify.com/s/files/` — theme assets (JS, CSS, images) served via Shopify CDN
- `cdn.shopify.com/s/trekkie/` — theme analytics snippets
- Store's own domain (e.g. `store.myshopify.com` or custom domain) — **only for `.js` file URLs**, not inline page-URL frames. Inline frames (where `sourceUrl` is a page URL) are commonly injected by third-party tools (New Relic, GTM, app extensions) and should not be treated as merchant-owned.
- `cdn.shopify.com/s/files/` — theme assets, but note this path also serves third-party app scripts. If the filename doesn't follow standard theme naming (e.g. has a vendor-looking prefix like `tyz-`, `bc-`, `rebuy-`) classify as vendor, not merchant.

## Platform code (not fixable by merchant)
- `cdn.shopify.com/shopifycloud/` — Shopify's own platform scripts
- `cdn.shopify.com/s/shopifycloud/` — same
- `shop.app/` — Shop Pay / Shop app infrastructure
- `checkout.shopify.com/` — Shopify checkout (not editable on standard plans)
- `monorail-edge.shopifysvc.com/` — Shopify analytics infrastructure

## Vendor code (share with vendor)
Third-party app scripts embedded in the theme. Common examples:
- `static.klaviyo.com/` — Klaviyo
- `heyethos.com/` — Hey Ethos (loyalty)
- `static-tracking.klaviyo.com/`
- `yotpo.com/`
- `loox.io/`
- `cdn.shopify.com/extensions/` — Shopify app extensions (Checkout UI extensions, Web Pixel extensions, monitoring apps). Always third-party vendor code.
- Any domain not matching the store or Shopify CDN

## Headless / custom frontend note
If this Shopify store runs a headless frontend (Next.js, Remix, Hydrogen, etc.), theme asset paths won't match the patterns above. Merchant code will be on the store's own CDN (Vercel, AWS CloudFront, etc.). In this case, classify by vendor script identification and ERR_TYPE — see `references/platforms/generic.md`.

## HTTP errors
- Any 4XX HTTP error is platform/vendor regardless of whether a JS stack trace is present. Do not classify 4XX errors as fixable.
- 403, 429, and similar are almost always Shopify's bot/abuse protection doing its job, not a code defect. Surface in the platform section.

## "Script error"
An error whose message/type is `Script error` is a cross-origin error with no usable detail — never fixable. If the origin can be identified as a known third-party domain, surface it in the vendor section. Otherwise, drop it.
