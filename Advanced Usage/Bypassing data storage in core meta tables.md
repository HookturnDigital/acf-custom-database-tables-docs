# Bypassing data storage in core meta tables

By default, the plugin does not prevent storage of metadata to the core meta tables as doing so may introduce
interoperability issues with other plugins that are dependent on metadata existing in the core meta tables.

However, you may configure your site to bypass the core metatables where fields will store data in a custom database
table. This is broken down into two parts:

1. Bypassing field values
2. Bypassing field key references *(used internally by ACF to map field names to field keys)*

## Important considerations before choosing this option

If you decide to bypass data storage in core metatables, it is important you bear the following in mind:

1. You must be using ACF JSON if you decide to bypass the field key references as well as the field values. If you
   disable key reference storage, ACF may be unable to retrieve field values from any source.
2. Many third party plugins expect — and even depend — on data in the core meta tables. Be sure you don't need to use
   these plugins or be prepared to implement some form of workaround should the need arise.
3. Without metadata in the core meta tables, WordPress meta queries will no longer function. If you want to use custom
   table data in conjunction with `WP_Query`,
   see [this article.](https://hookturn.io/2019/09/how-to-use-acf-custom-database-tables-data-with-wp_query-objects/)

## How to bypass field values

### Targeting specific fields (recommended)

For granular control over which fields will be bypassed from core meta tables, you may use the following filter:

```php
<?php
add_filter( 'acfcdt/store_acf_field_values_in_core_meta', function ( $null, $field_array, $object_id, $object_type ) {

	// Return false to prevent a specific field from storing data in core meta tables.
	if ( $field_array['key'] === 'field_1234567890' ) {
		return false;
	}

	// Return true to ensure a specific field always stores data in both custom and core meta tables.
	if ( $field_array['name'] === 'my_custom_field' ) {
		return true;
	}

	// Fall back to default/global setting.
	return $null;
	
}, 10, 4 );
```

### Targeting all fields (global/fallback setting)

To bypass storage of **all meta values where a custom table is found**, you may use the following filter:

```php
<?php
/*
 * Disables storing of meta data values in core meta tables where a custom 
 * database table has been defined for fields. Any fields that aren't mapped
 * to a custom database table will still be stored in the core meta tables. 
 */
add_filter( 'acfcdt/settings/store_acf_values_in_core_meta', '__return_false' );
```

## How to bypass **field key references**

**Note:** you should only do this if you are using ACF JSON. If your field groups aren’t represented in JSON files, you
may have problems storing and retrieving data.

### Targeting specific fields (recommended)

For granular control over which field key references will be bypassed from core metatables, you may use the following
filter:

```php
<?php
add_filter( 'acfcdt/store_acf_field_key_references_in_core_meta', function ( $null, $field_array, $object_id, $object_type ) {

	// Return false to prevent a specific field from storing key references in core meta tables.
	if ( $field_array['key'] === 'field_1234567890' ) {
		return false;
	}

	// Return true to ensure a specific field always stores key references in both custom and core meta tables.
	if ( $field_array['name'] === 'my_custom_field' ) {
		return true;
	}

	// Fall back to default/global setting.
	return $null;
	
}, 10, 4 );
```

### Targeting all fields (global/fallback setting)

To bypass storage of **all ACF field key references for fields mapped to custom database tables**, you may use the
following filter:

```php
<?php
/*
 * Disables storing of ACF field key references in core meta tables where a custom 
 * database table has been defined for fields. Any fields that aren't mapped to a 
 * custom database table will still have their key references stored in the core 
 * meta tables. 
 */
add_filter( 'acfcdt/settings/store_acf_keys_in_core_meta', '__return_false' );
```