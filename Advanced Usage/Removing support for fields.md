# Removing support for fields

You can remove fields from the supported field list, if you need to. When doing so, you have the ACF field array to evaluate, so you have a good degree of flexibility:

```php
<?php
/*
 * Removing fields from the 'supported fields' list. This affects table definition
 * generation and can go in your functions.php file or a plugin.
 */
add_filter( 'acfcdt/is_supported_field', function ( $is_supported, $field ) {

	// removing support for a built in field type
	if ( $field['type'] === 'text' ) {
		return false;
	}

	// removing support for a specific field by name
	if ( $field['name'] === 'my_custom_field' ) {
		return false;
	}

	return $is_supported;
}, 10, 2 );

```