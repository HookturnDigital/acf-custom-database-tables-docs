# Bypassing data storage in core meta tables

In this article:

---

By default, the plugin does not bypass storage of metadata to the core meta tables as doing so can create issue with other plugins that are dependent on meta data existing in the core meta tables. 

However, you may configure your site to bypass core tables where data is stored in a custom database table. This is broken down into two parts:

1. Bypassing field values
2. Bypassing field key references *(used internally by ACF to map field names to field keys)*

## Important considerations before choosing this option

If you decide to bypass data storage in core meta tables, it is important you bear the following in mind: 

1. You must be using ACF JSON if you decide to bypass the field key references as well as the field values. If you disable key reference storage, ACF won't be able to retrieve field values from any source. 
2. Many third party plugins expect — and even depend — on data in the core meta tables. Be sure you don't need to use these plugins or be prepared to implement some form of workaround should the need arise. 
3. Without meta data in the core meta tables, WordPress meta queries will no longer function. If you want to use custom table data in conjunction with `WP_Query`, see [this article.](https://hookturn.io/2019/09/how-to-use-acf-custom-database-tables-data-with-wp_query-objects/)

## How to bypass field values

To bypass storage of meta values where a custom table is found, use the following filter:

[https://gist.github.com/mishterk/8c9be3f25626de55a9b004e9b7defb28#file-acfcdt-plugin-bypass-values-in-core-meta-tables-php](https://gist.github.com/mishterk/8c9be3f25626de55a9b004e9b7defb28#file-acfcdt-plugin-bypass-values-in-core-meta-tables-php)

## How to bypass field key references

To bypass storage of ACF field key references for fields mapped to a custom database table, use the following filter:

**Note:** you should only do this if you are using ACF JSON. If your field groups aren’t represented in JSON files, you will have problems storing and retrieving data.

[https://gist.github.com/mishterk/8353e16ce3b5384dec988f63cd39e697#file-acfcdt-plugin-bypass-key-references-in-core-meta-tables-php](https://gist.github.com/mishterk/8353e16ce3b5384dec988f63cd39e697#file-acfcdt-plugin-bypass-key-references-in-core-meta-tables-php)