# Stack trace classification — BigCommerce

## Merchant code (fixable)
- Store theme assets served from the store's own domain or BigCommerce CDN under the merchant's store path
- Stencil theme JS files — typically at paths like `/assets/js/` or `/theme/`
- Custom scripts added via Script Manager that reference the store domain

## Platform code (not fixable by merchant)
- `cdn11.bigcommerce.com/` — BigCommerce platform CDN (core scripts, checkout)
- `login.bigcommerce.com/` — authentication infrastructure
- BigCommerce checkout scripts (not editable on standard plans)

## Vendor code (share with vendor)
- External scripts added via Script Manager from third-party domains
- Any CDN not matching BigCommerce or the store domain

## HTTP errors without JS stack traces
403 errors on BigCommerce are often from the platform's access control or DDoS protection. 429 may be API rate limits. Flag as infrastructure.
