# Installation & Activation

1. Download the plugin zip file from your [Hookturn account page](https://hookturn.io/account).
2. Head to the **plugins** area of your website – **WP Admin > Plugins > Add New** – and upload the file.
3. Activate the plugin.

## Enabling ACF JSON

Whilst not a requirement, we highly recommend you consider enabling ACF Local JSON before creating any tables. ACF Local
JSON improves the performance of ACF and makes it possible for you to remove ACF’s key references from your core meta
tables.

If you decide to bypass both **values** and **field key references** you **must** have ACF JSON enabled and have all
field groups saved in JSON format in order for ACF to work correctly. Without field key references in the database, ACF
relies on the ACF JSON field groups to determine field mappings. Without those, data will not be stored correctly.

Enabling ACF JSON is as simple as creating an `acf-json` directory inside your theme directory.
See [https://www.advancedcustomfields.com/resources/local-json/](https://www.advancedcustomfields.com/resources/local-json/)
for further information.

With ACF JSON enabled, the plugin will store all custom database table definitions inside the `acf-json/database-tables`
directory.

## Activating complex field support

If you intend to store ACF field values from complex fields (i.e; repeater fields) in custom database tables, you need
to enable support for these fields.
See [how to enable repeater field support](../Advanced%20Usage/Working%20with%20repeater%20fields.md#how-to-enable-repeater-field-support)
.