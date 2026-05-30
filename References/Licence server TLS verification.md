# Licence server TLS verification

The plugin calls the Hookturn licence server (`https://hookturn.io/`) over HTTPS during licence activation, deactivation, and periodic validation. By default, these requests verify the server's TLS certificate — that's the standard, secure WordPress behaviour, and it matches what the bundled Easy Digital Downloads updater does for plugin update checks.

On rare occasions, a customer site's PHP process cannot complete TLS verification — typically because of an outdated CA certificate bundle, a corporate proxy that performs TLS interception, or a Cloudflare Flexible SSL configuration. In these cases the licence call fails with a "Could not connect" or similar error even though the licence itself is valid.

The proper fix is server-side: update OpenSSL and your CA bundle, reconfigure the proxy, or correct the Cloudflare SSL setting. If those aren't options in the short term, the plugin exposes the standard EDD `edd_sl_api_request_verify_ssl` filter so you can opt out of verification.

## Disabling TLS verification

```php
<?php
add_filter( 'edd_sl_api_request_verify_ssl', '__return_false' );
```

Drop that in your `wp-config.php`, in a custom configuration plugin, or in an
[MU plugin](https://wordpress.org/support/article/must-use-plugins/). See
[Configuration via code](Configuration%20via%20code.md) for guidance on where to host
small customisation snippets like this.

The filter applies to both the plugin's licence service and the bundled EDD plugin updater — a single filter call covers all licence-related HTTPS requests the plugin makes.

## Re-enabling verification once you've fixed the underlying issue

Remove the filter call from wherever you added it. There is no other state to clean up — the filter defaults to `true`, so the plugin returns to verifying TLS certificates as soon as the override is gone.

## Why we recommend keeping verification enabled

The licence server returns the activation status that the plugin stores locally. If verification is disabled, a network attacker positioned between your site and the licence server can present a forged certificate and serve a fabricated "licence valid" or "licence invalid" response. That can either bypass licensing checks on a site that should not be running the plugin, or remotely disable a legitimately licensed site.

Disable verification only when you've confirmed the certificate problem is on your side and you cannot resolve it in the short term, and re-enable it as soon as the underlying issue is fixed.
