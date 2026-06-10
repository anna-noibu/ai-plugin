# Stack trace classification — WooCommerce / WordPress

## Merchant code (fixable)
- `/wp-content/themes/[active-theme]/` — merchant's active theme
- `/wp-content/themes/[child-theme]/` — child theme overrides
- Custom plugins built for this store in `/wp-content/plugins/[custom-plugin]/`

## Platform code (not fixable by merchant)
- `/wp-includes/` — WordPress core
- `/wp-content/plugins/woocommerce/` — WooCommerce core
- `/wp-admin/` — admin scripts

## Vendor code (share with vendor)
- `/wp-content/plugins/[third-party-plugin]/` — third-party plugins (Klaviyo for WooCommerce, etc.)
- External CDN scripts not matching the store domain

## Note on plugin vs theme errors
Errors in `/wp-content/plugins/` may be fixable if it's a merchant-owned or agency-built plugin, or vendor issues if it's a purchased/free third-party plugin. Use the plugin name to determine ownership — merchant-built plugins are usually named after the store or agency.

## HTTP errors without JS stack traces
403 errors are typically WordPress access control or a security plugin (Wordfence, etc.). 429 may be rate limiting. Flag as infrastructure.
