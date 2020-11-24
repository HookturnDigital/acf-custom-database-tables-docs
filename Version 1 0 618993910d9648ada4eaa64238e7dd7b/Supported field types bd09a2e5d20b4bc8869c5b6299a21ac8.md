# Supported field types

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