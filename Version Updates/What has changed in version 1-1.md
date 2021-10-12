# What has changed in version 1.1?

Version 1.1 is a much larger release than expected as the time since releasing version 1.0.5 has been significant,
allowing the scope to creep on this version. There is a fair bit to observe here, particularly for anyone who is using
the plugin with their own customisation layer in place.

Aside from repeater field support, a lot of attention has been put into building an extensive arsenal of automated tests
to safeguard against regressions and problematic changes. This does not remove the need to test the update on a
development/staging server, however. Please always take a backup and test the plugin updates in a safe environment to
ensure uninterrupted functionality of your websites.

## Added

### Added plugin settings panel to the WordPress admin.

The settings panel is found in **Custom Fields > Database Tables > Settings** and provides settings for the following:

- Enabling complex field support (repeaters).
- Global setting controls.
- Controlling core meta bypass at a global level.
- Toggling modules on or off.
- Enabling third party plugin compatibility handlers.

All the settings in the panel can also be controlled using PHP filters. When controlled by a filter, the admin setting
will be deactivated as code-based config takes priority.

### Added Repeater field support.

Repeater data can now be saved as encoded data to a single column in a field group's custom database table or, where
sub-table creation is enabled, can be stored in separate tables altogether where each repeater row maps to a table row
and each sub-field maps to a table column.

**Note:** For details on this,
see [Advanced Usage > Working with repeater fields](../Advanced%20Usage/Working%20with%20repeater%20fields.md).

### Added the ability to control column data types via WordPress filters.

Using filters, it is now possible to override the default data type used when creating/updating table columns in order
to better facilitate custom SQL queries and manage a more efficient database.

**Note:** For details on this,
see [Advanced Usage > Controlling column data types](../Advanced%20Usage/Controlling%20column%20data%20types.md).

### Added the ability to execute custom code via WordPress hooks after each table schema is created/updated in the database.

The ability to run processes right after schema update opens up a number of possibilities including:

- Custom index creation.
- Custom logging/notification.
- Custom data migration handlers.
- Table field format modifications that might not be possible due to limitations in WordPress' `dbDelta()` function.

**Note:** For details on this,
see [Advanced Usage > Running custom actions after a table is updated/created](../Advanced%20Usage/Running%20custom%20actions%20after%20a%20table%20is%20updated%20or%20created.md)
.

### Added the ability to disable/enable data intercepts to and/or from custom database tables.

The ability to disable data intercepts makes it possible to use the plugin to store data in a custom table but still
retrieve from core meta tables and also bypass updating custom tables where it might not be desired.

**Note:** For details on this,
see [Advanced Usage > Disabling storage/retrieval to/from custom database tables](../Advanced%20Usage/Disabling%20storage%20or%20retrieval%20to%20and%20from%20custom%20database%20tables.md)
.

### Added a 'tools' tab and our first tool for rebuilding the table map JSON system on demand.

In order to support repeaters in a sensible manner, the structure of the generated table JSON defintion files and the
PHP map file needed to change. This involved the introduction of a background process that will handle this in one
action without site administrators having to re-save each field group individually.

On update to version 1.1, the process will run automatically to ensure all active tables continue to function with the
modified map system.

For convenient, this process can be run on demand and allows site administrators to choose exactly which field groups
are handled. The tool is located in **Custom Fields > Database Tables > Tools**.

The tools area will have more tools added in subsequent releases as we build more features into the plugin.

### Added an opt-in compatibility layer for WP All Import

Due to the strategy the WP All Import plugin uses when importing data, several core ACF hooks and filters aren't
executed which renders the intercept logic in ACF Custom Database Tables useless when importing data. We've added a
compatibility layer which works around this issue as best we can. This is to be considered experimental while we work
with the WP All Import team to see if we can improve compatibility there. The compatibility layer can be enabled in
**Custom Fields > Database Tables > Settings > Compatibility**.

### Added filters for controlling encode/decode behaviour of field values when storing/retrieving custom table data

We've added a number of filters that make it possible to handle edge cases such as:

- Disabling encoding/decoding of values.
- Replacement of the encoding/decoding strategies for custom table data.
- Customisation of how the data is JSON encoded.

These are rare-use cases but serve those in need of fine control in situations where unicode escaping causes issues or
there is a desire to store and return a JSON-encoded string intead of returning an object/array. These can also
facilitate the preference to use serialization (or some other strategy) in place of JSON encoding.

