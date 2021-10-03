# WP All Import plugin compatibility

As of version 1.1.0, it's possible to use **WP All Import** in conjunction with **ACF Custom Database Tables** to import
data into custom database tables.

This has been tested using all supported fields — including repeaters — and is working when importing both user and post
objects. All table formats — including sub (repeater) tables and join tables — appear to be receiving data as expected.

## Important — please read

It's important to note that this compatibility layer is complex and may require finessing as the **WP All Import**
plugin evolves. If you notice anything unusual when importing data, please let us know so that we can investigate and,
if possible, deploy a patch.

## Requirements

- The **Advanced Custom Fields** plugin must be installed and activated
- The **WP All Import** plugin must be installed and activated
- The **WP All Import Advanced Custom Fields Add-on** plugin must be installed and activated
- The **ACF Custom Database Tables** plugin (version 1.1.0 or greater) must be installed and activated
- The **WP All Import Plugin Compatibility** layer must be enabled in **Custom Fields > Database Tables > Settings >
  Compatibility**

## About the compatibility layer

At the time of writing, the **WP All Import** plugin's import process does not use ACF's built-in API functions to
handle data. This is a problem for any third-party plugin that intercepts ACF's functions to handle the data in any way
on its way into the database. Fortunately, the **WP All Export** plugin uses ACF's functions correctly when exporting
data so exports are unaffected.

To facilitate importing, the **ACF Custom Database Tables** plugin contains a built-in compatibility layer which, when
enabled, runs some additional functionality during import. The additional functionality fires the necessary hooks and
filters required by **ACF Custom Database Tables** to move data into the correct tables. This also facilitates core meta
bypassing if/when enabled.

## How to enable the WP All Import compability layer

1. In the WordPress admin, Head to **Custom Fields > Database Tables > Settings**
1. In the **Compatibility** section, check the box next to the **Enable WP All Import plugin compatibility** option.
1. Click the **Save Changes** button to save your configuration.

## Considerations, issues, problems, etc

These issues may well be problems without custom tables in play, but I want to share anything I noticed that may benefit
you when planning out your import/export configuration.

### Join table imports may produce errors if/when two posts are in the same relationship field with the same name

When exporting relationship fields, WP All Export may export a character separated list of **post titles**. e.g;
`My first post|My second post`. If you happen to have two posts with the **same title**, you may run into issues when
importing the relationship field.

Whilst testing, I noticed that a relationship field that was configured to use a join table failed to store any data
when there were two posts with identical post titles.

A possible workaround here (I haven't tested this) could be to use a custom function during export that exports the ID
instead of the post title.

### The default separator charactor for repeaters can be inconsistent between the import and export plugins

The release notes for version 1.6.5 of **WP All Export** indicate this may no longer be an issue but just be mindful
that you need to check the separating character when importing repeater data to ensure it matches the data. If it
doesn't, the data will not import correctly. 