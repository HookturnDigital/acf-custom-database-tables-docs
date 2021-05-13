# Globally activate sub tables

It is possible to set a global setting that will enable sub tables on all eligible fields. At the time of writing, the only fields eligible for sub tables are repeaters.

We don’t recommend this as you will have a very large number of database tables but, if you need to, you can use the following:

```php
<?php
// This will enable sub tables on all eligible fields globally. This affects
// table definition generation and can go in your functions.php file or a plugin.
add_filter( 'acfcdt/settings/enable_sub_tables_globally', '__return_true' );
```