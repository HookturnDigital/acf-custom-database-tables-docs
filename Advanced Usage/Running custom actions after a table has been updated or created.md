# Running custom actions after a table has been updated or created

As of version 1.1, it is possible to run custom handlers during the table update process. The ability to run processes right after schema update opens up a number of possibilities including:

- Custom index creation.
- Custom logging/notification.
- Custom data migration handlers.
- Table field format modifications that might not be possible due to limitations in WordPress' `dbDelta()` function.

## How to enable the after table schema update module

You may enable the module either via the admin settings or via PHP using a WordPress filter. 

Once the module is enabled, actions will fire during the table schema update process that support executing custom PHP code.

### To enable the module via the WordPress admin:

1. Head to **Custom Fields > Database Tables > Settings** 
2. In the **Module** section, check the **Enable after table schema update actions** option
3. Click **Save Changes**. 

### To enable the module via PHP:

```php
<?php
// Note: this code needs to run before your theme's functions.php so putting this in a configuration plugin
// is the way to go. An MU plugin would be ideal as it will ensure your configuration cannot be deactivated
// in the WordPress admin.
add_filter( 'acfcdt/settings/activate_modules', 'xyz_activate_table_schema_update_module');

/**
 * @param array $modules Array of modules and their active status.
 *
 * @return array
 */
function xyz_activate_table_schema_update_module( $modules ) {
	$modules['after_table_schema_update'] = true;

	return $modules;
}
```

## How to run custom actions after each table has been updated

You may run a callback that is invoked after each individual table has been updated. The callback receives both the `dbDelta()` results up to that point as well as the table name allowing you to carry out tasks on more than one table from the one callback. 

**Note:** It's important to take note that this is actually a filter so that you can append messages to the output in the WordPress admin. This means your callback function must return the first parameter — in this case, `$db_delta_results` — for the plugin to continue functioning correctly.

```php
<?php
add_filter( 'acfcdt/after_create_or_update_table', 'xyz_run_after_each_table_is_updated', 10, 2 );

/**
 * @param array $db_delta_results An array of dbDelta() output messages that will appear in the admin.
 * @param string $table_name The table name — without the WordPress DB prefix — that was just updated.
 *
 * @return array
 */
function xyz_run_after_each_table_is_updated( $db_delta_results, $table_name ) {
	// ... do whatever you like here ...

	return $db_delta_results;
}
```

## How to run a custom action after a specific table has been updated

If you only wish to run an action after a specific table, you may use a dynamic version of the same hook. The hooked callback will only be invoked after the specific table has been created or updated. 

**Note:** It's important to take note that this is actually a filter so that you can append messages to the output in the WordPress admin. This means your callback function must return the first parameter — in this case, `$db_delta_results` — for the plugin to continue functioning correctly.

```php
<?php
add_filter( 'acfcdt/after_create_or_update_table/my_custom_table', 'xyz_run_after_table_is_updated' );

/**
 * @param array $db_delta_results An array of dbDelta() output messages that will appear in the admin.
 *
 * @return array
 */
function xyz_run_after_table_is_updated( $db_delta_results ) {
	// ... do whatever you like here ...

	return $db_delta_results;
}
```

## How to run a custom action after all tables have been updated

There may be times when you need to run a single action after all tables have been updated. You may do so as follows:

```php
<?php
add_filter( 'acfcdt/after_create_or_update_tables', 'xyz_run_after_all_tables_updated' );

/**
 * @param array $db_delta_results Array of notices returned by dbDelta() while updating the tables.
 *
 * @return array
 */
function xyz_run_after_all_tables_updated( $db_delta_results ) {
	// …do whatever you like here…
  
	// Add a custom message to the output displated in the admin.
	$db_delta_results[] = 'Some custom message';

	return $db_delta_results;
}
```

## An example of creating a custom index after a table has been created or updated

The following example illustrates how you could utilise this capability to programmatically create table indexes on your custom tables.

**Note:** It's important to take note that this is actually a filter so that you can append messages to the output in the WordPress admin. This means your callback function must return the first parameter — in this case, `$db_delta_results` — for the plugin to continue functioning correctly.

```php
<?php
add_filter( 'acfcdt/after_create_or_update_table/my_custom_table', 'xyz_create_table_index', 10 );

/**
 * @param array $db_delta_results An array of dbDelta() output messages that will appear in the admin.
 *
 * @return array
 */
function xyz_create_table_index( $db_delta_results ) {
	global $wpdb;

	$table_name = 'my_custom_table';
	$full_table_name = $wpdb->prefix . $table_name;
	$index_name = 'my_custom_index';

	$index_already_exists = $wpdb->get_var( "SHOW INDEX FROM `{$full_table_name}` WHERE key_name = '$index_name'" );

	if ( $index_already_exists ) {
		return $db_delta_results;
	}

    $index_created = $wpdb->query( "CREATE INDEX `$index_name` ON `{$full_table_name}` (some_field(255), some_other_field(255));" );

    if ( $index_created ) {
        $db_delta_results["$table_name.$index_name"] = "Created index `$index_name` on table `{$full_table_name}`.";

    } else {
        $db_delta_results["$table_name.$index_name"] = "Could not create index on `{$full_table_name}`: {$wpdb->last_error}";
    }

	return $db_delta_results;
}
```