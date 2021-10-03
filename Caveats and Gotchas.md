# Caveats & Gotchas

1. You may face issues with third party systems that rely on WordPress' core table structure (e.g; search plugins,
   database management plugins, etc.) If you find situations where this is a problem, please let us know so that we can
   investigate solutions.
1. If a table definition does not contain any columns, the table will not be created.
1. In cases where ACF JSON was enabled after field groups were created, you will need to save the field group again in
   order for ACF JSON to create the field group's JSON file. Without the field group's JSON file, data won't be handled
   correctly.