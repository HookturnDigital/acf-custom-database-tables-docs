# Enabling support for a third-party field

ACF Custom Database Tables ships with support for the field types bundled with ACF. Field types provided by third-party
plugins — such as ACF Extended, or any other plugin that registers its own ACF field type — are not supported out of the
box. To store a third-party field's values in a custom database table, you need to register the field type as supported.

This is done with the `acfcdt/is_supported_field` filter:

```php
<?php
/*
 * Register a third-party ACF field type as supported by ACF Custom Database Tables.
 * Drop this in your theme's functions.php or a small site-specific plugin.
 */
add_filter( 'acfcdt/is_supported_field', function ( $is_supported, $field ) {

	// The field type name as registered by the third-party plugin.
	if ( $field['type'] === 'acfe_advanced_link' ) {
		return true;
	}

	return $is_supported;
}, 10, 2 );
```

Once the field type is registered as supported, it will be included in table definition generation, and its values will
be written to your custom table on save.

## Finding the field type name

The `$field['type']` value is the slug the third-party plugin uses when it registers the field via `acf_register_field_type()`.
You can confirm it by inspecting the field array in your code (for example, by `var_dump`ing `$field` inside the filter)
or by looking at the third-party plugin's source.

## Controlling the column data type

Third-party fields often store more complex values than the core ACF field types. If the default column type isn't a
good fit, override it using the `acfcdt/set_column_data_type` filter — see
[Controlling column data types](./controlling-column-data-types/) for the full reference.

## Transforming the value before storage

If the third-party field returns a structure that needs reshaping before it hits the table — for example, flattening a
multi-part value into a string, or encoding an array as JSON — use the `acfcdt/filter_value_before_update` filter. See
[Filtering values before insertion](./filtering-values-before-insertion/).

## See also

- [Activating support for a custom field type](./activating-support-for-a-custom-field-type/) — the same filter, framed
  for field types you've written yourself.

If you run into issues with the data a particular third-party field provides, let us know so we can account for it in
a future update.
