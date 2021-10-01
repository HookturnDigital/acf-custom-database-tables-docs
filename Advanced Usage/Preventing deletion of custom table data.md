# Preventing deletion of custom table data

By default, the plugin will intercept calls made to delete posts and remove all custom table data when the post is
removed from the database.

**Note:** Moving a post to the trash does not trigger data removal from custom database tables. This only occurs when
the post is removed permanently from the database via the WordPress admin or by invoking
the [wp_delete_post() function](https://developer.wordpress.org/reference/functions/wp_delete_post/).

Be mindful that the deleted post IDs will remain in the custom database tables if you decide to allow the data to
persist after the post is deleted. Be sure to consider this in any custom SQL queries you may write.

## How to prevent deletion of ACF field values from custom database tables

You may prevent data removal through the use of either the `acfcdt/delete_all_custom_table_data`
or the `acfcdt/delete_custom_table_data` filters. These filters are very similar but have slightly different approaches
as outlined below.

### Where to place this code?

Whilst it's possible to place these snippets within your theme's `functions.php` file, it is a safer bet to place them
in a configuration plugin. An [MU plugin](https://wordpress.org/support/article/must-use-plugins/) is ideal as it isn't
at risk of accidental deactivation.

## How to prevent deletion of all custom table values

You may use the `acfcdt/delete_all_custom_table_data` filter to prevent deletion of custom table data from all custom
tables. The data passed to the filter allows evaluation of the post ID and/or post type. The filter runs once when
deleting a post and requires a `boolean` value be returned. e.g;

```php
add_filter( 'acfcdt/delete_all_custom_table_data', function ( $bool, $post_id, $post_type ) {
    // Don't delete custom table data for this post type.
    if ( $post_type === 'my_custom_post_type' ) {
        return false;
    }
    
    return $bool;
}, 10, 3 );
```

## How to prevent deletion of data from specific custom database tables

You may use the `acfcdt/delete_custom_table_data` filter to prevent deletion of custom table data from specific tables.
The data passed to the filter allows evaluation of the post ID, post type, and/or the database table name. The filter
runs inside of a loop that works through each custom database table name for the given post type. The filter requires
a `boolean` value be returned. e.g;

```php
add_filter( 'acfcdt/delete_custom_table_data', function ( $bool, $post_id, $post_type, $table_name ) {
    // Don't delete custom table data for this DB table.
    if ( $table_name === 'my_custom_db_table' ) {
        return false;
    }
    return $bool;
}, 10, 4 );
```

**Note:** the database table name does not include the database table prefix.

### Handling sub tables and/or join tables

When using sub tables (for repeater fields) or join tables (for relational fields), these tables need to be evaluated
individually as they are considered separate tables in their own right. e.g;

```php
add_filter( 'acfcdt/delete_custom_table_data', function ( $bool, $post_id, $post_type, $table_name ) {
    // Don't delete custom table data for this DB table.
    if ( $table_name === 'my_custom_db_table' ) {
        return false;
    }
    
    // Don't delete custom table data for a repeater table.
    if ( $table_name === 'my_custom_db_table__my_repeater' ) {
        return false;
    }
    
    // Don't delete custom table data for a relational field using a join table.
    if ( $table_name === 'my_custom_db_table__my_relationship' ) {
        return false;
    }
    
    return $bool;
}, 10, 4 );
```