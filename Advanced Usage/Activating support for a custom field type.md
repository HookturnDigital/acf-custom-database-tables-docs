# Activating support for a custom field type

If you are using custom ACF field types and wish to store their values in a custom database table, you need to register
the field type as supported. You can do so using the following:

```php
<?php
/*
 * This will register a custom field type as supported by the plugin. 
 * This affects table definition generation and can go in your functions.php 
 * file or a plugin.
 */
add_filter( 'acfcdt/is_supported_field', function ( $is_supported, $field ) {

	// registering a custom field type as supported
	if ( $field['type'] === 'acf_custom_field_type' ) {
		return true;
	}

	return $is_supported;
}, 10, 2 );
```

If you have any issues with the data from a custom field type, please let us know, so we can analyse how the custom
field provides data and account for this in a future update.