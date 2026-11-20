# WP-CLI commands

ACF Custom Database Tables ships a set of WP-CLI commands under the `wp acfcdt` namespace. Some
inspect your table map and diagnose configuration problems. Others apply table definition changes,
rebuild the table map and manage the licence, all without the admin UI. They're handy on headless
servers, in CI pipelines, and when working with support.

To apply table changes as part of a deployment, see
[Deploying table changes](Deploying%20table%20changes.md).

All commands require ACF to be installed and active. If ACF is missing, `wp acfcdt doctor` will
tell you exactly that; every other command is unavailable until ACF is active.

## Exit codes

Every command follows the same convention, so you can use them as CI gates:

- `0`: the command ran and everything is healthy, or the change was made and checked.
- `1`: the command ran and found a problem (a failing check, a missing cache file, an unmapped field), or a change failed. WP-CLI also exits `1` when it rejects the arguments themselves, such as a missing argument or an unknown subcommand.
- `2`: the command couldn't produce a result from the input it was given, such as a selector the plugin doesn't support or an import file it can't use. A command that exits `2` hasn't changed anything.

The commands that make changes never exit `0` after a failed step. They check their own result (for
example, that the table map cache they wrote loads back correctly) before reporting success.

## Output formats

`map status`, `map check`, `tables list` and `license status` accept `--format=<format>` with `table`
(default), `json`, `csv` and `yaml`. `doctor` accepts `table` and `json`. The commands that make changes
print plain lines and have no `--format` option.

**A note for scripts capturing JSON output:** depending on your server's PHP `error_reporting`
configuration, WP-CLI itself can emit PHP deprecation notices into stdout. If your pipeline
captures command output, filter those lines before parsing. For example:

```bash
wp acfcdt map status --format=json | grep -Ev "^(PHP )?(Deprecated|Notice|Warning):|^$"
```

## `wp acfcdt doctor`

Runs a health check: environment versions, system requirement checks, the table map cache state,
working directory accessibility, and plugin settings.

```bash
wp acfcdt doctor
wp acfcdt doctor --format=json
```

Exits `0` when all checks pass and `1` when any check fails, so a deploy pipeline can gate on it:

```bash
wp acfcdt doctor || exit 1
```

The output is limited to plugin-relevant data (versions, debug flags, the plugin's own working
directories, and its settings), so it's safe to paste into a support ticket.

## `wp acfcdt map status`

Reports the state of the table map cache file and the currently loaded map: cache file presence
and readability, table count, and mapped field count.

```bash
wp acfcdt map status
wp acfcdt map status --format=json
```

This is the first thing to run when field values appear to be reading from core meta instead of
your custom tables. A missing or empty cache file means ACF reads/writes fall back to core meta
silently. `map status` makes that state loud, with an exit code of `1` and a message naming the
fix.

## `wp acfcdt map check <field_name> <selector>`

Checks whether a specific ACF field resolves to a custom table in a specific context.

```bash
wp acfcdt map check my_field 123      # post ID 123
wp acfcdt map check my_field user_3   # user ID 3
```

The output echoes the decoded selector (type, ID, resolved context) alongside the field lookup
result and resolved table name, so a mistyped selector is immediately visible.

Custom tables support post and user contexts only. Passing `option` or `term_<id>` selectors
reports that explicitly (exit `2`) rather than a misleading "not mapped".

Exits `0` when the field is mapped, `1` when it isn't.

## `wp acfcdt map rebuild`

Rebuilds the table map cache from your table definition JSON files, then loads the new cache back and
checks it.

```bash
wp acfcdt map rebuild
```

Use it when `map status` reports a missing, empty or unreadable cache file. It prints the definitions
folder and cache file it used, so you can confirm it read the right place.

It refuses to replace the cache when it finds no definition files. That usually means the definitions
folder resolved somewhere else for the user running the command (see
[Deploying table changes](Deploying%20table%20changes.md)), and an empty map would send every field back
to core meta.

`map rebuild` doesn't regenerate definitions from your field groups, and it doesn't create or alter
tables. That makes it narrower than the admin **Rebuild Map System** tool. To regenerate definitions,
run `wp acfcdt definitions generate` first. To apply them to the database, run `wp acfcdt tables update`,
which rebuilds the cache as well.

Exits `0` when the cache was rebuilt and verified, and `1` on any failure.

## `wp acfcdt tables list`

Lists the tables currently loaded in the map, each table's type (`primary`, `join`, or `sub`),
and the JSON definition file that owns it.

```bash
wp acfcdt tables list
wp acfcdt tables list --format=json
```

Join and sub tables are defined within their parent table's definition file and are attributed
to it. A table that's in the map but has no definition file in the current JSON directory is
flagged, which usually means the map cache is stale.

## `wp acfcdt tables update [--yes]`

Creates and updates your custom tables from the table definition JSON files. It's the command-line
equivalent of the **Update Tables** tool in the admin.

```bash
wp acfcdt tables update          # asks you to confirm you have a backup
wp acfcdt tables update --yes    # no prompt, for scripts and CI
```

