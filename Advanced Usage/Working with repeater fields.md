# Working with repeater fields

By default, the plugin ignores repeater fields allowing their data to pass through to ACF for storage in core metatables. This is to maintain consistency after updating to version 1.1. 

You may enable repeater support using either the plugin's settings panel, or a WordPress filter and then further configure the plugin — using WordPress filters — to create sub-tables for specific repeaters where necessary.

## A note on nested repeaters

At this stage, nested repeaters will be encoded into the top-level repeater data. Nested repeaters do not currently create their own tables.

## Some caveats to be aware of

### Potential API function limitations

Whilst the plugin support ACFs `have_rows()`, `the_row()`, `the_sub_field()`, `get_sub_field()`, `has_sub_field()` for displaying repeater data, you may experience issues with some other row-related functions. If you do run into any issues here, please let us know.

### Changing data storage location of an existing field

A repeater field can be stored either as an encoded field in a custom table, or as a normalised table of its own. Either of these storage options will fallback to core metadata if a value is not in a custom table. 

However, if a field's storage option changes — e.g; from 'encoded' to 'sub-tabled' — repeaters with existing data will appear to be empty as the plugin does not currently look to an alternative storage format for data. For this reason, the storage format is best determined at the beginning of a project or before live data is in play. 

If you are in a situation where you need to change the storage format and need to migrate data from one format to another, [email us](mailto:support@hookturn.io) for some advice on how to approach the situation.

## How to enable repeater field support

You may enable repeater field support either via the admin settings or via PHP using a WordPress filter. 

Once repeater field support is enabled, field group updates will factor in the repeater fields by either adding new columns or sub tables to the custom database table definition files.

### To enable repeater field support via the WordPress admin:

1. Head to **Custom Fields > Database Tables > Settings** 
2. In the **Complex Field Support** section, check the **Enable Repeater Support** option
3. Click **Save Changes**. 

### To enable repeater field support via PHP (recommended):

```php
<?php
// Enable repeater field support. 
add_filter( 'acfcdt/settings/enable_repeater_field_support', '__return_true' );
```

## How to create sub tables for repeater fields

If you would like repeater field data to be stored in a separate database table, you may use the following WordPress filter to enable sub-tables:

```php
<?php
// Hook into the table definition process to control whether or not eligible fields will
// result in sub tables. Note: only eligible fields are passed through this hook so it
// is not possible to force sub tables for ineligible field types. 
add_filter( 'acfcdt/field_creates_sub_table', 'xyz_create_sub_tables', 10, 3 );

/**
 * @param boolean $create_sub_table Whether or not to create a sub-table for the given field.
 * @param array $field The ACF field array.
 * @param string $table_name The table name without the WordPress database prefix. 
 *
 * @return bool Return TRUE to create a sub table for an eligible field. 
 */
function xyz_create_sub_tables( $create_sub_table, $field, $table_name ) {

	if ( $table_name === 'my_custom_table' and $field['name'] === 'my_repeater_field' ) {
		$create_sub_table = true;
	}

	return $create_sub_table;
}
```

#### Important!

When a repeater field is configured to create a sub table, the current pre-release version creates an auto-incrementing field called `id`. Be mindful this ID field **is not persistent** and will potentially change as the row order changes. This happens due to the way repeaters are handled in the ACF core as repeater rows do not intrinsically have their own ID. 

For this reason, if you need to update repeater rows directly via SQL, it is safer to use the `post_id` and `_sort_order` columns.

We'll be investigating ways to make these persistent but for now, it's best not to rely on the `id` column as a permanent identifier for a repeater row. 