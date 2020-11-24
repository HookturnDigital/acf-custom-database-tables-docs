# What is this plugin?

The **ACF Custom Database Tables** plugin is an extension for the Advanced Custom Fields plugin. The extension makes it possible for developers to generate custom database tables that map to ACF Field Groups where each field represents a column. This approach provides the option to store key entity (post, user, custom post type) metadata as a single row of data.

Features include:

1. Table generation, creation, and modification from the WordPress admin
2. Filters to prevent particular fields from generating a column
3. Filters to disable core metadata storage where a custom table is available for that data
4. Filters to enable join tables for eligible ACF field types
5. Filters to remove support for particular fields