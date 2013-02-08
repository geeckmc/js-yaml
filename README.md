JS-YAML - YAML 1.2 parser and serializer for JavaScript
=======================================================

[![Build Status](https://secure.travis-ci.org/nodeca/js-yaml.png)](http://travis-ci.org/nodeca/js-yaml)

[Online Demo](http://nodeca.github.com/js-yaml/)


This is an implementation of [YAML](http://yaml.org/), a human friendly data
serialization language. Started as [PyYAML](http://pyyaml.org/) port, it was
completely rewritten from scratch. Now it's very fast, and supports 1.2 spec.


Breaking changes in 1.x.x -> 2.0.x
----------------------------------

If your have not used __custom__ tags or loader classes - no changes needed. Just
upgrade library and enjoy high parse speed.

In other case, you should rewrite your tag constructors and custom loader
classes, to conform new schema-based API. See
[examples](https://github.com/nodeca/js-yaml/tree/master/examples) for details.
Note, that parser internals were completely rewritten.


Installation
------------

### YAML module for node.js

```
npm install js-yaml
```


### CLI executable

If you want to inspect your YAML files from CLI, install js-yaml globally:

```
npm install js-yaml -g
```

#### Usage

```
usage: js-yaml [-h] [-v] [-c] [-j] [-t] file

Positional arguments:
  file           File with YAML document(s)

Optional arguments:
  -h, --help     Show this help message and exit.
  -v, --version  Show program's version number and exit.
  -c, --compact  Display errors in compact mode
  -j, --to-json  Output a non-funky boring JSON
  -t, --trace    Show stack trace on error
```


### Bundled YAML library for browsers

``` html
<script src="js-yaml.min.js"></script>
<script type="text/javascript">
var doc = jsyaml.load('greeting: hello\nname: world');
</script>
```

Browser support was done mostly for online demo. If you find eny errors - feel
free to send pull requests with fixes. Also note, that IE and other old browsers
needs [es5-shims](https://github.com/kriskowal/es5-shim) to operate.


API
---

JS-YAML automatically registers handlers for `.yml` and `.yaml` files. You can
load them just with `require`. That's mostly equivalent to calling `load()` on
fetched content of a file. Just with one string!

``` javascript
require('js-yaml');

// Get document, or throw exception on error
var doc = require('/home/ixti/example.yml');

console.log(doc);
```

See [examples](https://github.com/nodeca/js-yaml/tree/master/examples) for
more details.


### load (string [ , options ])

Parses `string` as single YAML document. Returns a JavaScript object or throws
`YAMLException` on error.

NOTE: This function **does not** understands multi-document sources, it throws
exception on those.

options:

- `schema` _(default: `DEFAULT_SCHEMA`)_ - specifies a schema to use.
- `validate` _(default: true)_ - enables/disables input stream validation
  (unprintable symbols and other YAML requirements). If you are sure about
  input - set false to get some speed boost
- `strict` _(default - false)_ makes the loader to throw errors instead of
  warnings.
- `legacy` _(default: false)_ - makes the loader to expect YAML 1.1 documents if
  such documents have no explicit %YAML directive.
- `name` _(default: null)_ - string to be used as a file path in error/warning
  messages.
- `resolve` _(default - true)_ enables/disables resolving of nodes with
  non-specific `?` tag. That are plain scalars without any explicit tag.
  When false, all plain scalars will loaded as strings (that will inprove
  loading speed).


### loadAll (string, iterator [ , options ])

Same as `load()`, but understands multi-document sources and apply `iterator` to
each document.

``` javascript
var yaml = require('js-yaml');

yaml.loadAll(data, function (doc) {
  console.log(doc);
});
```


### safeLoad (string [ , options ])

Same as `load()` but uses `SAFE_SCHEMA` by default - only recommended tags of
YAML specification (no JavaScript-specific tags, e.g. `!!js/regexp`).


### safeLoadAll (string, iterator [ , options ])

Same as `loadAll()` but uses `SAFE_SCHEMA` by default - only recommended tags of
YAML specification (no JavaScript-specific tags, e.g. `!!js/regexp`).


### dump (object [ , options ])

Serializes `object` as YAML document.

options:

- `indent` _(default: 2)_ - indentation width to use (in spaces).
- `flowLevel` (default: -1) - specifies level of nesting, when to switch from
  block to flow style for collections. -1 means block style everwhere
- `schema` _(default: `DEFAULT_SCHEMA`)_ specifies a schema to use.
- `styles` - "tag" => "style" map. Each tag may have own set of styles.

styles:

``` none
!!null
  "canonical"   => "~"

!!int
  "binary"      => "0b1", "0b101010", "0b1110001111010"
  "octal"       => "01", "052", "016172"
  "decimal"     => "1", "42", "7290"
  "hexadecimal" => "0x1", "0x2A", "0x1C7A"

!!null, !!bool, !!float
  "lowercase"   => "null", "true", "false", ".nan", '.inf'
  "uppercase"   => "NULL", "TRUE", "FALSE", ".NAN", '.INF'
  "camelcase"   => "Null", "True", "False", ".NaN", '.Inf'
```

By default, !!int uses `decimal`, and !!null, !!bool, !!float use `lowercase`.


### safeDump (object [ , options ])

Same as `dump()` but uses SAFE_SCHEMA by default - only recommended tags of YAML
specification (no JavaScript-specific tags, e.g. `!!js/regexp`).


Supported YAML types
--------------------

The list of standard YAML tags and corresponding JavaScipt types. See also
[YAML tag discussion](http://pyyaml.org/wiki/YAMLTagDiscussion) and
[YAML types repository](http://yaml.org/type/).

```
!!null ''                   # null
!!bool 'yes'                # bool
!!int '3...'                # number
!!float '3.14...'           # number
!!binary '...base64...'     # buffer
!!timestamp 'YYYY-...'      # date
!!omap [ ... ]              # array of key-value pairs
!!pairs [ ... ]             # array or array pairs
!!set { ... }               # array of objects with given keys and null values
!!str '...'                 # string
!!seq [ ... ]               # array
!!map { ... }               # object
```

**JavaScript-specific tags**

```
!!js/regexp /pattern/gim            # RegExp
!!js/undefined ''                   # Undefined
!!js/function 'function () {...}'   # Function
```




## Caveats

Note, that you use arrays or objects as key in JS-YAML. JS do not allows objects
or array as keys, and stringifies (by calling .toString method) them at the
moment of adding them.

``` yaml
---
? [ foo, bar ]
: - baz
? { foo: bar }
: - baz
  - baz
```
``` javascript
{ "foo,bar": ["baz"], "[object Object]": ["baz", "baz"] }
```

Also, reading of properties on implicit block mapping keys is not supported yet.
So, the following YAML document cannot be loaded.

``` yaml
&anchor foo:
  foo: bar
  *anchor: duplicate key
  baz: bat
  *anchor: duplicate key
```

## License

View the [LICENSE](https://github.com/nodeca/js-yaml/blob/master/LICENSE) file
(MIT).
