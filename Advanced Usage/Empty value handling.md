# Empty value handling

When deleting a field value from a custom database table — either by passing an empty value to `update_field()` or by
invoking `delete_field()` — the data type must be considered to ensure the table column receives the correct 'empty'
value. This is especially true when [controlling column data types](Controlling%20column%20data%20types.md) for maximum
efficiency.

For example, a string type requires an empty string whereas numerical and date types require a NULL value to effectively
clear the field. The plugin handles this for you but there may be situations where you require control over this
behaviour. For this reason, a collection of filters are available that allow you to control:

1. Whether an incoming value should be considered empty — this will result in an empty table column.
1. The value that should be used as an empty value when updating a particular data type.
1. The value that should be used as an empty value when updating a particular field.

## Controlling whether an incoming value should be considered empty

By default, the plugin considers empty strings and `null` to be empty values. This is in line with WordPress' core meta
handling system and maintains behaviour consisten with ACF. However, there may be other values you wish to result in an
empty column.

An example of this might be when passing an empty array to update_field() when updating a relationship field. When saved
to the database, the empty array is serialized in core metatables, and is JSON encoded in custom DB tables. Both data
stores have an equivalent string representation of an empty array.

The following example demonstrates how to configure relationship fields with empty arrays to store an empty value in
custom database tables:

```php
<?php
add_filter( 'acfcdt/value_is_empty', 'xyz_filter_value_is_empty', 10, 3 );

/**
 * @param bool $is_empty Whether or not the value is considered empty.
 * @param mixed $value The incoming value to be saved.
 * @param array $field The ACF field array.
 *
 * @return bool
 */
function xyz_filter_value_is_empty( $is_empty, $value, $field ) {
	if ( $field['type'] === 'relationship' and $value === [] ) {
		return true;
	}

	return $is_empty;
}
```

Be sure to be test thoroughly when checking for empty values. Avoid using `empty()` unless it is exactly what you are
after as `empty( 0 ) === TRUE` and this can result in empty values when a field actually contains a zero.

## Specifying the empty value for a particular field

You may specify which value should be used for a given field to represent an empty value. This value is intended for use
to clear the table value for the field in question but may also be used to specify alternative values when necessary.

```php
<?php
add_filter( 'acfcdt/empty_value_for_field', 'xyz_filter_empty_value_for_field', 10, 2 );

/**
 * @var mixed $value The value used to 'empty' the relevant database table field.
 * @var array $field The ACF field array.
 */
function xyz_filter_empty_value_for_field( $value, array $field ) {
	if ( $field['name'] === 'my_custom_field' ) {
		return null; // for this particular field, an empty value should be null
	}

	return $value;
}
```

## Specifying the empty value for a particular data type

You may specify which value should be used for a given SQL data type to represent an empty value. This value is intended
for use to clear the table value for any field with a specific data type.

```php
<?php
add_filter( 'acfcdt/empty_value_for_data_type', 'xyz_filter_empty_value_for_data_type', 10, 2 );

/**
 * @var mixed $value The empty value for the given column data type.
 * @var string $type The SQL data type.
 */
function xyz_filter_empty_value_for_data_type( $value, $type ) {
	if ( $type === 'smallint' ) {
		return null;
	}

	return $value;
}
```