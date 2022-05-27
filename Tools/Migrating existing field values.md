# Migrating existing field values

When adding the plugin to an existing site there is the potential for a lot of ACF field values to already exist in the
database. The field value migration tool makes it possible to:

1. Copy (or move) existing data from core meta tables into ACF custom database tables.
2. Copy (or move) existing data from ACF custom database tables back into core meta tables.

## When to use the field value migration tool?

There are a few scenarios where the field value migration tool will be useful:

### When adding the plugin to a site with existing data

Use the migration tool to migrate data from core meta tables to newly created custom tables.

### Before removing the plugin

Use the migration tool to backfill data from custom tables back to core meta tables.

### After changing table configuration

Use the migration tool to ensure data is where it needs to be.

Note that it may be necessary to backfill data to core meta tables before changing the structure. Once the new structure
is in place, you may then migrate the data back to the new table structure.

This is required in situation where an existing encoded repeater is changed to store data in a normalised sub table or
where an existing relationship field is changed to store data in a join table.

This is necessary as the plugin does not know what the previous structure was — it just moves between core and custom
data structures.

## How to run a migration

1. In the WordPress admin, head to **Custom Fields > Database Tables > Tools**.
2. Location the **Migrate Field Data** tool and click the **Configure Migration** button.
3. Select the object type, data flow direction, fields to migrate, and any additional options that apply.
4. Click the **Start Migration** button.

The migration will now run as a background process so you don't need to keep the tab open. The migration processes posts
in batches so it will be capable of processing large numbers of objects without running into memory issues or timeouts.

## How to limit which objects are passed through the migration process

The migration tool uses native WordPress object queries to query a batch of objects to pass through the migration. You
may choose to further limit which objects are queried for migration through the use of filters.

### Controlling the post object query

The post object query uses `WP_Query` to query object IDs matching the set of configured criteria. Before the query is
executed, the `acfcdt/migrate_field_data/post_query_args` filter is applied providing an opportunity to set custom
`WP_Query` args. You may use the filter to add constraints such as taxonomy and meta queries, date queries, post status
parameters, etc. See [the WP_Query class reference](https://developer.wordpress.org/reference/classes/WP_Query/) for a
full list of available query args.

**Note:** there are a number of query args that the migration tool specifically requires in order to function correctly.
These special args are applied _after_ the filter and cannot be controlled:

- `post_type`
- `posts_per_page`
- `orderby`
- `order`
- `no_found_rows`
- `update_post_meta_cache`
- `update_post_term_cache`
- `fields`

```php
add_filter('acfcdt/migrate_field_data/post_query_args', function( $args ){
	
	// Specify an exact set of post IDs to migrate.
	$args['post__in'] = [1,3,5,7,9];

	// Add a taxonomy constraint.
	$args['tax_query'] = [
		[
            'taxonomy' => 'my-taxonomy',
            'field'    => 'slug',
            'terms'    => 'example',
		]       
	];

	return $args;
});
```

### Controlling the user object query

The user object query uses `WP_User_Query` to query user IDs matching the set of configured criteria. Before the query
is executed, the `acfcdt/migrate_field_data/user_query_args` filter is applied providing an opportunity to set custom
`WP_User_Query` args. You may use the filter to add constraints such as user roles, registration date, custom fields
etc. See [the WP_User_Query class reference](https://developer.wordpress.org/reference/classes/wp_user_query/) for a
full list of available query args.

**Note:** there are a number of query args that the migration tool specifically requires in order to function correctly.
These special args are applied _after_ the filter and cannot be controlled:

- `number`
- `orderby`
- `order`
- `fields`
- `count_total`

```php
add_filter('acfcdt/migrate_field_data/user_query_args', function( $args ){
	
	// Specify an exact set of user IDs to migrate.
	$args['include'] = [2,4,7];

	// Limit the query to users that have a specific role.
	$args['role'] = 'Administrator';

	return $args;
});
```