For details on this,
see [Advanced Usage > Overriding default field value encode and decode behaviours](../Advanced%20Usage/Overriding%20default%20field%20value%20encode%20and%20decode%20behaviours.md)
.

### Added filters for preventing the deletion of custom table data.

We've added some filters which make it possible to prevent custom table data from being deleted when a post is deleted.
The filters make it possible to target this logic to specific post IDs, post types, or entire database tables.

This might be useful for anyone who is collecting data using post objects that are ephemeral.

For detail on this,
see [Advanced Usage > Preventing deletion of custom table data](../Advanced%20Usage/Preventing%20deletion%20of%20custom%20table%20data.md)
.

## Fixed

### Fixed some issues preventing the use of this plugin on some Redis configurations.

We had a few reports of issues occurring on Redis configurations. This came down to us using dependency injection to
pass the global `$wpbd` instance through our application. This is now resolved.

### Fixed a bug where bypassed field names from one post type could affect core storage of another when updating both in the same process.

Prior to version 1.1, it was possible for the core data bypass functionality to prevent core meta storage when updating
multiple post types in the one process. This would happen under the following conditions:

1. Core meta storage was disabled.
2. Two post types were sharing a field name.
3. One post type would be storing data to a custom table and one was not.
4. Both of the object were updated programatically in the same process.

What was basically happening here is the field name would be marked for bypass when Post Type A was updated and when
Post Type B was subsequently updated with the intention of storing its data in core meta tables, the field would be
bypassed and no data would be stored in core.

It is now possible to have two different post types sharing field names and storing data in a mix of custom tables and
core meta tables.

### Fixed an issue where post revisions could interfer with custom database table data removal on post delete

If a post type had revisions enabled, the removal of those revisions while deleting the post was intefering with the
plugin's ability to delete any data from custom database tables for the given post. The handling around this has been
improved so this is no longer the case. Automated tests are in place to protect against this in future.

### Fixed some deprecation notices that appear in PHP 8

Not much to say here other than the plugin should no longer produce deprecation warnings when used with PHP version 8.

## Removed

### Removed duplicated `acf/update_value` ACF core filter and added the `acfcdt/filter_value_before_update` filter.

Prior to version 1.1, when intercepting calls to `update_field()` the plugin was passing data through a duplicated core
ACF API filter before being stored in a custom database table as follows;

```php
<?php
// ACFs core filter for 3rd party customization
$value = apply_filters( "acf/update_value", $value, $selector, $field, $value );
```

In version 1.1, we've had to remove our internal use of this filter as it interfered with our ability to support
repeaters. Instead, developers can now use a new filter that we've introduced —`acfcdt/filter_value_before_update` — in
order to modify a value before it is stored in a custom table.

**Note:** To learn more about this filter, see the **Advanced Usage > Filtering values before insertion** section below.

In addition to removing the filter, the `acfcdt/settings/allow_acf_update_value_filters` setting has also now been
removed as it is no longer relevant.

### Removed duplicated `acf/load_value` ACF core filter.

Prior to version 1.1, when intercepting calls to `get_field()` and `the_field()` we were handling data in a way that
required us to duplicate ACF's core API filters as follows;

```php
<?php
// filter for 3rd party customization
$value = apply_filters( "acf/load_value", $value, $selector, $field );
$value = apply_filters( "acf/load_value/type={$field['type']}", $value, $selector, $field );
$value = apply_filters( "acf/load_value/name={$field['_name']}", $value, $selector, $field );
$value = apply_filters( "acf/load_value/key={$field['key']}", $value, $selector, $field );
```

This was done as a way to maintain consistency with ACF as the data returned by our plugin was being handed back to ACF
in such a way that it would 'short-circuit' ACF's value retrieval and bypass the load value filters.

In version 1.1, we've changed the way we intercept calls to `get_field()` and `the_field()` by going deeper and
returning data to ACF's internal call to get the metadata. In doing so, data returned from the custom database table is
handed back to ACF for processing via its built-in API filters and cache systems meaning we no longer need to duplicate
the load value filters.

In addition to removing the filter, the `acfcdt/settings/allow_acf_load_value_filters` setting has also now been removed
as it is no longer relevant.