# Activating join tables on eligible fields

Join tables are available where required for eligible fields and can be activated using filters. The following fields are eligible for join tables:

1. Relationship
2. Post Object
3. Page Link
4. User
5. Taxonomy

## Global activation

It is possible to set a global setting that will enable join tables on all eligible fields. We don’t recommend this, as you will have a very large number of database tables, but if you need to, you can use the following:

```php
<?php
/*
 * This will enable join tables on all eligible fields globally. This affects
 * table definition generation and can go in your functions.php file or a plugin.
 */
add_filter( 'acfcdt/settings/enable_join_tables_globally', '__return_true' );
```

## Activation by table name, field type, and/or field name

```php
<?php
/*
 * This demonstrates some of the options available for join table activation. 
 * This affects table definition generation and can go in your functions.php 
 * file or a plugin.
 */
add_filter( 'acfcdt/field_creates_join_table', function ( $create_join_table, $field, $table_name ) {

	// activate for an entire table
	if ( $table_name === 'my_custom_table' ) {
		$create_join_table = true;
	}

	// activate for a field type
	if ( $field['type'] === 'post_object' ) {
		$create_join_table = true;
	}

	// activate for a particular field by name
	if ( $field['name'] === 'my_post_object_field' ) {
		$create_join_table = true;
	}

	return $create_join_table;
}, 10, 3 );
```

## Activation for all eligible fields configured to accept multiple values

If you prefer, you can use the ACF field array options to enable join tables on fields that match specific conditions. For example, you may wish to enable join tables on all eligible fields that accept multiple values:

```php
<?php
/*
 * This will enable join tables for all eligible fields that are configured
 * to accept multiple values. This affects table definition generation and 
 * can go in your functions.php file or a plugin.
 */
add_filter( 'acfcdt/field_creates_join_table', function ( $create_join_table, $field, $table_name ) {

	// these fields us the 'allow multiple' toggle
	if ( in_array( $field['type'], [ 'post_object', 'page_link', 'user' ] ) ) {
		if ( $field['multiple'] ) {
			$create_join_table = true;
		}
	}

	// relationship field uses the min/max approach
	if ( $field['type'] === 'relationship' ) {
		if ( $field['max'] != 1 ) {
			$create_join_table = true;
		}
	}

	// taxonomy field uses different field types
	if ( $field['type'] === 'taxonomy' ) {
		if ( in_array( $field['field_type'], [ 'multi_select', 'checkbox' ] ) ) {
			$create_join_table = true;
		}
	}

	return $create_join_table;
}, 10, 3 );
```

Note: This approach should only be used when you know field’s settings are concrete.