# proxyquireify-es3

This is a fork of [proxyquireify](https://github.com/thlorenz/proxyquireify) that supports ECMAScript 3 so that it works in older browsers such as IE8.

## Installation

    npm install proxyquireify-es3

## Example 

**foo.js**:

```js
var bar = require('./bar');

module.exports = function () {
  return bar.kinder() + ' ist ' + bar.wunder();
};
```

**foo.test.js**:

```js
var proxyquire = require('proxyquireify-es3')(require);

var stubs = { 
  './bar': { 
      wunder: function () { return 'wirklich wunderbar'; }
    , kinder: function () { return 'schokolade'; }
  }
};

var foo = proxyquire('./src/foo', stubs);

console.log(foo()); 
```

**browserify.build.js**:

```js
var browserify = require('browserify');
var proxyquire = require('proxyquireify-es3');

browserify()
  .plugin(proxyquire.plugin)
  .require(require.resolve('./foo.test'), { entry: true })
  .bundle()
  .pipe(fs.createWriteStream(__dirname + '/bundle.js'));
```

load it in the browser and see:

    schokolade ist wirklich wunderbar

## API

### proxyquire.plugin()

**proxyquireify-es3** functions as a browserify plugin and needs to be registered with browserify like so:

```js
var browserify = require('browserify');
var proxyquire = require('proxyquireify-es3');

browserify()
  .plugin(proxyquire.plugin)
  .require(require.resolve('./test'), { entry: true })
  .bundle()
  .pipe(fs.createWriteStream(__dirname + '/bundle.js'));
```

Alternatively you can register **proxyquireify-es3** as a plugin from the command line like so:

```sh
browserify -p proxyquireify-es3/plugin test.js > bundle.js
```

### proxyquire.browserify()

#### Deprecation Warning

This API to setup **proxyquireify-es3** was used prior to [browserify plugin](https://github.com/substack/node-browserify#bpluginplugin-opts) support.

It has not been removed yet to make upgrading **proxyquireify-es3** easier for now, but it **will be deprecated in future
versions**. Please consider using the plugin API (above) instead.

****

To be used in build script instead of `browserify()`, autmatically adapts browserify to work for tests and injects
require overrides into all modules via a browserify transform.

```js
proxyquire.browserify()
  .require(require.resolve('./test'), { entry: true })
  .bundle()
  .pipe(fs.createWriteStream(__dirname + '/bundle.js'));
```

****

### proxyquire(request: String, stubs: Object)

- **request**: path to the module to be tested e.g., `../lib/foo`
- **stubs**: key/value pairs of the form `{ modulePath: stub, ... }`
  - module paths are relative to the tested module **not** the test file 
  - therefore specify it exactly as in the require statement inside the tested file
  - values themselves are key/value pairs of functions/properties and the appropriate override

```js
var proxyquire =  require('proxyquireify-es3')(require);
var barStub    =  { wunder: function () { 'really wonderful'; } };

var foo = proxyquire('./foo', { './bar': barStub })
```

#### Important Magic 

In order for browserify to include the module you are testing in the bundle, **proxyquireify-es3** will inject a
`require()` call for every module you are proxyquireing. So in the above example `require('./foo')` will be injected at
the top of your test file.

### noCallThru

By default **proxyquireify-es3** calls the function defined on the *original* dependency whenever it is not found on the stub.

If you prefer a more strict behavior you can prevent *callThru* on a per module or per stub basis.

If *callThru* is disabled, you can stub out modules that weren't even included in the bundle. **Note**, that unlike in
proxquire, there is no option to prevent call thru globally.

```js
// Prevent callThru for path module only
var foo = proxyquire('./foo', {
    path: {
      extname: function (file) { ... }
    , '@noCallThru': true
    }
  , fs: { readdir: function (..) { .. } }
});

// Prevent call thru for all contained stubs (path and fs)
var foo = proxyquire('./foo', {
    path: {
      extname: function (file) { ... }
    }
  , fs: { readdir: function (..) { .. } }
  , '@noCallThru': true
});

// Prevent call thru for all stubs except path
var foo = proxyquire('./foo', {
    path: {
      extname: function (file) { ... }
    , '@noCallThru': false
    }
  , fs: { readdir: function (..) { .. } }
  , '@noCallThru': true
});
```

## More Examples

- [foobar](https://github.com/roberttod/proxyquireify-es3/tree/master/examples/foobar)
