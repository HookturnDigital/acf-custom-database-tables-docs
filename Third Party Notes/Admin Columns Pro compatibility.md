# WP All Import plugin compatibility

After some simple tests, we found that the Admin Columns Pro plugin can be used in conjunction with data handled by ACF
Custom Database Tables with some caveats to be aware of.

Displaying and inline-editing field values works fine. Filtering and sorting, however, is currently not possible due to
the internal mechanisms that drive this functionality. There is currently a dependency on metadata existing in core meta
tables for Admin Columns Pro to understands how to filter and sort records.

We'll be working on improving this functionality in a future release and it will likely ship as an opt-in compatibility
layer.

For now, if you need to sort/filter using Admin Columns Pro, it's best to continue storing the required data in core
meta tables. As of version 1.1, you can selectively choose which fields are bypassed from core meta tables.
See [targeting specific fields](../Advanced%20Usage/Bypassing%20data%20storage%20in%20core%20meta%20tables.md#targeting-specific-fields-recommended)
.