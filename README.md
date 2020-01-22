# ui5-simple-require

## Description

Import UI5 modules into NodeJS applications, allowing isolation of UI5 components and injection of dependencies to create an isolated test environment.

## Requirements
* [NodeJS](https://nodejs.org/en/download/), version 8.0 or higher

## Installation
Install at your Node.js project:
```
$ npm install @ui5/simple-require --save-dev
```

## Usage

When importing UI5 modules, there are few things to consider, the two major things that need to be resolved are *SAP's Global Context* and *Dependency Lookup*. This tool provides ways to inject both dependencies based on their path as well as properties that will be injected in global context when the module is loaded. 

### Require UI5 Modules 

`ui5require` will load modules that are in `sap.ui.define` style. For example, suppose you have the following UI5 module:

```js
sap.ui.define([], function() {
  const myClass = function() { /* CODE */ };
    /* CODE */
  return myClass;
});
```

When running the import code: 

```javascript
const ui5require = require('ui5-module-loader').ui5require;
const MyClass = ui5require('./myUI5Class.js');

var myObject = new MyClass();
```

The constant variable `ui5class` will contain the constructor function. 

### Resolving Dependencies

Suppose a similar module now with defined dependencies.

```js
sap.ui.define(['/path/to/Dependency'], function() {
  const myClass = function(Dependency) { /* CODE */ };
    /* CODE */
  return myClass;
});
```

Dependencies will be resolved in the following order:

- Position injected
- Path injected
- Loaded

#### Position Injected

```js
const myClass = ui5require('./myUI5Class.js', [ new FakeDependency() ]);
```

#### Path Injected

Injecting by path will create a virtual path lookup. Where, whenever a dependency is required, it will first look at this injected path. 

```js
const moduleLoader = require('ui5-module-loader');
const ui5require = moduleLoader.ui5require;

moduleLoader.inject('/path/to/dependency', new FakeDependency);
const myClass = ui5require('./myUI5Class.js');
```

#### Loaded 

When no injection methods are passed, module loader will try to load the module by its path. Meaning that if a file that matches `/path/to/Dependency` is found in the path, the original dependency is loaded. 

### Injecting Global Dependency

Injecting a global dependency works similar to injecting dependencies. A global lookup is created for whenever a new module is loaded a global context is injected. 

```js
const moduleLoader = require('ui5-module-loader');
const ui5require = moduleLoader.ui5require;

moduleLoader.globalContext({ someValue: "custom value" });
const myClass = ui5require('./myUI5Class.js');
```

The injected object will be under `global.sap` object. Such as:

```js
// ... ui5 module
sap.someValue; // evaluates to "custom value"
```

## Example

Using mocha and chai for writting unit tests.

```js
// MoneyChanger.js
sap.ui.define(["./coin", "./note", "./CurrencyServer"], function(Coin, Note, CurrencyServer) {
	var MoneyChanger = function() {};

	MoneyChanger.prototype.getChange = function(i) {
             ... 
	}

	return MoneyChanger;
});

************************************************************
// MoneyChangerTest.js
const expect = require('chai').expect;

const API = require('ui5-module-loader');
const ui5require = require('ui5-module-loader').ui5require;

API.inject('/src/main/webapp/money/CurrencyServer', FakeCurrencyServer);

const MoneyChanger = ui5require('/src/main/webapp/money/changer');
const Note = ui5require('/src/main/webapp/money/note');
const Coin = ui5require('/src/main/webapp/money/coin');

describe("Should test money changer", () => {

  let changer;

  beforeEach(() => {
    changer = new MoneyChanger();
  })

  it("Should create money changer class", () => {
    expect(changer).to.be.an("object");
  });
	
  it("Should get one coin from value 1", () => {
    expect(changer.getChange(1)).to.be.deep.equal([ new Coin(1) ]);
  });

  it("Should get one note from value 2", () => {
    expect(changer.getChange(2)).to.be.deep.equal([ new Note(2) ]);  
  });

  it("Should get one note and one coin from value 3", () => {
    expect(changer.getChange(3)).to.be.deep.equal([ new Note(2), new Coin(1) ]);
  });

  // ...

});

```

## API

#### `ui5require(path [, position_dependencies] [, global_context])`

- `path` \<string\>
- `position_dependencies` \<Array\>
- `global_context` \<Object\>
- **Returns:** \<Object\> Loaded Module.


#### `inject(path, dependency)`

- `path` \<string\>
- `dependency` \<Object\> 


#### `clearInjection()`

Deletes any dependencies passed with `inject`


#### `globalContext(context)`

- `context` \<string\>


#### `clearGlobalContext()`

Deletes any global object passed with `globalContext(...)`



#### `createExtendableFromPrototype(prototype)`

- `prototype` \<Object\>
- **Returns:** \<Object\> prototype with UI5's fake `extend` method. 


#### `createExtendableFromObj(prototype)`

- `prototype` \<string\>
- **Returns:** \<Object\> class with UI5's fake `extend` method. 


#### `[DEPRECATED] import(path [, position_dependencies] [, global_context])`

- `path` \<string\>
- `position_dependencies` \<Array\>
- `global_context` \<Object\>
- **Returns: ** \<Object\> Loaded Module.

## Support

If you think you found a bug or need help using the module, please create a new [github issue](https://github.com/SAP/ui5-simple-require/issues/new). 


## Licence
Copyright (c) 2020 SAP SE or an SAP affiliate company. All rights reserved.
This file is licensed under the Apache Software License, v. 2 except as noted otherwise in the [LICENSE](LICENSE.txt) file.
