---
id: version-7.2.0-babel-plugin-proposal-private-methods
title: @babel/plugin-proposal-private-methods
sidebar_label: proposal-private-methods
original_id: babel-plugin-proposal-private-methods
---

> Note: Support for private accessors (getters and setters) was added in [7.3.0](https://babeljs.io/blog/2019/01/21/7.3.0)!

## Example

```js
class Counter extends HTMLElement {
  #xValue = 0;

  #clicked() {
    this.#x++;
  }
}
```

## Installation

```sh
$ npm install @babel/plugin-proposal-private-methods
```

## Usage

### Via `.babelrc` (Recommended)

**.babelrc**

Without options:

```json
{
  "plugins": ["@babel/plugin-proposal-private-methods"]
}
```

With options:

```json
{
  "plugins": [["@babel/plugin-proposal-private-methods", { "loose": true }]]
}
```

### Via CLI

```sh
$ babel --plugins @babel/plugin-proposal-private-methods script.js
```

### Via Node API

```javascript
require("@babel/core").transform("code", {
  plugins: ["@babel/plugin-proposal-private-methods"],
});
```

## Options

### `loose`

`boolean`, defaults to `false`.

> Note: The `loose` mode configuration setting _must_ be the same as [`@babel/proposal-class-properties`](plugin-proposal-class-properties.md).

When true, private methods will be assigned directly on its parent
via `Object.defineProperty` rather than a `WeakSet`. This results in improved
performance and debugging (normal property access vs `.get()`) at the expense
of potentially leaking "privates" via things like `Object.getOwnPropertyNames`.

Let's use the following as an example:

```javascript
class Foo {
  constructor() {
    this.publicField = this.#privateMethod();
  }

  #privateMethod() {
    return 42;
  }
}
```

By default, this becomes:

```javascript
var Foo = function Foo() {
  "use strict";

  babelHelpers.classCallCheck(this, Foo);

  _privateMethod.add(this);

  this.publicField = babelHelpers
    .classPrivateMethodGet(this, _privateMethod, _privateMethod2)
    .call(this);
};

var _privateMethod = new WeakSet();

var _privateMethod2 = function _privateMethod2() {
  return 42;
};
```

With `{ loose: true }`, it becomes:

```javascript
var Foo = function Foo() {
  "use strict";

  babelHelpers.classCallCheck(this, Foo);
  Object.defineProperty(this, _privateMethod, {
    value: _privateMethod2,
  });
  this.publicField = babelHelpers
    .classPrivateFieldLooseBase(this, _privateMethod)
    [_privateMethod]();
};

var _privateMethod = babelHelpers.classPrivateFieldLooseKey("privateMethod");

var _privateMethod2 = function _privateMethod2() {
  return 42;
};
```

## References

- [Proposal: Private methods and getter/setters for JavaScript classes](https://github.com/tc39/proposal-private-methods)
