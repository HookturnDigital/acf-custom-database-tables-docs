# Filtering values before insertion

If you need to modify values before they are stored in a custom database table, you can use the following filter:

```php
<?php
add_filter('acfcdt/filter_value_before_update', 'xyz_filter_value_before_insert', 10, 3);

/**
 * @param mixed $value The field value being stored. 
 * @param string $selector Either the post ID or a compound ACF selector depending on the object type. e.g; user_1.
 * @param array $field_array The ACF field array.
 *
 * @return mixed
 */
function xyz_filter_value_before_insert( $value, $selector, $field_array ) {
	
	if ( $field_array['name'] === 'my-custom-field' ) {
		// …modify the value here…
	}

	return $value;
}
```

If you need to filter fields within a repeater, you need to use RegEx to target individual fields.

```php
<?php
add_filter( 'acfcdt/filter_value_before_update', 'xyz_filter_repeater_value_before_insert', 10, 3 );

/**
 * @param mixed $value The field value being stored.
 * @param string $selector Either the post ID or a compound ACF selector depending on the object type. e.g; user_1.
 * @param array $field_array The ACF field array.
 *
 * @return mixed
 */
function xyz_filter_repeater_value_before_insert( $value, $selector, $field_array ) {

	// Given the dynamic nature of repeaters, we need to use RegEx to target field patterns. The parentheses
	// in the following ReGex patterns ensure the row number is available in the $matches array. 
	if ( preg_match( '/^my_repeater_field_(\d+)_subfield_a$/', $field_array['name'], $matches ) ) {
		$row_number = $matches[1];
		$value = "Custom value for subfield A in row $row_number";

	} else if ( preg_match( '/^my_repeater_field_(\d+)_subfield_b$/', $field_array['name'], $matches ) ) {
		$row_number = $matches[1];
		$value = "Custom value for subfield B in row $row_number";
	}

	// If you need to target nested repeater fields, adjust the pattern to suit.
	if ( preg_match( '/^my_repeater_field_(\d+)_nested_repeater_(\d+)_subfield$/', $field_array['name'], $matches ) ) {
		$row_number = $matches[1];
		$inner_row_number = $matches[2];
		$value = "Custom value for subfield in row $inner_row_number of repeater field row $row_number";

	}

	return $value;
}
```