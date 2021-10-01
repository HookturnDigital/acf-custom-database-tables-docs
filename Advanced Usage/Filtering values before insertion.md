# Filtering values before insertion

You may use the `acfcdt/filter_value_before_update` filter to modify values before they are stored in custom database
tables. The filter runs after ACF's core `acf/update_value` filter and does not affect values stored in core meta tables
— this purely provides a way to manipulate data stored in custom tables only. If you need to modify data consistently
across both storage mediums, consider using the
[acf/update_value](https://www.advancedcustomfields.com/resources/acf-update_value/) filter instead.

## Where to place these code snippets

Whilst it's possible to place these snippets within your theme's `functions.php` file, it is a safer bet to place them
in a configuration plugin. An [MU plugin](https://wordpress.org/support/article/must-use-plugins/) is ideal as it isn't
at risk of accidental deactivation.

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

## Filtering repeater values

It's important to note that the full repeater field's payload isn't passed to to this filter. Instead, each individual
field withing the repeater is passed through this filter for individual processing as ACF works through the sub field
structure. If you need to filter fields within a repeater, you need to use RegEx to target individual fields due to the
nature of complex field names. e.g;

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