# Working with repeater fields

By default, the plugin ignores repeater fields allowing their data to pass through to ACF for storage in core
metatables. This is to maintain consistency after updating to version 1.1.

You may enable repeater support using either the plugin's settings panel, or a WordPress filter and then further
configure the plugin — using WordPress filters — to create sub-tables for specific repeaters where necessary.

## A note on nested repeaters

At this stage, nested repeaters will be encoded into the top-level repeater data. Nested ACF repeaters are not currently
eligible for creating their own normalized database tables.

## Some caveats to be aware of

### Potential API function limitations

We strive to fully support ACF's template API and as a result we have an extensive number of automated tests to check
for continued functionality of the template functions that ACF offers as part of its template API. The following
functions are all currently covered by our tests:

- `get_field()`
- `the_field()`
- `update_field()`
- `delete_field()`
- `have_rows()`
- `the_row()`
- `get_row()`
- `the_row_index()`
- `get_row_index()`
- `add_row()`
- `delete_row()`
- `the_sub_field()`
- `has_sub_field()`
- `get_sub_field()`
- `add_sub_row()`
- `delete_sub_field()`
- `delete_sub_row()`
- `update_row()`
- `update_sub_field()`
- `update_sub_row()`
- `get_sub_field_object()`
- `get_row_sub_field()`
- `get_row_sub_value()`

Whilst extensive, our tests may not cover all use cases/scenarios so if you happen to find any issues here, please let
us know via [support@hookturn.io](mailto:support@hookturn.io) so we can further improve our deep support of ACF's
template API.

### Changing data storage format of an existing field

If the data structure is changed after a field is live and has been collecting data, existing posts/users may appear to
be missing data. The data is still in the database but the plugin does not fallback to the previous data structure where
custom table data is missing. See [Important note on structural changes](#important-note-on-structural-changes) for
details.

### Repeater table IDs may change

See [Important note on the `id` field](#important-note-on-the-id-field) for details.

## How to enable repeater field support

You may enable repeater field support either via the admin settings or via PHP using a WordPress filter.

Once repeater field support is enabled, field group updates will factor in the repeater fields by either adding new
columns or sub tables to the custom database table definition files.

### To enable repeater field support via PHP (recommended):

```php
<?php
// Enable repeater field support. 
add_filter( 'acfcdt/settings/enable_repeater_field_support', '__return_true' );
```

### To enable repeater field support via the WordPress admin:

1. Head to **Custom Fields > Database Tables > Settings**
2. In the **Complex Field Support** section, check the **Enable Repeater Support** option
3. Click **Save Changes**.

## Choosing a data structure for repeater fields

By default, when repeater support is enabled for a custom database table, a single column will be created to represent
the repeater data and all sub field values will be JSON-encoded within that single column.

As an altenative, you may choose to store your repeater data in a normalized table of its own, giving the data a
structure that is much better suited to both custom SQL queries and analysis. The result of data organised in this
fashion can be a much faster performing application where complex SQL queries are required.

### Should I just always use normalized tables?

There is no silver bullet and each use case is different so you should use the most appropriate structure for your use
case. Encoded payloads are easier to manage and can involve less DB queries to access the data, but it is harder and
less performant to perform SQL queries on this data. On the other hand, normalized database tables are much easier to
read, query, and analyse but if you have all repeater fields across an application in custom DB tables, you may be
inadvertently adding unecessary queries and joins whilst also potentially making your database harder to reason about.

Personally, I would think carefully about the nature of a specific repeater field and whether or not I need to perform
SQL queries directly on the data within it. Only then would I decide to use a normalized table structure for that
particular repeater field.

### Important note on structural changes

Regardless of whether a repeater field value is encoded or normalized into a table of its own, either of these storage
options will fallback to core metadata if a value is not in a custom table. However, if a field's storage option changes
— e.g; from 'encoded' to 'sub-tabled' — repeaters with existing data may appear to be empty as the plugin does not
currently fall back to the previous structure before finally falling back to core metadata. For this reason, the storage
format is best determined at the beginning of a project or before live data is in play.

If you are in a situation where you need to change the storage format and need to migrate data from one format to
another, it would be wise to export all data before making the structural change so you can reimport it after making the
change. At this time there is no built-in import, export, or migration feature to assist this so you'll need to either
use a third party plugin or write some custom SQL. We are working on a data migrator which will likely ship in version
1.2.

## How to create sub tables for repeater fields

You may opt to enable separate tables for repeater fields. Doing so gives you maximum flexibility and much greater
performance benefits when it comes to querying data using custom SQL statements.

### Important note on the `id` field

When a repeater field is configured to create a sub table, the current pre-release version creates an auto-incrementing
field called `id`. Be mindful this ID field **is not persistent to the repeater data row** and will potentially change
as the row order changes or as rows are removed from the repeater. This happens due to the way repeaters are handled in
ACFs core as repeater field rows do not intrinsically have their own ID but instead rely on row indexes which change
with the number and order of rows.

The `id` field currently exists on all tables created by the plugin as it is built into the underlying data model
system. We'll be investigating ways to make these persistent for the use case of repeater fields but for now, it's best
not to rely on the `id` column as a permanent identifier for a repeater row.

If you need permanent IDs it would be best to create a custom post type to leverage the power of WordPress' built-in
object model and manage relationships between objects instead of using repeater rows as objects.

Alternatively, you may consider using a custom field as a manually designated ID or potentially hooking in to generate
your own unique values that act as IDs. Just be sure to observe field name conflicts that could arise as documented
in [caveats & gotchas](../Caveats%20and%20Gotchas.md).

### Enabling a sub table for a repeater field

If you would like repeater field data to be stored in a separate normalized database table, you may use the following
WordPress filter to enable the sub-table:

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

The code is best added to a custom configuration plugin for your project but can also be added to your
theme's `functions.php` if preferred.

Once the code is in place, the field group must be saved in the WordPress admin in order to affect changes to the table
JSON definition file. From there, running the table creation/update process from the notification that appears will
apply the changes to the database. In this case, a new table will be created for the repeater field. 