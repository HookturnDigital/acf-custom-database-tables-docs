# Query utilities and helpers

The plugin does not ship dedicated query helper functions. Because your field data lives in structured, normalised tables, you query it the same way you'd query any other custom table in WordPress — through ACF's API, through `$wpdb`, or through `WP_Query` with custom clauses. Each approach has a place, and the right one depends on what you're trying to do.

## Querying through ACF's API

The plugin intercepts calls to ACF's standard field-access functions and resolves them against the custom tables transparently. For everyday template work — reading a single field, looping a repeater, fetching a flexible content layout — you don't need to touch SQL.

```php
<?php
// Resolves against the custom table for the post's field group.
$value = get_field( 'my_field', $post_id );

// Same for output.
the_field( 'my_field', $post_id );

// And for writes.
update_field( 'my_field', 'new value', $post_id );
```

The advantage of this path is that your template code is identical whether or not custom tables are enabled — switching storage modes does not require code changes.

The trade-off is that ACF's API reads one record at a time. For listing, searching, or filtering across many rows, you want SQL.

## Querying directly with `$wpdb`

Because the data is stored in regular database columns, the WordPress `$wpdb` instance is the most direct way to query across many rows or fields. This is where the performance benefit of custom tables shows up most clearly — particularly when filtering on multiple field values at once.

```php
<?php
global $wpdb;

$table = $wpdb->prefix . 'my_custom_table';

$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT post_id, my_field, another_field
     FROM {$table}
     WHERE my_field = %s
       AND another_field > %d",
    'some value',
    100
) );
```

Always run user-supplied values through `$wpdb->prepare()`. The table name is whatever you configured in the field group's **Custom Table Name** setting, prefixed with `$wpdb->prefix`.

## Fetching a list of posts with `post__in`

When you want a list of *posts* — not raw column values — filtered or sorted by data held in a custom table, one option is a two-step pattern. First, run a SQL query against the custom table to collect the IDs of the matching posts. Then hand that array of IDs to `WP_Query` through the `post__in` argument. You get back fully-formed `WP_Post` objects and the standard template loop, while the filtering happens in a query against your custom table rather than a stack of meta queries.

The appeal of this pattern is that it bolts onto an ordinary `WP_Query` without you having to modify the query's joins or where-clauses, which makes it the most approachable of the SQL-backed options. It also hands you direct control over pagination: because you set the `LIMIT` on the ID query, the array passed to `post__in` stays small no matter how many rows match, and you can fetch a total with a cheap `COUNT(*)` on the custom table. Measured against an equivalent `meta_query` on `wp_postmeta`, it's dramatically faster — a lookup that took several seconds as a stacked meta query can drop to around a second. Against the join approach below it's a closer call; see [Which approach should I use?](#which-approach-should-i-use) at the end of this page.

```php
<?php
global $wpdb;

$table = $wpdb->prefix . 'my_custom_table';

// 1. Collect the matching post IDs straight from the custom table.
$post_ids = $wpdb->get_col( $wpdb->prepare(
    "SELECT post_id
     FROM {$table}
     WHERE date_start >= %s
       AND location = %s
     ORDER BY date_start ASC",
    current_time( 'Ymd' ),
    'some location'
) );

// 2. Only run WP_Query if we actually matched something (see the note below).
if ( $post_ids ) {
    $query = new WP_Query( [
        'post_type'      => 'any',
        'post__in'       => $post_ids,
        'orderby'        => 'post__in',
        'posts_per_page' => 4,
    ] );

    if ( $query->have_posts() ) {
        while ( $query->have_posts() ) {
            $query->the_post();
            // Standard template tags work here — the_title(), get_field(), etc.
        }
        wp_reset_postdata();
    }
}
```

Two things worth knowing:

