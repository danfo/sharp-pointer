# sharp-pointer

A complete implementation of JSON Pointer ([RFC 6901](https://tools.ietf.org/html/rfc6901)) for nodejs and modern browsers.

## Install

```bash
npm install sharp-pointer
```

## Use/Import

[nodejs](https://nodejs.org/en/)
```javascript
var ptr = require('sharp-pointer')
```

### Module API

**Classes:**

* [`JsonPointer`](#user-content-jsonpointer-class) : _class_ &ndash; a convenience class for working with JSON pointers.
* [`JsonReference`](#user-content-jsonreference-class) : _class_ &ndash; a convenience class for working with JSON references.

**Functions:**

* [`.create(pointer)`](#user-content-createpointer)
* [`.has(target,pointer)`](#user-content-hastargetpointer)
* [`.get(target,pointer)`](#user-content-gettargetpointer)
* [`.set(target,pointer,value,force)`](#user-content-settarget-pointer-value-force)
* [`.flatten(target,fragmentId)`](#user-content-flattentarget-fragmentid)
* [`.list(target,fragmentId)`](#user-content-listtarget-fragmentid)
* [`.map(target,fragmentId)`](#user-content-maptarget-fragmentid)

All example code assumes data has this structure:
```javascript
var data = {
  legumes: [{
    name: 'pinto beans',
    unit: 'lbs',
    instock: 4
  }, {
    name: 'lima beans',
    unit: 'lbs',
    instock: 21
  }, {
    name: 'black eyed peas',
    unit: 'lbs',
    instock: 13
  }, {
    name: 'plit peas',
    unit: 'lbs',
    instock: 8
  }]
}
```

#### .create(pointer)

Creates an instance of the `JsonPointer` class.

_arguments:_
* `pointer` : _string, required_ &ndash; a JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5) or [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6)

_returns:_
* a new [`JsonPointer` instance](#user-content-jsonpointer-class)

_example:_
```javascript
var pointer = ptr.create('/legumes/0');
// fragmentId: #/legumes/0
```

#### .has(target,pointer)

Determins whether the specified `target` has a value at the `pointer`'s path.

_arguments:_
* `target` : _object, required_ &ndash; the target object
* `pointer` : _string, required_ &ndash; a JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5) or [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6)

_returns:_
* the dereferenced value or _undefined_ if nonexistent

#### .get(target,pointer)

Gets a value from the specified `target` object at the `pointer`'s path

_arguments:_
* `target` : _object, required_ &ndash; the target object
* `pointer` : _string, required_ &ndash; a JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5) or [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6)

_returns:_
* the dereferenced value or _undefined_ if nonexistent

_example:_
```javascript
var value = ptr.get(data, '/legumes/1');
// fragmentId: #/legumes/1
```

#### .set(target, pointer, value, force)

Sets the `value` at the specified `pointer` on the `target`. The default behavior is to do nothing if `pointer` is nonexistent.

_arguments:_
* `target` : _object, required_ &ndash; the target object
* `pointer` : _string, required_ &ndash; a JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5) or [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6)
* `value` : _any_ &ndash; the value to be set at the specified `pointer`'s path
* `force` : _boolean, optional_ &ndash; indicates [whether nonexistent paths are created during the call](https://tools.ietf.org/html/rfc6901#section-7)

_returns:_
* The prior value at the pointer's path &mdash; therefore, _undefined_ means the pointer's path was nonexistent.


_example:_
```javascript
var prior = ptr.set(data, '#/legumes/1/instock', 50);
```

example force:
```javascript
var data = {};

ptr.set(data, '#/peter/piper', 'man', true);
ptr.set(data, '#/peter/pan', 'boy', true);
ptr.set(data, '#/peter/pickle', 'dunno', true);

console.log(JSON.stringify(data, null, '  '));
```
```json
{
  "peter": {
    "piper": "man",
    "pan": "boy",
    "pickle": "dunno"
  }
}
```

#### .list(target, fragmentId)

Lists all of the pointers available on the specified `target`.

> See [a discussion about cycles in the object graph later in this document](#user-content-object-graph-cycles-and-recursion) if you have interest in how such is dealt with.

_arguments:_
* `target` : _object, required_ &ndash; the target object
* `fragmentId` : _boolean, optional_ &ndash; indicates whether fragment identifiers should be listed instead of pointers

_returns:_
* an array of `pointer-value` pairs

_example:_
```javascript
var list = ptr.list(data);
```
```json
[ ...
  {
    "pointer": "/legumes/2/unit",
    "value": "ea"
  },
  {
    "pointer": "/legumes/2/instock",
    "value": 9340
  },
  {
    "pointer": "/legumes/3/name",
    "value": "plit peas"
  },
  {
    "pointer": "/legumes/3/unit",
    "value": "lbs"
  },
  {
    "pointer": "/legumes/3/instock",
    "value": 8
  }
]
```

_`fragmentId` example:_
```javascript
var list = ptr.list(data, true);
```
```json
[ ...
  {
    "fragmentId": "#/legumes/2/unit",
    "value": "ea"
  },
  {
    "fragmentId": "#/legumes/2/instock",
    "value": 9340
  },
  {
    "fragmentId": "#/legumes/3/name",
    "value": "plit peas"
  },
  {
    "fragmentId": "#/legumes/3/unit",
    "value": "lbs"
  },
  {
    "fragmentId": "#/legumes/3/instock",
    "value": 8
  }
]
```

#### .flatten(target, fragmentId)

Flattens an object graph (the `target`) into a single-level object of `pointer-value` pairs.

_arguments:_
* `target` : _object, required_ &ndash; the target object
* `fragmentId` : _boolean, optional_ &ndash; indicates whether fragment identifiers should be listed instead of pointers

_returns:_
* a flattened object of `property-value` pairs as properties.

_example:_
```javascript
var obj = ptr.flatten(data, true);
```
```json
{ ...
  "#/legumes/1/name": "lima beans",
  "#/legumes/1/unit": "lbs",
  "#/legumes/1/instock": 21,
  "#/legumes/2/name": "black eyed peas",
  "#/legumes/2/unit": "ea",
  "#/legumes/2/instock": 9340,
  "#/legumes/3/name": "plit peas",
  "#/legumes/3/unit": "lbs",
  "#/legumes/3/instock": 8
}
```

#### .map(target, fragmentId)

Flattens an object graph (the `target`) into a [Map object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map).

_arguments:_
* `target` : _object, required_ &ndash; the target object
* `fragmentId` : _boolean, optional_ &ndash; indicates whether fragment identifiers should be listed instead of pointers

_returns:_
* a Map object containing key-value pairs where keys are pointers.

_example:_
```javascript
var map = ptr.map(data, true);

for (let it of map) {
  console.log(JSON.stringify(it, null, '  '));
}
```
```json
...
["#/legumes/0/name", "pinto beans"]
["#/legumes/0/unit", "lbs"]
["#/legumes/0/instock", 4 ]
["#/legumes/1/name", "lima beans"]
["#/legumes/1/unit", "lbs"]
["#/legumes/1/instock", 21 ]
["#/legumes/2/name", "black eyed peas"]
["#/legumes/2/unit", "ea"]
["#/legumes/2/instock", 9340 ]
["#/legumes/3/name", "plit peas"]
["#/legumes/3/unit", "lbs"]
["#/legumes/3/instock", 8 ]
```

#### .decode(pointer)

Decodes the specified `pointer`.

_arguments:_
* `pointer` : _string, required_ &ndash; a JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5) or [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6).

_returns:_
* An array of path segments used as indexers to descend from a root/`target` object to a referenced value.

_example:_
```javascript
var path = ptr.decode('#/legumes/1/instock');
```
```json
[ "legumes", "1", "instock" ]
```

#### .decodePointer(pointer)

Decodes the specified `pointer`.

_arguments:_
* `pointer` : _string, required_ &ndash; a JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5).

_returns:_
* An array of path segments used as indexers to descend from a root/`target` object to a referenced value.

_example:_
```javascript
var path = ptr.decodePointer('/people/wilbur dongleworth/age');
```
```json
[ "people", "wilbur dongleworth", "age" ]
```

#### .encodePointer(path)

Encodes the specified `path` as a JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5).

_arguments:_
* `path` : _Array, required_ &ndash; an array of path segments

_returns:_
* A JSON pointer in [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5).

_example:_
```javascript
var path = ptr.encodePointer(['people', 'wilbur dongleworth', 'age']);
```
```json
"/people/wilbur dongleworth/age"
```

#### .decodeUriFragmentIdentifier(pointer)

Decodes the specified `pointer`.

_arguments:_
* `pointer` : _string, required_ &ndash; a JSON pointer in [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6).

_returns:_
* An array of path segments used as indexers to descend from a root/`target` object to a referenced value.

_example:_
```javascript
var path = ptr.decodePointer('#/people/wilbur%20dongleworth/age');
```
```json
[ "people", "wilbur dongleworth", "age" ]
```

#### .encodeUriFragmentIdentifier(path)

Encodes the specified `path` as a JSON pointer in [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6).

_arguments:_
* `path` : _Array, required_ - an array of path segments

_returns:_
* A JSON pointer in [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6).

_example:_
```javascript
var path = ptr.encodePointer(['people', 'wilbur dongleworth', 'age']);
```
```json
"#/people/wilbur%20dongleworth/age"
```

#### .noConflict()

Restores a conflicting `JsonPointer` variable in the global/root namespace (not necessary in node, but useful in browsers).

_example:_
```html
	<!-- ur codez -->
	<script src="/json-ptr-0.3.0.min.js"></script>
	<script>
		// At this point, JsonPointer is the json-ptr module
		var ptr = JsonPointer.noConflict();
		// and now it is restored to whatever it was before the json-ptr import.
	</script>
```

### `JsonPointer` Class

Encapsulates pointer related operations for a specified `pointer`.

_properties:_
* `.path` : _array_ &ndash; contains the pointer's path segements.
* `.pointer` : _string_ &ndash; the pointer's [JSON string representation](https://tools.ietf.org/html/rfc6901#section-5)
* `.uriFragmentIdentifier` : _string_ &ndash; the pointer's [URI fragment identifier representation](https://tools.ietf.org/html/rfc6901#section-6)

_methods:_
#### .has(target)
Determins whether the specified `target` has a value at the pointer's path.

#### .get(target)
Looks up the specified `target`'s value at the pointer's path if such exists; otherwise _undefined_.

#### .set(target, value, force)
Sets the specified `target`'s value at the pointer's path, if such exists.If `force` is specified (_truthy_), missing path segments are created and the value is always set at the pointer's path.

_arguments:_
* `target` : _object, required_ &ndash; the target object
* `value` : _any_ &ndash; the value to be set at the specified `pointer`'s path
* `force` : _boolean, optional_ &ndash; indicates [whether nonexistent paths are created during the call](https://tools.ietf.org/html/rfc6901#section-7)

_result:_
* The prior value at the pointer's path &mdash; therefore, _undefined_ means the pointer's path was nonexistent.

## Tests

Tests are written using [mocha](http://mochajs.org/) and [expect.js](https://github.com/Automattic/expect.js).

```bash
npm test
```

... or ...

```bash
mocha
```

## License

[MIT](https://github.com/flitbit/json-ptr/blob/master/LICENSE)
