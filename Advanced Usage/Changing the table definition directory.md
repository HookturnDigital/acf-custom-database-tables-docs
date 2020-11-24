# Changing the table definition directory

The plugin will attempt to automatically generate a directory to house all related files when an ACF Field Group is saved and the table generation on that field group is active. The possible locations for this directory are:

1. If using ACF JSON, the `acf-json/database-tables` directory will be created.
2. If no acf-json directory is available, the plugin will attempt to create `uploads/acf-custom-database-tables` directory.

It is possible, however, for you to customise the location and name of this directory using either of the following options:

## Using a PHP constant

You can define the following PHP constant in your `wp-config.php` file or in a configuration plugin, provided it is defined before the `plugins_loaded` action hook:

[https://gist.github.com/mishterk/208594cdcb164e265a21dff0cfa500fb#file-acfcdt-plugin-changing-the-json-directory-by-php-constant-php](https://gist.github.com/mishterk/208594cdcb164e265a21dff0cfa500fb#file-acfcdt-plugin-changing-the-json-directory-by-php-constant-php)

Using this approach is definitive and the directory path specified will not be passed to the filter for override. You may need to create this directory yourself and make sure it is writable. See the diagnostic information available in **Custom Fields > Database Tables > Help** if you are having issues around this.

## Using a filter

If you prefer, you can use the following filter to modify the JSON directory:

[https://gist.github.com/mishterk/8e30187dec5c98e08b1d60d35159f1f0#file-acfcdt-plugin-changing-the-json-directory-by-filter-php](https://gist.github.com/mishterk/8e30187dec5c98e08b1d60d35159f1f0#file-acfcdt-plugin-changing-the-json-directory-by-filter-php)

Again, this code needs to run before the plugins_loaded action hook fires, so this needs to go in a configuration plugin.