# ACF Custom Database Tables plugin documentation 

This repository is the source documentation for the [ACF Custom Database Tables WordPress plugin](https://hookturn.io/downloads/acf-custom-database-tables/).

The plugin is an extension for Advanced Custom Fields that makes it possible to store ACF data in organised, structured database tables either alongside or in place of WordPress' core meta tables.

## Main features of the plugin

- Easily create custom database tables with our simple UI.
- Data storage in core metatables can be deactivated entirely.
- Relational fields can form their own tables.
- Map ACF fields to columns in custom database tables.
- It is also possible (currently through the use of filters) to bypass saving of data to core meta tables which can drastically reduce the size of those tables.

## The benefits of using custom tables to store your ACF field data

- **Portability.** Export a whole table with ease.
- **Searchability.** Structured data can be searched much faster and with much greater 
- **Speed.** Structured data can offer less work to database engines.
- **Scalability.** More efficient queries means less strain on resources 

## Frequently Asked Questions

### Does this plugin work with user data?

Yes. Custom database tables can be defined for field groups that have a User Form in their location settings.

### Can I use this plugin for custom post types?

Absolutely. Whilst the plugin supports the built in types of post and page, it is ideally suited for storing custom post type meta data in structured tables.

### Can I reuse an ACF field group on more than one object?

At this time, only one object mapping is supported per field group, so selecting more than one object (post type, user) at a time will deactivate the custom table definition settings.

We are working on support for multiple objects per table. This should ship with a subsequent 1.x release.

For now, if you need the same field group on multiple objects, you’ll need to duplicate the field group and have a custom table for each.

### Does this plugin work with taxonomies?

The plugin doesn't currently support the **taxonomy form** location rule, so field groups used in this context cannot be saved to custom database tables at this time.

However, the plugin does support the ACF **taxonomy field**. The data from this field can be stored either as a serialised JSON string in a single column or it can be configured to create a relational table where each selected term has its own table row.

Taxonomy form location rule support is something we are open to adding should there be a demand for it. If you would like to request this feature, please let us know.

### What happens when I run the table update process?

The plugin parses all table definition (JSON) files and creates SQL table schema from the contained JSON. It then passes the SQL through WordPress' built-in `dbDelta()` function, which performs modifications to the database. The plugin also rebuilds the cached table map as part of this process.

This is a non-destructive process. It won’t delete columns or tables, but it will modify them where instructed to do so. You should always take a backup of your database before you run any sort of database schema modification, just to be safe.

### Will I lose any custom indexes I've set on my tables?

The table create/update process uses WordPress' internal `dbDelta()` function to process the table schema. `dbDelta()` does not currently remove indexes, so you shouldn't lost any custom indexes you have manually created on your table.

**Important:** As with anything that manipulates your data, you should take a full backup before running the procedure in case something does go wrong.

### Will columns be removed from my tables if I've removed them from the field group?

The table create/update process uses WordPress' internal `dbDelta()` function to process the table schema. `dbDelta()` does not currently remove columns, so you shouldn't lost any columns that you have removed from your table definition files.

**Important:** As with anything that manipulates your data, you should take a full backup before running the procedure in case something does go wrong.

### What happens if I rename an ACF field?

If you rename an ACF field, a new column will be added to the database table the next time you run the update process.

The plugin won’t know about this change of field name, so you will likely have issues with data retrieval on already saved fields. Any subsequent data updates will be saved to the new column.

If you need to do this on a live site, you’ll need to get existing data moved from the original column into the new column.

### Can I use complex fields such as repeaters?

Repeater fields are supported as of version 1.1. See the **Advanced Usage > Working with repeater fields** section for more information.

Support for other complex fields such as groups will ship in a later version. At this time, data from unsupported fields will pass through the plugin to be handled by ACF in the normal fashion.

### Will this make my database more efficient?

The plugin has the potential to make your database activity more efficient by reducing the amount of data you have in your post and user meta tables.

Additionally, the plugin has built in mechanisms that query and cache an entire row of data at a time in WPs object cache. This means that, when you use ACFs API functions – e.g; `get_field()` and `update_field()` – in a template file, only the first call to the function will call the database provided all other data in subsequent calls is from the same database table and for the same parent object (post, page, user). There is a slight overhead here, but we are currently working on ways to further improve the efficiency and performance of the plugin.

If using some custom SQL or some other custom search mechanism to search your custom tables, you should be able to yield faster results using well-organised tables than if you were searching the post meta table with lots of data.

### Are there any built in tools for querying custom tables?

We are currently working on tools to help developers query their custom tables. If you have requests or suggestions for tools/functionality you would like to see, please let us know.

### Will existing data be migrated across to custom tables?

We are currently working on a migration tool to handle just this. At present, existing data will remain in the core meta tables which the plugin will fall back to where custom table data does not exist. When you next save the post/object, the data will be saved to the appropriate custom tables.
