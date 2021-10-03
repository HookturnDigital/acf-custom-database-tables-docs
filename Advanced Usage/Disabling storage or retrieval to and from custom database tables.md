# Disabling storage or retrieval to and from custom database tables

A collection of action hooks can be fired to disable/enable data intercept handlers when necessary. Once disabled, data
will not be intercepted until re-enabled.

You may fire these actions once for a global effect should you need to.

## How to disable data retrieval from custom database tables

```php
<?php
// Disables the get field intercept
do_action('acfcdt/disable_get_field_intercept');

// Data will only come from core meta tables
get_field('my-field');

// Re-enable the get field intercept
do_action('acfcdt/enable_get_field_intercept');
```

## How to disable data storage to custom database tables

```php
<?php
// Disable the update field intercept
do_action('acfcdt/disable_update_field_intercept');

// Data will only be inserted/updated in core meta tables
update_field('my-field', 'some value');

// Re-enable the update field intercept
do_action('acfcdt/enable_update_field_intercept');
```