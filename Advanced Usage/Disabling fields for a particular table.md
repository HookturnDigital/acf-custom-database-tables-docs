# Disabling fields for a particular table

If you need to prevent fields from generating columns on a particular table, you can use the following filter:

```php
<?php
/*
 * Filtering the array of supported fields for a given table. This affects table 
 * definition generation and can go in your functions.php file or a plugin.
 */
add_filter( 'acfcdt/field_group_supported_fields', function ( $supported_fields, $table_name ) {

	// omit the WordPress database prefix
	if ( $table_name !== 'my_custom_table' ) {
		return $supported_fields;
	}

	// filtering unwanted fields from the fields array
	$supported_fields = array_filter( $supported_fields, function ( $field ) {
		$is_excluded = in_array( $field['name'], [ 'my_text_field', 'my_text_area_field' ] );

		return ! $is_excluded;
	} );

	return $supported_fields;
}, 10, 2 );
```