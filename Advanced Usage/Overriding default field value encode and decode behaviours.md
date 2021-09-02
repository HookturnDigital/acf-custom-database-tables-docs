# Overriding default field value encode and decode behaviours

When storing a value in a database table, the value cannot be an array or object as these will result in database
errors. To protect against this, WordPress' core meta system attempts to serialize all values before storing them in the
core tables. This allows us to pass non-scalar values (arrays, objects) to functions such as `update_meta()` and its
derivatives. ACF builds upon these core meta functions and so, by extension, supports passing of non-scalar values to
`update_field()`.

## Custom database tables are JSON encoded instead of serialized

In the core meta tables, non-scalar values are serialised but in ACF Custom Database Tables, we use JSON encoding as the
JSON format is more readable, portable, and so lends itself to a wider use case. In line with core behavour, the plugin
will, by default, attempt to encode data before storing it and decode it after retrieving it from the database.

This variation in format, however, does result in some small discrepancies in how data is handled for certain edge
cases.

## The most likely example

One such example is storing a JSON-encoded string in an ACF text field. If you pass a JSON encoded string — e.g;
`{"my":"simple", "json":"object"}` — to `update_field()`, both the core and custom tables will see it as a string and
store it as such. Both the core and custom tables have the same value stored.

The variation occurs in the output when using `get_field()` to retrieve the field value. If the value is returned from a
core table, it will return as the same encoded string that was saved — the core meta system only tries to deserialize
data, not decode JSON.

If the value is returned from a custom database table, the plugin will decode any encoded JSON resulting in an array
instead of the encoded JSON string. This is the expected behaviour and closely matches what happens in core but it can
be problematic where you might be expecting to receive encoded JSON strings. It can also be an issue in the admin as the
expected value is a string, not an array.

For this reason, there are a number of filters available to facilitate specific use-cases.

## Customising the the encode/decode handling for custom table data

There are filters available to allow the following points of control:

1. Disable encoding.
2. Disable decoding.
3. Replace the encoding strategy.
4. Replace the decoding strategy.
5. Customise the arguments used when JSON encoding values.

Note that all of the filters involved have access to the ACF field array so it is possible to apply the logic to a
specific field, field type, globally, etc.

### How to prevent encoding of an ACF field value

You may use the `acfcdt/field/should_encode_value` to return a boolean value to control whether or not a field's value
will be passed to the field value encoder. e.g;

```php
add_filter( 'acfcdt/field/should_encode_value', function( $bool, $field, $value ){
    if( $field['name'] === 'my_custom_field' ){
        return false;
    }
    
    return $bool;
}, 10, 3);
```

### How to prevent decoding of an ACF field value

You may use the `acfcdt/field/should_decode_value` to return a boolean value to control whether or not a field's value
will be passed to the field value encoder for decoding. e.g;

```php
add_filter( 'acfcdt/field/should_decode_value', function( $bool, $field, $value ){
    if( $field['name'] === 'my_custom_field' ){
        return false;
    }
    
    return $bool;
}, 10, 3);
```

### How to customise the encoding strategy

If you prefer not to use JSON encoding, you may customise the logic used to encode data using the
`acfcdt/filter_value_before_encode` filter. By returning a value that is not an object or an array, the field value
encoder will stop processing the value. e.g;

```php
add_filter( 'acfcdt/filter_value_before_encode', function( $value, $field ){
    if( $field['name'] === 'my_custom_field' ){
        $value = join( ',', $value ); // maybe use a comma-separated string?
    }

    return $value;
}, 10, 2);
```

**NOTE:** When using a custom encoding strategy, it's imperitive you also customise the decoding strategy to match.

#### How to customise the decoding strategy

If using a custom encoding strategy, you should also implement the reverse handling to ensure data is decoded
appropriately. You can do this using the `acfcdt/filter_value_before_decode` filter. By returning a value that is not a
string, the field value encoder will stop decoding the value. e.g;

```php
add_filter( 'acfcdt/filter_value_before_decode', function( $value, $field ){
    if( $field['name'] === 'my_custom_field' ){
        $value = explode( ',', $value );
    }

    return $value;
}, 10, 2);
```

### Controlling the arguments used when JSON encoding ACF field values

The built-in JSON encode handler uses WordPress' core `wp_json_encode()` function to encode the data. Some assumptions
have been made which might not suit all use cases so for fine control, we've made both the options and depth
configurable via the following filters.

#### Controlling the `options` parameter

By default, the plugin uses the `JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE` options when encoding data. You may
customise this using the `acfcdt/filter_json_encode_bitmask` filter. e.g;

```php
add_filter( 'acfcdt/filter_json_encode_bitmask', function( $options, $field ){
    return JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE;
}, 10, 2);
```

For a complete list of available options, see the possible values in
the [`flags` parameter of the `json_encode()` function documentation](https://www.php.net/manual/en/function.json-encode.php)
.

#### Controlling the `depth` parameter

By default, the plugin uses the default depth of 512. You may customise this using the `acfcdt/filter_json_encode_depth`
filter. e.g;

```php
add_filter( 'acfcdt/filter_json_encode_depth', function( $depth, $field ){
    return 256;
}, 10, 2);
```