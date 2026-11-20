# Deploying table changes

When you change a field group that has a custom table, two things need updating on every environment:
the table definition JSON file, and the database table itself. In the admin, that means saving the field
group and then running **Update Tables**. During a deployment, the [WP-CLI commands](WP-CLI%20commands.md)
do the same job without the admin.

## Before you start

**Run the commands as the site's user.** Use the same system user your web server runs WordPress as, not
`root` or a separate deploy user. The plugin only uses your ACF JSON folder when the current user can write
to it, so a different user can end up reading and writing a different definitions folder. Files created by
another user can also be unreadable to the site, which leaves your fields reading from core meta.

```bash
sudo -u www-data -H wp acfcdt map status --path=/path/to/wordpress
```

**Fix the definitions folder in `wp-config.php`.** Defining `ACFCDT_JSON_DIR` gives every user the same
folder. See [Changing the table definition directory](../Advanced%20Usage/Changing%20the%20table%20definition%20directory.md).

**Check the paths.** `wp acfcdt map status` shows the definitions folder and cache file the plugin is
using. `tables update` and `map rebuild` print both when they succeed. `definitions import` and
`definitions generate` print the files they wrote.

**Take a database backup** before `wp acfcdt tables update`.

## Choose a workflow

### Your definitions folder is in your repository

The definition files deploy with your code, so the only step is applying them:

```bash
wp acfcdt tables update --yes
```

### Your field groups sync, but your definitions don't

If your ACF field group JSON is in your repository but the table definitions aren't, generate the
definitions from the field groups first:

```bash
wp acfcdt definitions generate
wp acfcdt tables update --yes
```

`definitions generate` rewrites the definition files on every run, because each one records when it was
generated. If you commit your definitions, generate them on your own machine and commit the result
rather than generating them in CI.

### Your definitions only exist on another site

Sometimes the plugin can't use an ACF JSON folder (none exists, or it isn't writable) and no
`ACFCDT_JSON_DIR` constant or `acfcdt/json_dir` filter is set. In that case the definitions are saved to
`wp-content/uploads/acf-custom-database-tables`, which most teams don't commit. Definitions created on
staging then never reach your repository.

Export them from the site that has them and import them where you need them:

```bash
ssh staging "wp acfcdt definitions export" | wp acfcdt definitions import -
wp acfcdt tables update --yes
```

The import matches tables by name, so it updates the right file on the receiving site even when the two
sites gave it different file names. If the exporting site has only one definition file, the export has no
file name in it. Then pass `--file-name=<file-name>.json` to the import for a table the receiving site
doesn't have yet. The same pair works as a backup and restore:

```bash
wp acfcdt definitions export > definitions-backup.json
wp acfcdt definitions import definitions-backup.json
```

## An example deploy step

```bash
set -e
wp acfcdt definitions generate   # only if your definitions aren't in your repository
wp acfcdt tables update --yes
wp acfcdt doctor
```

`tables update` rebuilds and verifies the table map cache itself, so a separate `wp acfcdt map rebuild`
isn't needed. Every command exits non-zero when something goes wrong, so `set -e` stops the deploy at the
first failure.

## Licensing new environments

<!-- UNVERIFIED: the licence store decides this; the plugin always sends the stored key (verification pass, 2026-11-20). -->
A new environment, such as a staging copy or a short-lived CI site, gets no plugin updates until its
licence is activated. Activate it from the command line:

```bash
printf '%s' "$ACFCDT_LICENSE_KEY" | wp acfcdt license activate -
```

Piping the key in with `-` keeps it out of the server's process list and your shell history. Passing it
as an argument (`wp acfcdt license activate <key>`) also works, but other users on a shared server can see
it while the command runs.

<!-- UNVERIFIED: activation counting is the licence store's behaviour, not checkable in the plugin (verification pass, 2026-11-20). -->
Each activation uses one of your licence's site activations. When you tear an environment down,
deactivate it first:

```bash
wp acfcdt license deactivate || true
```

The `|| true` stops a failed deactivation from failing the teardown. If it does fail, the activation may
stay in use until you remove it from your account.

## `map rebuild` and the Rebuild Map System

The admin **Rebuild Map System** tool regenerates the definitions for the field groups you select (by
default, those that already have a definition file and a table) and rebuilds the map in the background. `wp acfcdt map rebuild` only rebuilds the table map cache from the definition
files that are already there, and it runs straight away. The command-line equivalent of the admin tool is
`wp acfcdt definitions generate` followed by `wp acfcdt map rebuild`.

## Extra output in your logs

PHP notices from WP-CLI, WordPress or another plugin can be printed to stdout alongside a command's
output. In a log that's only noise, but it breaks a piped export, because the import receives the notice
along with the JSON. If an import reports that its input isn't a JSON object, send PHP's errors to stderr
on the exporting side:

```bash
ssh staging "php -d display_errors=stderr \$(which wp) acfcdt definitions export" | wp acfcdt definitions import -
```
