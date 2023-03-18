# Controlling column data types

As of version 1.1, it is possible to control the data types of custom database table columns. Controlling the type can
be useful in reducing your database size and increasing the efficiency of your queries.

It's important to note, due to the way databases work, changing the data types on an existing database may result in
data loss where the existing data is not compatible with the new data type. It is critical you back up your database
before implementing any changes.

It's also important to understand the data types you wish to use as you could end up losing data on isert into the
database where the value isn't supported by the type. e.g; using an `int` type to store strings will result a '0' in
your database.

## Understanding the various data types available in MySQL

MySQL provides a variety of data types for different purposes. Here's a list of some of the most common MySQL data
types, along with their use cases:

### Numeric Data Types:

- `INT`: Used to store integer values (whole numbers). Suitable for storing counts, IDs, or quantities.
- `FLOAT`, `DOUBLE`: Used to store approximate floating-point numbers with single or double precision, respectively.
  Suitable for scientific or statistical data where approximate values are acceptable.
- `DECIMAL`: Used to store exact fixed-point numbers. Suitable for financial data or other use cases where exact values
  are required, such as product prices or measurements.

### Date and Time Data Types:

- `DATE`: Used to store date values in the format `YYYY-MM-DD`. Suitable for storing dates without time components, such
  as birthdates or event dates.
- `TIME`: Used to store time values in the format `hh:mm:ss`. Suitable for storing durations or time of day without a
  date component.
- `DATETIME`: Used to store both date and time values in the format `YYYY-MM-DD hh:mm:ss`. Suitable for storing
  timestamps of events or activities with both date and time components.
- `TIMESTAMP`: Similar to `DATETIME` but with automatic initialization and updating features based on the current
  timestamp. Suitable for storing the creation and modification timestamps of records.

### String Data Types:

- `CHAR(size)`: Used to store fixed-length character strings. Suitable for storing small, fixed-length strings like
  country codes, state abbreviations, or gender codes.
- `VARCHAR(size)`: Used to store variable-length character strings. Suitable for storing text data with varying lengths,
  such as names, addresses, or descriptions.
- `TEXT`, `MEDIUMTEXT`, `LONGTEXT`: Used to store large character strings with varying sizes. Suitable for storing
  longer text data like articles, comments, or product descriptions.
- `BINARY(size)`, `VARBINARY(size)`: Similar to `CHAR` and `VARCHAR`, but used to store binary data instead of character
  strings. Suitable for storing non-text data like images, audio, or serialized objects.

**Note:** The `(size)` component of a data type is not to be written literally in your code – if you do so, an error
will occur when attempting to modify the database schema. `size` is a placeholder for the actual size of the data type.
For example, `VARCHAR(255)` is a data type that can store up to 255 characters.

These are just some of the most common data types in MySQL. The specific data type you choose for a column will depend
on the requirements of your application and the nature of the data being stored. To learn more about MySQL data types,
see the [MySQL reference manual](https://dev.mysql.com/doc/refman/8.0/en/data-types.html).

## Some caveats on certain data types

- WordPress' `dbDelta()` function changes `tinytext`, `text`, and `mediumtext` to `longtext` so it is not possible to
  set these formats using the provided filters. To maintain these formats, it would be necessary to run a custom SQL
  statement on the table after table creation. e.g; `ALTER TABLE my_table MODIFY COLUMN my_field mediumtext;`
- Whilst `date` and `datetime` types are fine to use with the `date_picker` field, attempting to use the `year` type on
  a date picker will break. A workaround for this is to use a normal text field and give that the type `date`.

## A high-level overview of controlling data types

Controlling the data types of your custom table columns is a 3-step process:

1. Use PHP filters to modify the data type of the desired fields — the callback function/s run **when a field group is
   saved**.
1. Save the relevant field group/s. The new data types will appear in the table JSON configuration files.
1. Run the table update process to apply the new data types to the tables.

## How to filter column data types

You have two possible points of control here to filter column data types. One is a generic filter which you could use to
evaluate the table and field names from within your hooked callback:

```php
<?php
// This is the first of two possible approaches to override the field data type.
// This approach allows you to evaluate the type, table name (unprefixed), and column 
// name to determine what type you would like to return. All columns on all custom 
// tables pass through this filter.
add_filter( 'acfcdt/set_column_data_type', 'xyz_set_column_data_types', 10, 3 );

/**
 * @param string $type The field data type
 * @param string $table_name The table name without the WordPress table prefix
 * @param string $column_name The field/column name
 *
 * @return string
 */
function xyz_set_column_data_types( $type, $table_name, $column_name ) {

	if ( $table_name === 'my_custom_table' ) {
		if ( $column_name === 'field_1' ) {
			return 'varchar(20)';
		}
	}

	if ( $table_name === 'my_other_custom_table' ) {
		switch ( $column_name ) {
			case 'custom_field':
				return 'varchar(50)';

			case 'other_custom_field':
				return 'bigint';
		}
	}

	return $type;
}
```

The other is a dynamic version of the generic filter that allows you to specify which table and field you wish to
modify. This simplifies your callback function at the cost of the ability to evaluate table and field names
programmatically:

```php
<?php
// This is the second of two possible approaches to override the field data type.
// This approach allows you to target a specific table.column. It is simpler to 
// read and only the specific column is passed through making it a bit more 
// economical.
// Note: The table name in the filter does NOT include the WordPress table prefix.
add_filter( 'acfcdt/set_column_data_type/my_custom_table.my_custom_field', 'xyz_set_column_data_type' );

/**
 * @param string $type The field data type
 *
 * @return string
 */
function xyz_set_column_data_type( $type ) {
	return 'varchar(60)';
}
```