# Deactivating routing to or from a custom table

If you've decided you no longer want a particular custom database table to participate in ACF read/write routing — and instead want all data for those fields to flow through the core meta tables — the table can be removed from the plugin's map system. Once it's out of the map, the plugin no longer intercepts reads and writes for those fields, and ACF behaves as it would without ACF Custom Database Tables installed for that field group.

This is the permanent path. For temporary, programmatic toggling of the read/write intercepts at runtime, see [Disabling storage or retrieval to and from custom database tables](./disabling-storage-or-retrieval-to-and-from-custom-database-tables/) instead.

## 1. Disable table definition management on the field group

Open the field group in the ACF admin and set **Manage Table Definition** to **false**, then save the field group.

This toggle controls whether saving the field group will update the table's schema blueprint. It does not, on its own, stop data being routed to or from the custom table — that's what the remaining steps are for. Turning it off first prevents the table definition being regenerated when you save changes elsewhere.

## 2. Delete the table's JSON definition file

The plugin builds its routing map from the JSON files that describe each custom table. Removing the JSON file for the table you want to retire is what takes it out of the map.

The file lives in one of two locations:

- **If ACF JSON is enabled:** `acf-json/database-tables/` inside your theme.
- **Otherwise:** `wp-content/uploads/acf-custom-database-tables/`.

If you're unsure which applies, the path shown in the create/update tables file list on the **Manage Tables** screen will tell you. Find the file whose contents reference the target table name and delete it.

## 3. Rebuild the table map

Head to **Custom Fields > Database Tables > Manage Tables** and run the create/update tables process.

This rebuilds the routing map from the remaining JSON definitions. With the deleted file gone, the retired table is no longer in the map, and ACF will read and write its values to the core meta tables from this point on. The custom table itself is left in place — drop it manually if you no longer need the data.

## A note on existing data

Removing a table from the map doesn't migrate data out of it. If the fields had previously been writing only to the custom table (for example, with `acfcdt/settings/store_acf_values_in_core_meta` set to `false`), the core meta tables will be empty for those fields and reads will return nothing until the data is re-saved. Re-save the relevant posts, or run a one-off migration, to repopulate core meta before relying on it.
