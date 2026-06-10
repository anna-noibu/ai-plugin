# Stack trace classification — Shopify

## Merchant code (fixable)
Stack frames from these paths point to code the merchant owns and can edit:
- `cdn.shopify.com/s/files/` — theme assets (JS, CSS, images) served via Shopify CDN
- `cdn.shopify.com/s/trekkie/` — theme analytics snippets
- Store's own domain (e.g. `store.myshopify.com` or custom domain) for any inline scripts

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
- Any domain not matching the store or Shopify CDN

## Headless / custom frontend note
If this Shopify store runs a headless frontend (Next.js, Remix, Hydrogen, etc.), theme asset paths won't match the patterns above. Merchant code will be on the store's own CDN (Vercel, AWS CloudFront, etc.). In this case, classify by vendor script identification and ERR_TYPE — see `references/platforms/generic.md`.

## HTTP errors without JS stack traces
403, 429, and similar HTTP errors with no JavaScript stack trace are almost always Shopify's bot/abuse protection doing its job, not a code defect. Surface in the platform section, not fixable.
