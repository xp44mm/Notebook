## 24 Modules

### 24.1 Overview: syntax of ECMAScript modules

#### 24.1.1 Exporting

```js
// Named exports
export function f() {}
export const one = 1;
export {foo, b as bar};

// Default exports
export default function f() {} // declaration with optional name
// Replacement for `const` (there must be exactly one value)
export default 123;

// Re-exporting from another module
export * from './some-module.mjs';
export {foo, b as bar} from './some-module.mjs';
```

#### 24.1.2 Importing

```js
// Named imports
import {foo, bar as b} from './some-module.mjs';
// Namespace import
import * as someModule from './some-module.mjs';
// Default import
import someModule from './some-module.mjs';

// Combinations:
import someModule, * as someModule from './some-module.mjs';
import someModule, {foo, bar as b} from './some-module.mjs';

// Empty import (for modules with side effects)
import './some-module.mjs';
```

### 24.2 JavaScript source code formats

The current landscape of JavaScript modules is quite diverse: ES6 brought built-in modules, but the source code formats that came before them, are still around, too. Understanding the latter helps understand the former, so let’s investigate. The next sections describe the following ways of delivering JavaScript source code:

- *Scripts* are code fragments that browsers run in global scope. They are precursors of modules.
- *CommonJS modules* are a module format that is mainly used on servers (e.g., via Node.js).
- *AMD modules* are a module format that is mainly used in browsers.
- *ECMAScript modules* are JavaScript’s built-in module format. It supersedes all previous formats.

Tbl. 17 gives an overview of these code formats. Note that for CommonJS modules and ECMAScript modules, two filename extensions are commonly used. Which one is appropriate depends on how you want to use a file. Details are given later in this chapter.

|                   | Runs on              | Loaded | Filename ext. |
| :---------------- | :------------------- | :----- | :------------ |
| Script            | browsers             | async  | `.js`         |
| CommonJS module   | servers              | sync   | `.js .cjs`    |
| AMD module        | browsers             | async  | `.js`         |
| ECMAScript module | browsers and servers | async  | `.js .mjs`    |

#### 24.2.1 Code before built-in modules was written in ECMAScript 5

Before we get to built-in modules (which were introduced with ES6), all code that you’ll see, will be written in ES5. Among other things:

- ES5 did not have `const` and `let`, only `var`.
- ES5 did not have arrow functions, only function expressions.

### 24.3 Before we had modules, we had scripts

Initially, browsers only had *scripts* – pieces of code that were executed in global scope. As an example, consider an HTML file that loads script files via the following HTML:

```html
<script src="other-module1.js"></script>
<script src="other-module2.js"></script>
<script src="my-module.js"></script>
```

The main file is `my-module.js`, where we simulate a module:

```js
var myModule = (function () { // Open IIFE
  // Imports (via global variables)
  var importedFunc1 = otherModule1.importedFunc1;
  var importedFunc2 = otherModule2.importedFunc2;

  // Body
  function internalFunc() {
    // ···
  }
  function exportedFunc() {
    importedFunc1();
    importedFunc2();
    internalFunc();
  }

  // Exports (assigned to global variable `myModule`)
  return {
    exportedFunc: exportedFunc,
  };
})(); // Close IIFE
```

`myModule` is a global variable that is assigned the result of immediately invoking a function expression. The function expression starts in the first line. It is invoked in the last line.

This way of wrapping a code fragment is called *immediately invoked function expression* (IIFE, coined by Ben Alman). What do we gain from an IIFE? `var` is not block-scoped (like `const` and `let`), it is function-scoped: the only way to create new scopes for `var`-declared variables is via functions or methods (with `const` and `let`, you can use either functions, methods, or blocks `{}`). Therefore, the IIFE in the example hides all of the following variables from global scope and minimizes name clashes: `importedFunc1`, `importedFunc2`, `internalFunc`, `exportedFunc`.

Note that we are using an IIFE in a particular manner: at the end, we pick what we want to export and return it via an object literal. That is called the *revealing module pattern* (coined by Christian Heilmann).

This way of simulating modules, has several issues:

- Libraries in script files export and import functionality via global variables, which risks name clashes.
- Dependencies are not stated explicitly, and there is no built-in way for a script to load the scripts it depends on. Therefore, the web page has to load not just the scripts that are needed by the page but also the dependencies of those scripts, the dependencies’ dependencies, etc. And it has to do so in the right order!

