## What has changed in version 1.1

## Added

### Added plugin settings panel to the WordPress admin.

The settings panel is found in **Custom Fields > Database Tables > Settings** and provides settings for the following:

- Enabling complex field support (repeaters).
- Global setting controls.
- Controlling core meta bypass at a global level.
- Toggling modules on or off.
- Enabling third party plugin compatibility handlers.

All the settings in the panel can also be controlled using PHP filters. When controlled by a filter, the admin setting will be deactivated as code-based config takes priority. 

### Added Repeater field support.

Repeater data can now be saved as encoded data to a single column in a field group's custom database table or, where sub-table creation is enabled, can be stored in separate tables altogether where each repeater row maps to a table row and each sub-field maps to a table column.

**Note:** For details on this, see the **Advanced Usage > Working with repeater fields** section below.

### Added the ability to control column data types via WordPress filters.

Using filters, it is now possible to override the default data type used when creating/updating table columns in order to better facilitate custom SQL queries and manage a more efficient database. 

**Note:** For details on this, see the **Advanced Usage > Controlling column data types** section below.

### Added the ability to execute custom code via WordPress hooks after each table schema is created/updated in the database.

The ability to run processes right after schema update opens up a number of possibilities including:

- Custom index creation.
- Custom logging/notification.
- Custom data migration handlers.
- Table field format modifications that might not be possible due to limitations in WordPress' `dbDelta()` function.

**Note:** For details on this, see the **Advanced Usage > Running custom actions after a table is updated/created** section below.

### Added the ability to disable/enable data intercepts to and/or from custom database tables.

The ability to disable data intercepts makes it possible to use the plugin to store data in a custom table but still retrieve from core meta tables and also bypass updating custom tables where it might not be desired. 

**Note:** For details on this, see the **Advanced Usage > Disabling storage/retrieval to/from custom database tables** section below.

## Fixed

### Fixed some issues preventing the use of this plugin on some Redis configurations.

We had a few reports of issues occurring on Redis configurations. This came down to us using dependency injection to pass the global `$wpbd` instance through our application. This is now resolved.

### Fixed a bug where bypassed field names from one post type could affect core storage of another when updating both in the same process.

Prior to version 1.1, it was possible for the core data bypass functionality to prevent core meta storage when updating multiple post types in the one process. This would happen under the following conditions: 

1. Core meta storage was disabled.
2. Two post types were sharing a field name.
3. One post type would be storing data to a custom table and one was not.
4. Both of the object were updated programatically in the same process.

What was basically happening here is the field name would be marked for bypass when Post Type A was updated and when Post Type B was subsequently updated with the intention of storing its data in core meta tables, the field would be bypassed and no data would be stored in core.

It is now possible to have two different post types sharing field names and storing data in a mix of custom tables and core meta tables.

## Removed

### Removed duplicated `acf/update_value` ACF core filter and added the `acfcdt/filter_value_before_update` filter.

Prior to version 1.1, when intercepting calls to `update_field()` the plugin was passing data through a duplicated core ACF API filter before being stored in a custom database table as follows;

```php
<?php
// ACFs core filter for 3rd party customization
$value = apply_filters( "acf/update_value", $value, $selector, $field, $value );
```

In version 1.1, we've had to remove our internal use of this filter as it interfered with our ability to support repeaters. Instead, developers can now use a new filter that we've introduced —`acfcdt/filter_value_before_update` — in order to modify a value before it is stored in a custom table. 

**Note:** To learn more about this filter, see the **Advanced Usage > Filtering values before insertion** section below.

In addition to removing the filter, the `acfcdt/settings/allow_acf_update_value_filters` setting has also now been removed as it is no longer relevant.

### Removed duplicated `acf/load_value` ACF core filter.

Prior to version 1.1, when intercepting calls to `get_field()` and `the_field()` we were handling data in a way that required us to duplicate ACF's core API filters as follows;

```php
<?php
// filter for 3rd party customization
$value = apply_filters( "acf/load_value", $value, $selector, $field );
$value = apply_filters( "acf/load_value/type={$field['type']}", $value, $selector, $field );
$value = apply_filters( "acf/load_value/name={$field['_name']}", $value, $selector, $field );
$value = apply_filters( "acf/load_value/key={$field['key']}", $value, $selector, $field );
```

This was done as a way to maintain consistency with ACF as the data returned by our plugin was being handed back to ACF in such a way that it would 'short-circuit' ACF's value retrieval and bypass the load value filters.

In version 1.1, we've changed the way we intercept calls to `get_field()` and `the_field()` by going deeper and returning data to ACF's internal call to get the metadata. In doing so, data returned from the custom database table is handed back to ACF for processing via its built-in API filters and cache systems meaning we no longer need to duplicate the load value filters. 

In addition to removing the filter, the `acfcdt/settings/allow_acf_load_value_filters` setting has also now been removed as it is no longer relevant.