This modifies your database, so take a backup first. It asks for confirmation unless you pass `--yes`.
When it isn't run interactively (in a CI job, for example), `--yes` is required, and the command exits
`2` without changing anything if it's missing.

After the tables are updated, it checks that every table and column in your definitions now exists in the
database. WordPress's `dbDelta()` function doesn't report a change the database rejected, so without this
check a failed update could look like a success. If the check passes, the command rebuilds the table map
cache and verifies it. If the check fails, the cache is left as it was, and the command lists the missing
tables and columns.

Columns that are no longer in your definitions are left in place. Tables are never dropped.

Exits `0` when the tables and the cache are up to date, and `1` on any failure or when you decline the
prompt.

## `wp acfcdt definitions export`

Prints your table definition JSON to stdout.

```bash
wp acfcdt definitions export
wp acfcdt definitions export > definitions-backup.json
wp acfcdt definitions export | jq .
```

A single definition file exports as its own JSON document; multiple files export as a JSON
object keyed by file basename. Exits `1` if no definition files are found.

## `wp acfcdt definitions generate [<field-group-key>...]`

Writes table definition JSON files from the field groups that have a custom table enabled. It uses the
same process as saving a field group in the admin.

```bash
wp acfcdt definitions generate                         # every field group with a custom table
wp acfcdt definitions generate group_5f1a2b3c4d5e6     # just these field groups
```

Run it when your field groups have changed without being saved in the admin, such as after an ACF JSON
sync during a deployment. Then run `wp acfcdt tables update` to apply the definitions. `generate` doesn't
change any tables or the table map cache.

Each definition is written to the file name stored on its field group, so running it again updates the
same files. A field group with no stored definition file name is skipped with a warning, because each run
would otherwise create another definition file for the same table. Saving the field group once in the
admin stores a file name.

A field group is also skipped when a field group saved in the admin already uses its table name or its
definition file name. These are the same checks the admin makes when you save a field group. Field groups
that exist only as ACF JSON or PHP are not compared with each other.

Exits `0` when every definition was written, `1` when any field group failed or was skipped, and `2`
when a key you passed isn't a field group with a custom table enabled. Nothing is written on exit `2`.

## `wp acfcdt definitions import <file> [--file-name=<file-name>]`

Imports table definition JSON into the definitions folder. It takes the output of
`wp acfcdt definitions export`, from a file or, with `-`, from STDIN.

```bash
wp acfcdt definitions import definitions-backup.json
ssh staging "wp acfcdt definitions export" | wp acfcdt definitions import -
```

Use the export and import pair to move definitions between environments when your definitions folder
isn't in your repository (see
[Deploying table changes](Deploying%20table%20changes.md)), or to restore a backup.

Every definition is checked before anything is written. Definitions are held to what the plugin generates
itself: table, column and key names made of letters, numbers, `$` and underscores, the usual column
formats, a post or user relationship, and no semicolons or backticks anywhere. If any definition can't be
used, nothing is written and the command exits `2`.

Each environment generates its own definition file names, so the same table often has a different file
name on staging and production. When a table in the import is already defined in the definitions
folder, the import writes over that existing file, whatever the import calls it. A new table uses the file
name from the import. A single exported definition carries no file name, so importing a new table from
one needs `--file-name`:

```bash
wp acfcdt definitions import table.json --file-name=table_books.json
```

File names must be plain names ending in `.json`. A file name the import would have to change is
rejected rather than renamed.

If the definitions folder doesn't exist yet, the import creates it and protects it the same way the
plugin does when it saves a field group.

`import` doesn't change any tables or the table map cache. Run `wp acfcdt tables update` next.

Exits `0` when the definitions were imported, `1` when a file couldn't be written, and `2` for input that
can't be imported.

## `wp acfcdt license activate [<key>]`

Activates your licence for this site, so it receives plugin updates.

```bash
wp acfcdt license activate                                    # the key already stored on this site
printf '%s' "$ACFCDT_LICENSE_KEY" | wp acfcdt license activate -   # read the key from STDIN
```

<!-- UNVERIFIED: activation counting is the licence store's behaviour, not checkable in the plugin (verification pass, 2026-11-20). -->
When you pass a key, it's stored on the site first, replacing any existing key. Each activation uses one
of your licence's site activations.

A key passed as an argument is visible to other users in the server's process list. On a shared server,
pass `-` and pipe the key in, which also keeps it out of your shell history.

Exits `0` when the licence server confirms the licence is valid, `1` when it refuses (an expired key, no
activations left) or can't be reached, and `2` when no key is given or stored.

## `wp acfcdt license deactivate`

Deactivates your licence for this site, which frees up the activation.

```bash
wp acfcdt license deactivate
```

Exits `0` when the licence server confirms the deactivation, `1` when it doesn't or can't be reached,
and `2` when no key is stored.

## `wp acfcdt license status`

Shows whether a licence key is stored and the licence status recorded at the last activation. It reads
what's stored on this site and doesn't contact the licence server. The key itself is never shown.

```bash
wp acfcdt license status
wp acfcdt license status --format=json
```

Exits `0` when the stored status is valid, and `1` otherwise.
