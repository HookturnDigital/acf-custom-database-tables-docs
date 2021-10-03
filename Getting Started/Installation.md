# Installation & Activation

1. Download the plugin zip file from the [Hookturn website](https://hookturn.io/).
2. Head to the **plugins** area of your website – e.g; https://example.com/wp-admin/plugins.php – and upload the file.
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