### 24.4 Module systems created prior to ES6

Prior to ECMAScript 6, JavaScript did not have built-in modules. Therefore, the flexible syntax of the language was used to implement custom module systems *within* the language. Two popular ones are:

- CommonJS (targeting the server side)
- AMD (Asynchronous Module Definition, targeting the client side)

#### 24.4.1 Server side: CommonJS modules

The original CommonJS standard for modules was created for server and desktop platforms. It was the foundation of the original Node.js module system, where it achieved enormous popularity. Contributing to that popularity were the npm package manager for Node and tools that enabled using Node modules on the client side (browserify, webpack, and others).

From now on, *CommonJS module* means the Node.js version of this standard (which has a few additional features). This is an example of a CommonJS module:

```js
// Imports
var importedFunc1 = require('./other-module1.js').importedFunc1;
var importedFunc2 = require('./other-module2.js').importedFunc2;

// Body
function internalFunc() {
  // ···
}
function exportedFunc() {
  importedFunc1();
  importedFunc2();
  internalFunc();
}

// Exports
module.exports = {
  exportedFunc: exportedFunc,
};
```

CommonJS can be characterized as follows:

- Designed for servers.
- Modules are meant to be loaded *synchronously* (the importer waits while the imported module is loaded and executed).
- Compact syntax.

#### 24.4.2 Client side: AMD (Asynchronous Module Definition) modules



