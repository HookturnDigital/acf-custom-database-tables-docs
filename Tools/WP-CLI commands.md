# WP-CLI commands

ACF Custom Database Tables ships a set of WP-CLI commands under the `wp acfcdt` namespace for
inspecting your table map and diagnosing configuration problems without the admin UI. They're handy on
headless servers, in CI pipelines, and when working with support.

All commands require ACF to be installed and active. If ACF is missing, `wp acfcdt doctor` will
tell you exactly that; every other command is unavailable until ACF is active.

## Exit codes

Every command follows the same convention, so you can use them as CI gates:

- `0`: the command ran and everything is healthy (or data was exported successfully).
- `1`: the command ran and found a problem (a failing check, a missing cache file, an unmapped field). WP-CLI also exits `1` when it rejects the arguments themselves, such as a missing argument or an unknown subcommand.
- `2`: the command couldn't produce a result from the input it was given, such as a selector the plugin doesn't support.

## Output formats

`map status`, `map check` and `tables list` accept `--format=<format>` with `table` (default), `json`,
`csv` and `yaml`. `doctor` accepts `table` and `json`.

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

## `wp acfcdt definitions export`

Prints your table definition JSON to stdout.

```bash
wp acfcdt definitions export
wp acfcdt definitions export > definitions-backup.json
wp acfcdt definitions export | jq .
```

A single definition file exports as its own JSON document; multiple files export as a JSON
object keyed by file basename. Exits `1` if no definition files are found.
