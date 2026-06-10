# Stack trace classification — Magento / Adobe Commerce

## Merchant code (fixable)
- `pub/static/frontend/[Vendor]/[ThemeName]/` — merchant's custom theme
- `app/design/frontend/[Vendor]/[ThemeName]/` — theme source files
- Custom modules in `app/code/[Vendor]/` — merchant or agency-built modules

## Platform code (not fixable by merchant)
- `pub/static/frontend/Magento/` — Magento core frontend modules
- `lib/web/` — Magento core JS libraries
- `pub/static/adminhtml/` — admin panel code
- `vendor/magento/` — Magento core (Composer-installed)

## Vendor code (share with vendor)
- `vendor/[ThirdParty]/` — third-party extensions installed via Composer
- Any external CDN scripts not matching the store domain

## HTTP errors without JS stack traces
403 errors may be from Magento's access control or a WAF. 429 errors may indicate rate limiting at the server layer. Neither is typically a theme-level fix — flag as infrastructure.