The AMD module format was created to be easier to use in browsers than the CommonJS format. Its most popular implementation is [RequireJS](https://requirejs.org/). The following is an example of an AMD module.

```js
define(['./other-module1.js', './other-module2.js'],
  function (otherModule1, otherModule2) {
    var importedFunc1 = otherModule1.importedFunc1;
    var importedFunc2 = otherModule2.importedFunc2;

    function internalFunc() {
      // ···
    }
    function exportedFunc() {
      importedFunc1();
      importedFunc2();
      internalFunc();
    }
    
    return {
      exportedFunc: exportedFunc,
    };
  });
```

AMD can be characterized as follows:

- Designed for browsers.
- Modules are meant to be loaded *asynchronously*. That’s a crucial requirement for browsers, where code can’t wait until a module has finished downloading. It has to be notified once the module is available.
- The syntax is slightly more complicated.

On the plus side, AMD modules can be executed directly. In contrast, CommonJS modules must either be compiled before deployment or custom source code must be generated and evaluated dynamically [(think `eval()`)](https://exploringjs.com/impatient-js/ch_callables.html#eval). That isn’t always permitted on the web.

#### 24.4.3 Characteristics of JavaScript modules

Looking at CommonJS and AMD, similarities between JavaScript module systems emerge:

- There is one module per file.
- Such a file is basically a piece of code that is executed:
  - Local scope: The code is executed in a local “module scope”. Therefore, by default, all of the variables, functions, and classes declared in it are internal and not global.
  - Exports: If you want any declared entity to be exported, you must explicitly mark it as an export.
  - Imports: Each module can import exported entities from other modules. Those other modules are identified via *module specifiers* (usually paths, occasionally full URLs).
- Modules are *singletons*: Even if a module is imported multiple times, only a single “instance” of it exists.
- No global variables are used. Instead, module specifiers serve as global IDs.

通用格式 UMD(Universal Module Definition)，既可用於CommonJS，又可用於AMD的模塊打包模式。

```js
(function (root, factory) {
    if (typeof exports === 'object') {
        module.exports = factory();
    } else if (typeof define === 'function' && define.amd) {
        define(factory);
    } else {
        root.eventUtil = factory();
    }
})(this, function createModule() {
    return {
        addEvent: function(el, type, handle) {},
        removeEvent: function(el, type, handle) {},
    };
});
```



### 24.5 ECMAScript modules



*ECMAScript modules* (*ES modules* or *ESM*) were introduced with ES6. They continue the tradition of JavaScript modules and have all of their aforementioned characteristics. Additionally:

- With CommonJS, ES modules share the compact syntax and support for cyclic dependencies.
- With AMD, ES modules share being designed for asynchronous loading.

ES modules also have new benefits:

- The syntax is even more compact than CommonJS’s.
- Modules have *static* structures (which can’t be changed at runtime). That helps with static checking, optimized access of imports, dead code elimination, and more.
- Support for cyclic imports is completely transparent.

This is an example of ES module syntax:

```js
import {importedFunc1} from './other-module1.mjs';
import {importedFunc2} from './other-module2.mjs';

function internalFunc() {
  ···
}

export function exportedFunc() {
  importedFunc1();
  importedFunc2();
  internalFunc();
}
```

From now on, “module” means “ECMAScript module”.

#### 24.5.1 ES modules: syntax, semantics, loader API

The full standard of ES modules comprises the following parts:

1. Syntax (how code is written): What is a module? How are imports and exports declared? Etc.
2. Semantics (how code is executed): How are variable bindings exported? How are imports connected with exports? Etc.
3. A programmatic loader API for configuring module loading.

Parts 1 and 2 were introduced with ES6. Work on part 3 is ongoing.

### 24.6 Named exports and imports

#### 24.6.1 Named exports



Each module can have zero or more *named exports*.

As an example, consider the following two files:

```
lib/my-math.mjs
main.mjs
```

Module `my-math.mjs` has two named exports: `square` and `LIGHTSPEED`.

```
// Not exported, private to module
function times(a, b) {
  return a * b;
}
export function square(x) {
  return times(x, x);
}
export const LIGHTSPEED = 299792458;
```

To export something, we put the keyword `export` in front of a declaration. Entities that are not exported are private to a module and can’t be accessed from outside.

#### 24.6.2 Named imports



Module `main.mjs` has a single named import, `square`:

```
import {square} from './lib/my-math.mjs';
assert.equal(square(3), 9);
```

It can also rename its import:

```
import {square as sq} from './lib/my-math.mjs';
assert.equal(sq(3), 9);
```

##### 24.6.2.1 Syntactic pitfall: named importing is not destructuring

Both named importing and destructuring look similar:

```
import {foo} from './bar.mjs'; // import
const {foo} = require('./bar.mjs'); // destructuring
```

But they are quite different:

- Imports remain connected with their exports.

- You can destructure again inside a destructuring pattern, but the `{}` in an import statement can’t be nested.

- The syntax for renaming is different:

  ```
  import {foo as f} from './bar.mjs'; // importing
  const {foo: f} = require('./bar.mjs'); // destructuring
  ```

  Rationale: Destructuring is reminiscent of an object literal (including nesting), while importing evokes the idea of renaming.

![img](https://exploringjs.com/impatient-js/img-book/img/icons/puzzle-piece-regular.svg) **Exercise: Named exports**

```
exercises/modules/export_named_test.mjs
```

#### 24.6.3 Namespace imports



*Namespace imports* are an alternative to named imports. If we namespace-import a module, it becomes an object whose properties are the named exports. This is what `main.mjs` looks like if we use a namespace import:

```
import * as myMath from './lib/my-math.mjs';
assert.equal(myMath.square(3), 9);

assert.deepEqual(
  Object.keys(myMath), ['LIGHTSPEED', 'square']);
```

#### 24.6.4 Named exporting styles: inline versus clause (advanced)

The named export style we have seen so far was *inline*: We exported entities by prefixing them with the keyword `export`.

But we can also use separate *export clauses*. For example, this is what `lib/my-math.mjs` looks like with an export clause:

```
function times(a, b) {
  return a * b;
}
function square(x) {
  return times(x, x);
}
const LIGHTSPEED = 299792458;

export { square, LIGHTSPEED }; // semicolon!
```

With an export clause, we can rename before exporting and use different names internally:

```
function times(a, b) {
  return a * b;
}
function sq(x) {
  return times(x, x);
}
const LS = 299792458;

export {
  sq as square,
  LS as LIGHTSPEED, // trailing comma is optional
};
```

### 24.7 Default exports and imports



Each module can have at most one *default export*. The idea is that the module *is* the default-exported value.

![img](https://exploringjs.com/impatient-js/img-book/img/icons/lightbulb-regular.svg) **Avoid mixing named exports and default exports**

A module can have both named exports and a default export, but it’s usually better to stick to one export style per module.

As an example for default exports, consider the following two files:

```
my-func.mjs
main.mjs
```

Module `my-func.mjs` has a default export:

```
const GREETING = 'Hello!';
export default function () {
  return GREETING;
}
```

Module `main.mjs` default-imports the exported function:

```
import myFunc from './my-func.mjs';
assert.equal(myFunc(), 'Hello!');
```

Note the syntactic difference: the curly braces around named imports indicate that we are reaching *into* the module, while a default import *is* the module.

![img](https://exploringjs.com/impatient-js/img-book/img/icons/question-circle-regular.svg) **What are use cases for default exports?**

The most common use case for a default export is a module that contains a single function or a single class.

#### 24.7.1 The two styles of default-exporting

There are two styles of doing default exports.

First, you can label existing declarations with `export default`:

```
export default function foo() {} // no semicolon!
export default class Bar {} // no semicolon!
```

Second, you can directly default-export values. In that style, `export default` is itself much like a declaration.

```
export default 'abc';
export default foo();
export default /^xyz$/;
export default 5 * 7;
export default { no: false, yes: true };
```

##### 24.7.1.1 Why are there two default export styles?

The reason is that `export default` can’t be used to label `const`: `const` may define multiple values, but `export default` needs exactly one value. Consider the following hypothetical code:

```
// Not legal JavaScript!
export default const foo = 1, bar = 2, baz = 3;
```

With this code, you don’t know which one of the three values is the default export.

![img](https://exploringjs.com/impatient-js/img-book/img/icons/puzzle-piece-regular.svg) **Exercise: Default exports**

```
exercises/modules/export_default_test.mjs
```

#### 24.7.2 The default export as a named export (advanced)

Internally, a default export is simply a named export whose name is `default`. As an example, consider the previous module `my-func.mjs` with a default export:

```
const GREETING = 'Hello!';
export default function () {
  return GREETING;
}
```

The following module `my-func2.mjs` is equivalent to that module:

```
const GREETING = 'Hello!';
function greet() {
  return GREETING;
}

export {
  greet as default,
};
```

For importing, we can use a normal default import:

```
import myFunc from './my-func2.mjs';
assert.equal(myFunc(), 'Hello!');
```

Or we can use a named import:

```
import {default as myFunc} from './my-func2.mjs';
assert.equal(myFunc(), 'Hello!');
```

The default export is also available via property `.default` of namespace imports:

```
import * as mf from './my-func2.mjs';
assert.equal(mf.default(), 'Hello!');
```

![img](https://exploringjs.com/impatient-js/img-book/img/icons/question-circle-regular.svg) **Isn’t `default` illegal as a variable name?**

`default` can’t be a variable name, but it can be an export name and it can be a property name:

```
const obj = {
  default: 123,
};
assert.equal(obj.default, 123);
```

### 24.8 More details on exporting and importing

#### 24.8.1 Imports are read-only views on exports

So far, we have used imports and exports intuitively, and everything seems to have worked as expected. But now it is time to take a closer look at how imports and exports are really related.

Consider the following two modules:

```
counter.mjs
main.mjs
```

`counter.mjs` exports a (mutable!) variable and a function:

```
export let counter = 3;
export function incCounter() {
  counter++;
}
```

`main.mjs` name-imports both exports. When we use `incCounter()`, we discover that the connection to `counter` is live – we can always access the live state of that variable:

```
import { counter, incCounter } from './counter.mjs';

// The imported value `counter` is live
assert.equal(counter, 3);
incCounter();
assert.equal(counter, 4);
```

Note that while the connection is live and we can read `counter`, we cannot change this variable (e.g., via `counter++`).

There are two benefits to handling imports this way:

- It is easier to split modules because previously shared variables can become exports.
- This behavior is crucial for supporting transparent cyclic imports. Read on for more information.

#### 24.8.2 ESM’s transparent support for cyclic imports (advanced)

ESM supports cyclic imports transparently. To understand how that is achieved, consider the following example: fig. [7](https://exploringjs.com/impatient-js/ch_modules.html#fig:module-imports) shows a directed graph of modules importing other modules. P importing M is the cycle in this case.

![img](https://exploringjs.com/impatient-js/img-book/ac2a6c966087141ba9f77be190a2a8a63c8fc15f.svg)Figure 7: A directed graph of modules importing modules: M imports N and O, N imports P and Q, etc.

After parsing, these modules are set up in two phases:

- Instantiation: Every module is visited and its imports are connected to its exports. Before a parent can be instantiated, all of its children must be instantiated.
- Evaluation: The bodies of the modules are executed. Once again, children are evaluated before parents.

This approach handles cyclic imports correctly, due to two features of ES modules:

- Due to the static structure of ES modules, the exports are already known after parsing. That makes it possible to instantiate P before its child M: P can already look up M’s exports.

- When P is evaluated, M hasn’t been evaluated, yet. However, entities in P can already mention imports from M. They just can’t use them, yet, because the imported values are filled in later. For example, a function in P can access an import from M. The only limitation is that we must wait until after the evaluation of M, before calling that function.

  Imports being filled in later is enabled by them being “live immutable views” on exports.

### 24.9 npm packages



The *npm software registry* is the dominant way of distributing JavaScript libraries and apps for Node.js and web browsers. It is managed via the *npm package manager* (short: *npm*). Software is distributed as so-called *packages*. A package is a directory containing arbitrary files and a file `package.json` at the top level that describes the package. For example, when npm creates an empty package inside a directory `foo/`, you get this `package.json`:

```
{
  "name": "foo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Some of these properties contain simple metadata:

- `name` specifies the name of this package. Once it is uploaded to the npm registry, it can be installed via `npm install foo`.

- ```
  version
  ```

   

  is used for version management and follows

   

  semantic versioning

  , with three numbers:

  - Major version: is incremented when incompatible API changes are made.
  - Minor version: is incremented when functionality is added in a backward compatible manner.
  - Patch version: is incremented when backward compatible changes are made.

- `description`, `keywords`, `author` make it easier to find packages.

- `license` clarifies how you can use this package.

Other properties enable advanced configuration:

- `main`: specifies the module that “is” the package (explained later in this chapter).
- `scripts`: are commands that you can execute via `npm run`. For example, the script `test` can be executed via `npm run test`.

For more information on `package.json`, consult [the npm documentation](https://docs.npmjs.com/files/package.json).

#### 24.9.1 Packages are installed inside a directory `node_modules/`



npm always installs packages inside a directory `node_modules`. There are usually many of these directories. Which one npm uses, depends on the directory where one currently is. For example, if we are inside a directory `/tmp/a/b/`, npm tries to find a `node_modules` in the current directory, its parent directory, the parent directory of the parent, etc. In other words, it searches the following *chain* of locations:

- `/tmp/a/b/node_modules`
- `/tmp/a/node_modules`
- `/tmp/node_modules`

When installing a package `foo`, npm uses the closest `node_modules`. If, for example, we are inside `/tmp/a/b/` and there is a `node_modules` in that directory, then npm puts the package inside the directory:

```
/tmp/a/b/node_modules/foo/
```

When importing a module, we can use a special module specifier to tell Node.js that we want to import it from an installed package. How exactly that works, is explained later. For now, consider the following example:

```
// /home/jane/proj/main.mjs
import * as theModule from 'the-package/the-module.mjs';
```

To find `the-module.mjs` (Node.js prefers the filename extension `.mjs` for ES modules), Node.js walks up the `node_module` chain and searches the following locations:

- `/home/jane/proj/node_modules/the-package/the-module.mjs`
- `/home/jane/node_modules/the-package/the-module.mjs`
- `/home/node_modules/the-package/the-module.mjs`

#### 24.9.2 Why can npm be used to install frontend libraries?

Finding installed modules in `node_modules` directories is only supported on Node.js. So why can we also use npm to install libraries for browsers?

That is enabled via [bundling tools](https://exploringjs.com/impatient-js/ch_remaining-chapters-site.html), such as webpack, that compile and optimize code before it is deployed online. During this compilation process, the code in npm packages is adapted so that it works in browsers.

### 24.10 Naming modules

There are no established best practices for naming module files and the variables they are imported into.

In this chapter, I’m using the following naming style:

- The names of module files are dash-cased and start with lowercase letters:

  ```
  ./my-module.mjs
  ./some-func.mjs
  ```

- The names of namespace imports are lowercased and camel-cased:

  ```
  import * as myModule from './my-module.mjs';
  ```

- The names of default imports are lowercased and camel-cased:

  ```
  import someFunc from './some-func.mjs';
  ```

What are the rationales behind this style?

- npm doesn’t allow uppercase letters in package names ([source](https://docs.npmjs.com/files/package.json#name)). Thus, we avoid camel case, so that “local” files have names that are consistent with those of npm packages. Using only lowercase letters also minimizes conflicts between file systems that are case-sensitive and file systems that aren’t: the former distinguish files whose names have the same letters, but with different cases; the latter don’t.
- There are clear rules for translating dash-cased file names to camel-cased JavaScript variable names. Due to how we name namespace imports, these rules work for both namespace imports and default imports.

I also like underscore-cased module file names because you can directly use these names for namespace imports (without any translation):

```
import * as my_module from './my_module.mjs';
```

But that style does not work for default imports: I like underscore-casing for namespace objects, but it is not a good choice for functions, etc.

### 24.11 Module specifiers



*Module specifiers* are the strings that identify modules. They work slightly differently in browsers and Node.js. Before we can look at the differences, we need to learn about the different categories of module specifiers.

#### 24.11.1 Categories of module specifiers

In ES modules, we distinguish the following categories of specifiers. These categories originated with CommonJS modules.

- Relative path: starts with a dot. Examples:

  ```
  './some/other/module.mjs'
  '../../lib/counter.mjs'
  ```

- Absolute path: starts with a slash. Example:

  ```
  '/home/jane/file-tools.mjs'
  ```

- URL: includes a protocol (technically, paths are URLs, too). Examples:

  ```
  'https://example.com/some-module.mjs'
  'file:///home/john/tmp/main.mjs'
  ```

- Bare path: does not start with a dot, a slash or a protocol, and consists of a single filename without an extension. Examples:

  ```
  'lodash'
  'the-package'
  ```

- Deep import path: starts with a bare path and has at least one slash. Example:

  ```
  'the-package/dist/the-module.mjs'
  ```

#### 24.11.2 ES module specifiers in browsers

Browsers handle module specifiers as follows:

- Relative paths, absolute paths, and URLs work as expected. They all must point to real files (in contrast to CommonJS, which lets you omit filename extensions and more).
- The file name extensions of modules don’t matter, as long as they are served with the content type `text/javascript`.
- How bare paths will end up being handled is not yet clear. You will probably eventually be able to map them to other specifiers via lookup tables.

Note that [bundling tools](https://exploringjs.com/impatient-js/ch_remaining-chapters-site.html) such as webpack, which combine modules into fewer files, are often less strict with specifiers than browsers. That’s because they operate at build/compile time (not at runtime) and can search for files by traversing the file system.

#### 24.11.3 ES module specifiers on Node.js

![img](https://exploringjs.com/impatient-js/img-book/img/icons/exclamation-triangle-regular.svg) **Support for ES modules on Node.js is still new**

You may have to switch it on via a command line flag. See [the Node.js documentation](https://nodejs.org/api/esm.html) for details.

Node.js handles module specifiers as follows:

- Relative paths are resolved as they are in web browsers – relative to the path of the current module.
- Absolute paths are currently not supported. As a workaround, you can use URLs that start with `file:///`. You can create such URLs via [`url.pathToFileURL()`](https://exploringjs.com/impatient-js/ch_modules.html#converting-urls-paths).
- Only `file:` is supported as a protocol for URL specifiers.
- A bare path is interpreted as a package name and resolved relative to the closest `node_modules` directory. What module should be loaded, is determined by looking at property `"main"` of the package’s `package.json` (similarly to CommonJS).
- Deep import paths are also resolved relatively to the closest `node_modules` directory. They contain file names, so it is always clear which module is meant.

All specifiers, except bare paths, must refer to actual files. That is, ESM does not support the following CommonJS features:

- CommonJS automatically adds missing filename extensions.
- CommonJS can import a directory `foo` if there is a `foo/package.json` with a `"main"` property.
- CommonJS can import a directory `foo` if there is a module `foo/index.js`.

All built-in Node.js modules are available via bare paths and have named ESM exports – for example:

```
import * as path from 'path';
import {strict as assert} from 'assert';

assert.equal(
  path.join('a/b/c', '../d'), 'a/b/d');
```

##### 24.11.3.1 Filename extensions on Node.js

Node.js supports the following default filename extensions:

- `.mjs` for ES modules
- `.cjs` for CommonJS modules

The filename extension `.js` stands for either ESM or CommonJS. Which one it is is configured via the “closest” `package.json` (in the current directory, the parent directory, etc.). Using `package.json` in this manner is independent of packages.

In that `package.json`, there is a property `"type"`, which has two settings:

- `"commonjs"` (the default): files with the extension `.js` or without an extension are interpreted as CommonJS modules.
- `"module"`: files with the extension `.js` or without an extension are interpreted as ESM modules.

##### 24.11.3.2 Interpreting non-file source code as either CommonJS or ESM

Not all source code executed by Node.js comes from files. You can also send it code via stdin, `--eval`, and `--print`. The command line option `--input-type` lets you specify how such code is interpreted:

- As CommonJS (the default): `--input-type=commonjs`
- As ESM: `--input-type=module`

### 24.12 Loading modules dynamically via `import()`



So far, the only way to import a module has been via an `import` statement. That statement has several limitations:

- You must use it at the top level of a module. That is, you can’t, for example, import something when you are inside a block.
- The module specifier is always fixed. That is, you can’t change what you import depending on a condition. And you can’t assemble a specifier dynamically.

The `import()` operator changes that. Let’s look at an example of it being used.

#### 24.12.1 Example: loading a module dynamically

Consider the following files:

```
lib/my-math.mjs
main1.mjs
main2.mjs
```

We have already seen module `my-math.mjs`:

```
// Not exported, private to module
function times(a, b) {
  return a * b;
}
export function square(x) {
  return times(x, x);
}
export const LIGHTSPEED = 299792458;
```

This is what using `import()` looks like in `main1.mjs`:

```
const dir = './lib/';
const moduleSpecifier = dir + 'my-math.mjs';

function loadConstant() {
  return import(moduleSpecifier)
  .then(myMath => {
    const result = myMath.LIGHTSPEED;
    assert.equal(result, 299792458);
    return result;
  });
}
```

Method `.then()` is part of *Promises*, a mechanism for handling asynchronous results, which is covered [later in this book](https://exploringjs.com/impatient-js/ch_promises.html).

Two things in this code weren’t possible before:

- We are importing inside a function (not at the top level).
- The module specifier comes from a variable.

Next, we’ll implement the exact same functionality in `main2.mjs` but via a so-called *async function*, which provides nicer syntax for Promises.

```
const dir = './lib/';
const moduleSpecifier = dir + 'my-math.mjs';

async function loadConstant() {
  const myMath = await import(moduleSpecifier);
  const result = myMath.LIGHTSPEED;
  assert.equal(result, 299792458);
  return result;
}
```

![img](https://exploringjs.com/impatient-js/img-book/img/icons/question-circle-regular.svg) **Why is `import()` an operator and not a function?**

Even though it works much like a function, `import()` is an operator: in order to resolve module specifiers relatively to the current module, it needs to know from which module it is invoked. A normal function cannot receive this information as implicitly as an operator can. It would need, for example, a parameter.

#### 24.12.2 Use cases for `import()`

##### 24.12.2.1 Loading code on demand

Some functionality of web apps doesn’t have to be present when they start, it can be loaded on demand. Then `import()` helps because you can put such functionality into modules – for example:

```
button.addEventListener('click', event => {
  import('./dialogBox.mjs')
    .then(dialogBox => {
      dialogBox.open();
    })
    .catch(error => {
      /* Error handling */
    })
});
```

##### 24.12.2.2 Conditional loading of modules

We may want to load a module depending on whether a condition is true. For example, a module with [a polyfill](https://exploringjs.com/impatient-js/ch_modules.html#polyfills) that makes a new feature available on legacy platforms:

```
if (isLegacyPlatform()) {
  import('./my-polyfill.mjs')
    .then(···);
}
```

##### 24.12.2.3 Computed module specifiers

For applications such as internationalization, it helps if you can dynamically compute module specifiers:

```
import(`messages_${getLocale()}.mjs`)
  .then(···);
```

### 24.13 Preview: `import.meta.url`

[“`import.meta`”](https://github.com/tc39/proposal-import-meta) is an ECMAScript feature proposed by Domenic Denicola. The object `import.meta` holds metadata for the current module.

Its most important property is `import.meta.url`, which contains a string with the URL of the current module file. For example:

```
'https://example.com/code/main.mjs'
```

#### 24.13.1 `import.meta.url` and class `URL`

Class `URL` is available via a global variable in browsers and on Node.js. You can look up its full functionality in [the Node.js documentation](https://nodejs.org/api/url.html#url_class_url). When working with `import.meta.url`, its constructor is especially useful:

```
new URL(input: string, base?: string|URL)
```

Parameter `input` contains the URL to be parsed. It can be relative if the second parameter, `base`, is provided.

In other words, this constructor lets us resolve a relative path against a base URL:

```
> new URL('other.mjs', 'https://example.com/code/main.mjs').href
'https://example.com/code/other.mjs'
> new URL('../other.mjs', 'https://example.com/code/main.mjs').href
'https://example.com/other.mjs'
```

This is how we get a `URL` instance that points to a file `data.txt` that sits next to the current module:

```
const urlOfData = new URL('data.txt', import.meta.url);
```

#### 24.13.2 `import.meta.url` on Node.js

On Node.js, `import.meta.url` is always a string with a `file:` URL – for example:

```
'file:///Users/rauschma/my-module.mjs'
```

##### 24.13.2.1 Example: reading a sibling file of a module

Many Node.js file system operations accept either strings with paths or instances of `URL`. That enables us to read a sibling file `data.txt` of the current module:

```
import {promises as fs} from 'fs';

async function main() {
  const urlOfData = new URL('data.txt', import.meta.url);
  const str = await fs.readFile(urlOfData, {encoding: 'UTF-8'});
  assert.equal(str, 'This is textual data.\n');
}
main();
```

`main()` is an async function, as explained in [§38 “Async functions”](https://exploringjs.com/impatient-js/ch_async-functions.html).

[`fs.promises`](https://nodejs.org/api/fs.html#fs_fs_promises_api) contains a Promise-based version of the `fs` API, which can be used with async functions.

##### 24.13.2.2 Converting between `file:` URLs and paths

[The Node.js module `url`](https://nodejs.org/api/url.html) has two functions for converting between `file:` URLs and paths:

- `fileURLToPath(url: URL|string): string`
  Converts a `file:` URL to a path.
- `pathToFileURL(path: string): URL`
  Converts a path to a `file:` URL.

If you need a path that can be used in the local file system, then property `.pathname` of `URL` instances does not always work:

```
assert.equal(
  new URL('file:///tmp/with%20space.txt').pathname,
  '/tmp/with%20space.txt');
```

Therefore, it is better to use `fileURLToPath()`:

```
import * as url from 'url';
assert.equal(
  url.fileURLToPath('file:///tmp/with%20space.txt'),
  '/tmp/with space.txt'); // result on Unix
```

Similarly, `pathToFileURL()` does more than just prepend `'file://'` to an absolute path.

### 24.14 Polyfills: emulating native web platform features (advanced)



![img](https://exploringjs.com/impatient-js/img-book/img/icons/cogs-regular.svg) **Backends have polyfills, too**

This section is about frontend development and web browsers, but similar ideas apply to backend development.

*Polyfills* help with a conflict that we are facing when developing a web application in JavaScript:

- On one hand, we want to use modern web platform features that make the app better and/or development easier.
- On the other hand, the app should run on as many browsers as possible.

Given a web platform feature X:

- A

   

  polyfill

   

  for X is a piece of code. If it is executed on a platform that already has built-in support for X, it does nothing. Otherwise, it makes the feature available on the platform. In the latter case, the polyfilled feature is (mostly) indistinguishable from a native implementation. In order to achieve that, the polyfill usually makes global changes. For example, it may modify global data or configure a global module loader. Polyfills are often packaged as modules.

  - The term [*polyfill*](https://remysharp.com/2010/10/08/what-is-a-polyfill) was coined by Remy Sharp.

- A

   

  speculative polyfill

   

  is a polyfill for a proposed web platform feature (that is not standardized, yet).

  - Alternative term: *prollyfill*

- A

   

  replica

   

  of X is a library that reproduces the API and functionality of X locally. Such a library exists independently of a native (and global) implementation of X.

  - *Replica* is a new term introduced in this section. Alternative term: *ponyfill*

- There is also the term *shim*, but it doesn’t have a universally agreed upon definition. It often means roughly the same as *polyfill*.

Every time our web applications starts, it must first execute all polyfills for features that may not be available everywhere. Afterwards, we can be sure that those features are available natively.

#### 24.14.1 Sources of this section

- [“What is a Polyfill?”](https://remysharp.com/2010/10/08/what-is-a-polyfill) by Remy Sharp
- Inspiration for the term *replica*: [The Eiffel Tower in Las Vegas](https://en.wikipedia.org/wiki/Paris_Las_Vegas)
- Useful clarification of “polyfill” and related terms: [“Polyfills and the evolution of the Web”](https://www.w3.org/2001/tag/doc/polyfills/). Edited by Andrew Betts.