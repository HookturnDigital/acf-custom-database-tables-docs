# Activating join tables on eligible fields

Join tables are available where required for eligible fields and can be activated using filters. The following fields are eligible for join tables:

1. Relationship
2. Post Object
3. Page Link
4. User
5. Taxonomy

## Global activation

It is possible to set a global setting that will enable join tables on all eligible fields. We don’t recommend this, as you will have a very large number of database tables, but if you need to, you can use the following:

[https://gist.github.com/mishterk/dfd8125bb84b764e7f4fafc4ef3fb62d#file-acfcdt-plugin-global-join-table-activation-php](https://gist.github.com/mishterk/dfd8125bb84b764e7f4fafc4ef3fb62d#file-acfcdt-plugin-global-join-table-activation-php)

## Activation by table name, field type, and/or field name

[https://gist.github.com/mishterk/33c7f7e246801842abfdc90011d9dc61#file-acfcdt-plugin-join-table-activation-php](https://gist.github.com/mishterk/33c7f7e246801842abfdc90011d9dc61#file-acfcdt-plugin-join-table-activation-php)

## Activation for all eligible fields configured to accept multiple values

If you prefer, you can use the ACF field array options to enable join tables on fields that match specific conditions. For example, you may wish to enable join tables on all eligible fields that accept multiple values:

[https://gist.github.com/mishterk/a294336c2bb97d04be2c1424e42ab3ba#file-acfcdt-plugin-activate-join-tables-on-fields-with-multiple-values-php](https://gist.github.com/mishterk/a294336c2bb97d04be2c1424e42ab3ba#file-acfcdt-plugin-activate-join-tables-on-fields-with-multiple-values-php)

Note: This approach should only be used when you know field’s settings are concrete.