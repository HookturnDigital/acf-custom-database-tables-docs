# Introduction

## What is this plugin?

The **ACF Custom Database Tables** plugin is an extension for the Advanced Custom Fields plugin. The extension makes it possible for developers to generate custom database tables that map to ACF Field Groups where each field represents a column. This approach provides the option to store key entity (post, user, custom post type) metadata as a single row of data.

Features include:

1. Table generation, creation, and modification from the WordPress admin
2. Filters to prevent particular fields from generating a column
3. Filters to disable core metadata storage where a custom table is available for that data
4. Filters to enable join tables for eligible ACF field types
5. Filters to remove support for particular fields

## Why would I use it?

The plugin offers a few key benefits:

1. Organised, structured data is more readily available for export and can be searched efficiently.
2. By moving data out of the WordPress core meta tables, these tables can be more efficient in their own operations.
3. Increased efficiency in database operations frees up system resources increasing scalability of your site/application.
4. Advanced use offers join tables for eligible fields opening up the possibility of relationship-oriented custom SQL queries.

## How does it work?

The basic process is as follows:

1. The plugin introduces an option on an ACF Field Group’s admin page the allows a site administrator to enable a custom database table for that field group.
2. On Field Group save, a JSON file is generated – this is the table definition.
3. On a subsequent, manually-triggered action, these JSON files are converted to SQL and are then used to create new database tables or modify existing ones. A table map is also generated as part of this process.
4. The plugin uses the generated map to intercept metadata as it is saved and retrieved from the database. If that piece of metadata has a custom table defined for the related object (post, user, custom post type), the data will be stored/retrieved to/from that custom table.

## Basic requirements

The basic requirements for the plugin to function are as follows:

1. PHP version 5.6 (version 7 is recommended)
2. WordPress version 4.9
3. ACF version 5.6.10

### **ACF Local JSON**

Whilst not a requirement, we highly recommend using ACF JSON. Doing so enables you to further reduce the amount of metadata in your core meta tables by disabling ACF field key reference storage.

Enabling ACF JSON is as simple as creating an acf-json directory inside your theme directory. See [https://www.advancedcustomfields.com/resources/local-json/](https://www.advancedcustomfields.com/resources/local-json/) for further information.

## Supported field types

The plugin currently supports all built-in field types except for the following:

1. Repeater
2. Group
3. Flexible Content
4. Clone

These fields are complex and support for these is coming in the form of **sub-tables**. Support for these field types will ship in an early 1.x release.

Where a field type is not supported, it will be ignored by the plugin and will be passed back to ACF to be handled as per usual (stored in core meta tables).

Layout fields such as **message**, **accordion**, and **tab** are not considered as they do not store data.

### **Custom Field Types**

There is some basic support in place for custom field types but these field types need to be enabled/registered through the use of a simple WordPress filter. See the advanced usage section for instructions on how to do this.

Advanced support for custom field types is currently being explored which may include the possibility of custom data processing on field type registration. This should land in a 1.x release.