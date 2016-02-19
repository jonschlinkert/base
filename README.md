# base [![NPM version](https://img.shields.io/npm/v/base.svg)](https://www.npmjs.com/package/base) [![Build Status](https://img.shields.io/travis/node-base/base.svg)](https://travis-ci.org/node-base/base)

> base is the foundation for creating modular, unit testable and highly pluggable node.js applications, starting with a handful of common methods, like `set`, `get`, `del` and `use`.

## TOC

- [Install](#install)
- [Usage](#usage)
- [API](#api)
- [Test coverage](#test-coverage)
- [Related projects](#related-projects)
- [Generate docs](#generate-docs)
- [Running tests](#running-tests)
- [Contributing](#contributing)
- [Author](#author)
- [License](#license)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install

Install with [npm](https://www.npmjs.com/):

```sh
$ npm i base --save
```

## Usage

```js
var base = require('base');
```

**inherit**

```js
function App() {
  base.call(this);
}
base.extend(App);

var app = new App();
app.set('a', 'b');
app.get('a');
//=> 'b';
```

**instantiate**

```js
var app = base();
app.set('foo', 'bar');
console.log(app.foo);
//=> 'bar'
```

**Inherit or instantiate with a namespace**

A `.namespace()` method is exposed on the exported function to allow you to create a custom namespace for setting/getting on the instance.

```js
var Base = require('base')
var base = Base.namespace('cache');

var app = base();
app.set('foo', 'bar');
console.log(app.cache.foo);
//=> 'bar'
```

## API

All of the methods from [cache-base](https://github.com/jonschlinkert/cache-base)are exposed on the `base` API, as well as the following methods.

### [Base](index.js#L28)

Create an instance of `Base` with `options`.

**Params**

* `options` **{Object}**

**Example**

```js
var app = new Base();
app.set('foo', 'bar');
console.log(app.get('foo'));
//=> 'bar'
```

### [.is](index.js#L89)

Set the given `name` on `app._name` and `app.is*` properties. Used for doing lookups in plugins.

**Params**

* `name` **{String}**
* `returns` **{Boolean}**

**Example**

```js
app.is('foo');
console.log(app._name);
//=> 'foo'
console.log(app.isFoo);
//=> true
app.is('bar');
console.log(app.isFoo);
//=> true
console.log(app.isBar);
//=> true
console.log(app._name);
//=> 'bar'
```

### [.isRegistered](index.js#L126)

Returns true if a plugin has already been registered on an instance.

Plugin implementors are encouraged to use this first thing in a plugin
to prevent the plugin from being called more than once on the same
instance.

**Params**

* `name` **{String}**: The plugin name.
* `register` **{Boolean}**: If the plugin if not already registered, to record it as being registered pass `true` as the second argument.
* `returns` **{Boolean}**: Returns true if a plugin is already registered.

**Events**

* `emits`: `plugin` Emits the name of the plugin.

**Example**

```js
var base = new Base();
base.use(function(app) {
  if (app.isRegistered('myPlugin')) return;
  // do stuff to `app`
});

// to also record the plugin as being registered
base.use(function(app) {
  if (app.isRegistered('myPlugin', true)) return;
  // do stuff to `app`
});
```

### [.assertPlugin](index.js#L153)

Throws an error when plugin `name` is not registered.

**Params**

* `name` **{String}**: The plugin name.

**Example**

```js
var base = new Base();
base.use(function(app) {
  app.assertPlugin('base-foo');
  app.assertPlugin('base-bar');
  app.assertPlugin('base-baz');
});
```

### [.use](index.js#L177)

Define a plugin function to be called immediately upon init. Plugins are chainable and the only parameter exposed to the plugin is the application instance.

**Params**

* `fn` **{Function}**: plugin function to call
* `returns` **{Object}**: Returns the item instance for chaining.

**Events**

* `emits`: `use` with no arguments.

**Example**

```js
var app = new Base()
  .use(foo)
  .use(bar)
  .use(baz)
```

### [.lazy](index.js#L202)

Lazily invoke a registered plugin. **Note** that this method can only be used with:

1. plugins that _add a single method or property_ to `app`
2. plugins that do not (themselves) add a getter/setter property (they're already lazy)
3. plugins that do not return a function

**Params**

* `prop` **{String}**: The name of the property or method added by the plugin.
* `fn` **{Function}**: The plugin function
* `options` **{Object}**: Options to use when the plugin is invoked.
* `returns` **{Object}**: Returns the instance for chaining

**Example**

```js
app.lazy('store', require('base-store'));
```

### [.define](index.js#L234)

Define a non-enumerable property on the instance. Dot-notation is **not supported** with `define`.

**Params**

* `key` **{String}**: The name of the property to define.
* `value` **{any}**
* `returns` **{Object}**: Returns the instance for chaining.

**Events**

* `emits`: `define` with `key` and `value` as arguments.

**Example**

```js
// arbitrary `render` function using lodash `template`
define('render', function(str, locals) {
  return _.template(str)(locals);
});
```

### [.mixin](index.js#L253)

Mix property `key` onto the Base prototype. If base-methods
is inherited using `Base.extend` this method will be overridden
by a new `mixin` method that will only add properties to the
prototype of the inheriting application.

**Params**

* `key` **{String}**
* `val` **{Object|Array}**
* `returns` **{Object}**: Returns the instance for chaining.

### [.use](index.js#L275)

Static method for adding global plugin functions that will be added to an instance when created.

**Params**

* `fn` **{Function}**: Plugin function to use on each instance.

**Example**

```js
Base.use(function(app) {
  app.foo = 'bar';
});
var app = new Base();
console.log(app.foo);
//=> 'bar'
```

### [.extend](index.js#L287)

Static method for inheriting both the prototype and
static methods of the `Base` class. See [class-utils](https://github.com/jonschlinkert/class-utils)
for more details.

### [.Base.mixin](index.js#L325)

Static method for adding mixins to the prototype. When a function is returned from the mixin plugin, it will be added to an array so it can be used on inheriting classes via `Base.mixins(Child)`.

**Params**

* `fn` **{Function}**: Function to call

**Example**

```js
Base.mixin(function fn(proto) {
  proto.foo = function(msg) {
    return 'foo ' + msg;
  };
  return fn;
});
```

### [.Base.mixins](index.js#L346)

Static method for running currently saved global mixin functions against a child constructor.

**Params**

* `Child` **{Function}**: Constructor function of a child class

**Example**

```js
Base.extend(Child);
Base.mixins(Child);
```

### [.inherit](index.js#L358)

Similar to `util.inherit`, but copies all static properties,
prototype properties, and descriptors from `Provider` to `Receiver`.
[class-utils](https://github.com/jonschlinkert/class-utils)for more details.

## Test coverage

```
Statements   : 100% ( 83/83 )
Branches     : 100% ( 22/22 )
Functions    : 100% ( 19/19 )
Lines        : 100% ( 82/82 )
```

## Related projects

There are a number of different plugins available for extending base. Let us know if you create your own!

* [base-argv](https://www.npmjs.com/package/base-argv): Plugin that post-processes the argv object from simplify how args are mapped to options, tasks… [more](https://www.npmjs.com/package/base-argv) | [homepage](https://github.com/jonschlinkert/base-argv)
* [base-cli](https://www.npmjs.com/package/base-cli): Plugin for base-methods that maps built-in methods to CLI args (also supports methods from a… [more](https://www.npmjs.com/package/base-cli) | [homepage](https://github.com/jonschlinkert/base-cli)
* [base-config](https://www.npmjs.com/package/base-config): base-methods plugin that adds a `config` method for mapping declarative configuration values to other 'base'… [more](https://www.npmjs.com/package/base-config) | [homepage](https://github.com/jonschlinkert/base-config)
* [base-cwd](https://www.npmjs.com/package/base-cwd): Base plugin that adds a getter/setter for the current working directory. | [homepage](https://github.com/jonschlinkert/base-cwd)
* [base-data](https://www.npmjs.com/package/base-data): adds a `data` method to base-methods. | [homepage](https://github.com/jonschlinkert/base-data)
* [base-fs](https://www.npmjs.com/package/base-fs): base-methods plugin that adds vinyl-fs methods to your 'base' application for working with the file… [more](https://www.npmjs.com/package/base-fs) | [homepage](https://github.com/jonschlinkert/base-fs)
* [base-fs-rename](https://www.npmjs.com/package/base-fs-rename): Plugin for 'base' applications that adds a `rename` method that can be passed to `app.dest()`… [more](https://www.npmjs.com/package/base-fs-rename) | [homepage](https://github.com/jonschlinkert/base-fs-rename)
* [base-generators](https://www.npmjs.com/package/base-generators): Adds project-generator support to your `base` application. | [homepage](https://github.com/jonschlinkert/base-generators)
* [base-list](https://www.npmjs.com/package/base-list): base-runner plugin that prompts the user to choose from a list of registered applications and… [more](https://www.npmjs.com/package/base-list) | [homepage](https://github.com/doowb/base-list)
* [base-option](https://www.npmjs.com/package/base-option): Adds a few options methods to base, like `option`, `enable` and `disable`. See the readme… [more](https://www.npmjs.com/package/base-option) | [homepage](https://github.com/node-base/base-option)
* [base-pipeline](https://www.npmjs.com/package/base-pipeline): base-methods plugin that adds pipeline and plugin methods for dynamically composing streaming plugin pipelines. | [homepage](https://github.com/jonschlinkert/base-pipeline)
* [base-pkg](https://www.npmjs.com/package/base-pkg): Base plugin for adding a `pkg` object with get/set methods for getting data from package.json… [more](https://www.npmjs.com/package/base-pkg) | [homepage](https://github.com/jonschlinkert/base-pkg)
* [base-plugins](https://www.npmjs.com/package/base-plugins): Upgrade's plugin support in base-methods to allow plugins to be called any time after init. | [homepage](https://github.com/jonschlinkert/base-plugins)
* [base-project](https://www.npmjs.com/package/base-project): Base plugin that adds a `project` getter to the instance, for getting the name of… [more](https://www.npmjs.com/package/base-project) | [homepage](https://github.com/jonschlinkert/base-project)
* [base-questions](https://www.npmjs.com/package/base-questions): Plugin for base-methods that adds methods for prompting the user and storing the answers on… [more](https://www.npmjs.com/package/base-questions) | [homepage](https://github.com/jonschlinkert/base-questions)
* [base-runner](https://www.npmjs.com/package/base-runner): Orchestrate multiple instances of base-methods at once. | [homepage](https://github.com/jonschlinkert/base-runner)
* [base-store](https://www.npmjs.com/package/base-store): Plugin for getting and persisting config values with your base-methods application. Adds a 'store' object… [more](https://www.npmjs.com/package/base-store) | [homepage](https://github.com/jonschlinkert/base-store)
* [base-task](https://www.npmjs.com/package/base-task): base plugin that provides a very thin wrapper around [https://github.com/doowb/composer](https://github.com/doowb/composer) for adding task methods to… [more](https://www.npmjs.com/package/base-task) | [homepage](https://github.com/node-base/base-task)
* [base-tree](https://www.npmjs.com/package/base-tree): Add a tree method to generate a hierarchical tree structure representing nested applications and child… [more](https://www.npmjs.com/package/base-tree) | [homepage](https://github.com/doowb/base-tree)

## Generate docs

Generate readme and API documentation with [verb](https://github.com/verbose/verb):

```sh
$ npm i -d && npm run docs
```

Or, if [verb](https://github.com/verbose/verb) is installed globally:

```sh
$ verb
```

## Running tests

Install dev dependencies:

```sh
$ npm i -d && npm test
```

## Contributing

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](https://github.com/jonschlinkert/base/issues/new).

## Author

**Jon Schlinkert**

* [github/jonschlinkert](https://github.com/jonschlinkert)
* [twitter/jonschlinkert](http://twitter.com/jonschlinkert)

## License

Copyright © 2016 [Jon Schlinkert](https://github.com/jonschlinkert)
Released under the [MIT license](https://github.com/node-base/base/blob/master/LICENSE).

***

_This file was generated by [verb](https://github.com/verbose/verb), v0.9.0, on February 19, 2016._