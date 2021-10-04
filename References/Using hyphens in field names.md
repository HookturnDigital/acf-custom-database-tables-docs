# Using hyphens in field names

The plugin will support the use of hyphens — e.g; `my-field` — in ACF field names but be mindful that the hyphen will be
replaced by an underscore in the database table column name.

## Accessing the field values via ACFs API functions

You may continue to use ACFs API functions using the field name exactly as defined in your ACF field group as the
plugin's internal mapping system takes care of any conversion necessary to map the hyphenated field name to the
underscored equivalent column name.

### Important! Distinguishing two fields of the same name with only a hyphen can be problematic

If you attempt to use two fields of the same name and differentiate them using only hyphens and undescores —
e.g; `my-field` and `my_field` — they will be treated as the same field at the database layer as the plugin replaces the
hyphen with an underscore resulting in the same column name.

This will create problems when accessing the data as calls to both `get_field('my-field')` and `get_field('my_field')`
will both return data from the `my_field` table column.

## Accessing the field values via SQL

When using SQL queries to retrieve data, you will need to use the hyphenated version of the field name as SQL will treat
`my-field` and `my_field` as different fields. 