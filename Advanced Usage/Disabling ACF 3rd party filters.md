# Disabling ACF 3rd party filters

For consistency, we pass the data through ACFs 3rd party extension filters. If you need to disable this, you can do so using the following:

```php
<?php
/*
 * Thes two lines will deactivate ACFs 3rd party filters on data that is passed 
 * to or retreived from a custom database table. They will not disable the filters
 * on data being stored in the core meta tables.
 */
add_filter('acfcdt/settings/allow_acf_update_value_filters', '__return_false');
add_filter('acfcdt/settings/allow_acf_load_value_filters', '__return_false');
```