# Running custom actions after a table value is updated

You may use the `acfcdt/custom_table_value_updated` action hook to run custom functionality after a field value has been
saved in a custom database table.

A common use case could be to delete core meta table values as they are saved in a custom table. e.g;

```php
add_action( 'acfcdt/custom_table_value_updated', function ( $value, $post_id, $field_array ) {
	// Delete the value entry from the core meta table.
    delete_post_meta( $post_id, $field_array['name']);
    
    // Delete the key reference from the core meta table. Note: this can break ACF's ability to locate the field array
    // and create problems later on if no field references are available through other means. Only do this if you have
    // ACF JSON enabled.
    delete_post_meta( $post_id, "_{$field_array['name']}")
}, 10, 3 );
```