- **Guard the empty case.** An empty array passed to `post__in` is ignored by `WP_Query`, so it returns the full, unfiltered post list rather than nothing at all. The `if ( $post_ids )` check above prevents that surprise.
- **Preserve your SQL ordering with `'orderby' => 'post__in'`.** This tells `WP_Query` to return the posts in the same order as your ID array, so the `ORDER BY` you applied in SQL carries through to the rendered list. Without it, `WP_Query` falls back to its default ordering and your sort is lost.

For the full end-to-end walk-through, see [How to use ACF Custom Database Tables data with WP_Query objects](https://hookturn.io/how-to-use-acf-custom-database-tables-data-with-wp_query-objects/).

## Joining custom tables onto `WP_Query`

When you want to filter or sort a standard post query by values stored in a custom table — for example, "show me all `event` posts where `start_date` is in the future, ordered by `start_date`" — `WP_Query`'s `posts_join`, `posts_where`, and `posts_orderby` filters let you splice in the join.

```php
<?php
add_filter( 'posts_join', 'xyz_join_event_table', 10, 2 );
function xyz_join_event_table( $join, $query ) {
    global $wpdb;

    if ( $query->get( 'xyz_join_events' ) ) {
        $table  = $wpdb->prefix . 'my_events_table';
        $join  .= " LEFT JOIN {$table} ON {$table}.post_id = {$wpdb->posts}.ID ";
    }

    return $join;
}

add_filter( 'posts_where', 'xyz_filter_event_table', 10, 2 );
function xyz_filter_event_table( $where, $query ) {
    global $wpdb;

    if ( $query->get( 'xyz_join_events' ) ) {
        $table  = $wpdb->prefix . 'my_events_table';
        $where .= $wpdb->prepare( " AND {$table}.start_date >= %s ", current_time( 'mysql' ) );
    }

    return $where;
}

$events = new WP_Query( [
    'post_type'        => 'event',
    'xyz_join_events'  => true,
    'orderby'          => 'meta_value',
    'order'            => 'ASC',
] );
```

The custom query var (`xyz_join_events` in the example) gates the join so the filters only apply to the queries that need them. Without that guard, every `WP_Query` on the site would pick up the join.

For a working snippet covering joins, where clauses, and ordering against a custom table, see [this Gist](https://gist.github.com/mishterk/04a20b60addddb3878db17531d40a6fe).

## Which approach should I use?

The `post__in` pattern and the `WP_Query` join read from the same indexed custom table, so for most result sets the difference is small and either is a sound choice. When it does start to matter, a few things decide it:

- **Indexing matters more than the choice between the two.** The plugin indexes only the `id` and `post_id` columns, so any column you filter or sort on should have its own index — you add these yourself, and they persist across table updates. Without the right index neither approach performs well; with it, both do.
- **For large, filtered-and-sorted lists, a single join tends to be faster.** It's one round trip and one query plan, and the join back to `wp_posts` runs against the unique `post_id` key. The `post__in` pattern has to carry the full set of matching IDs back into PHP and re-embed them as an `IN (…)` list, which becomes the bottleneck once you're matching thousands of rows at once.
- **For paginated lists, `post__in` stays cheap and simple.** Putting the `LIMIT` on the ID query keeps the `IN (…)` list small, and a `COUNT(*)` on the custom table gives you the total without `WP_Query`'s default `SQL_CALC_FOUND_ROWS` running across the whole joined set.

If you're unsure, pick whichever reads more clearly for your case, and reach for `EXPLAIN` if a specific query turns into a hot spot.

## When you're new to custom SQL

If hand-written SQL inside WordPress is unfamiliar territory, start with the primer [Custom WordPress SQL queries for beginners](https://hookturn.io/custom-wordpress-sql-queries-for-beginners/). It covers `$wpdb`, `prepare()`, and the common patterns used in the examples above.

## A note on hyphens in field names

If your ACF field names contain hyphens, remember that the database column names use underscores. When querying through ACF's API you continue to use the hyphenated name; when querying directly via SQL you must use the underscored column name. See [Using hyphens in field names](./using-hyphens-in-field-names/) for the full detail.
