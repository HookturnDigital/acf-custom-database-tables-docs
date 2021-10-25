# Configuration via code

There are many possibilities for configuration via PHP code as demonstrated through these documents. Given the nature of
database configuration code, the ideal place to place any related configuration code would be a custom configuration
plugin. Code maintained in a plugin will persist across theme changes which is, in most cases, what you want.

An [MU plugin](https://wordpress.org/support/article/must-use-plugins/) is an ideal option as it isn't at risk of
accidental deactivation and WordPress will always load it.

## Can I put configuration code into a theme?

Whilst most of the snippets documented will function in a theme's `functions.php` file, it would be safer to go with a
custom configuration plugin.