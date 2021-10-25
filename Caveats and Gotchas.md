# Caveats & Gotchas

1. If choosing to [bypass core meta tables](Advanced%20Usage/Bypassing%20data%20storage%20in%20core%20meta%20tables.md),
   you may face issues with third party systems that rely on WordPress' core table structure (e.g; search plugins,
   database management plugins, etc.) If you find situations where this is a problem, please let us know so that we can
   investigate solutions.
2. If a table definition does not contain any columns, the table will not be created.
3. In cases where ACF JSON was enabled after field groups were created, you will need to save the field group again in
   order for ACF JSON to create the field group's JSON file. Without the field group's JSON file, data won't be handled
   correctly.
4. The use of hyphens in field names is supported but there are some considerations to be aware of.
   See [Using hyphens in field names](References/Using%20hyphens%20in%20field%20names.md)
5. Fields with the field name `id`, `post_id`, `user_id`, and/or `_sort_order` will result in errors as these are
   currently hard-coded, internal column names used by the plugin.