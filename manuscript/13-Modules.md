# Encapsulating Code With Modules

One of the most error-prone and confusing aspects of JavaScript has long been the "shared everything" approach to loading code. Whereas other languages have concepts such as packages, JavaScript lagged behind, and everything defined in every file shared the same global scope. As web applications became more complex and the amount of JavaScript used grew, the "shared everything" approach began to show problems with naming collisions, security concerns, and more. One of the goals of ECMAScript 6 was to solve this problem and bring some order into JavaScript applications. That's where modules come in.

## What are Modules?

*Modules* are JavaScript files that are loaded in a different mode (as opposed to *scripts*, which are loaded in the original way JavaScript worked). The reason this different mode is necessary is because module files have very different semantics than script files:

1. Module code automatically runs in strict mode and there's no way to opt-out of strict mode.
1. Variables created in the top level of a module are not automatically added to the shared global scope. They exist only within the top-level scope of the module.
1. The value of `this` in the top level of a module is `undefined`.
1. Modules do not allow HTML-style comments within the code (a leftover feature from the early browser days).
1. Modules must export anything that should be available to code outside of the module.
1. Modules may import bindings from other modules.

These differences may seem small at first glance, however, they represent a significant change in how JavaScript code is loaded and evaluated.

The real power of modules is the ability to export and importing only those bindings that are required, rather than everything in a file. A good understanding of exporting and importing is fundamental to understanding how modules differ from scripts.

## Basic Exporting and Importing

The new `export` keyword is used to expose parts of published code to other modules. In the simplest case, you can place `export` in front of any variable, function, or class declaration to export it from the module. For example:

```js
// export data
export var color = "red";
export let name = "Nicholas";
export const magicNumber = 7;

// export function
export function sum(num1, num2) {
    return num1 + num1;
}

// export class
export class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
}

// this function is private to the module
function subtract(num1, num2) {
    return num1 - num2;
}

// define a function...
function multiply(num1, num2) {
    return num1 * num2;
}

// ...and then export it later
export multiply;
```

There are a few things to notice in this example:

1. Every declaration is exactly the same as it would otherwise be without the `export` keyword.
1. Exported function and class declarations require a name. You cannot export anonymous functions or classes using this syntax (unless using the `default` keyword, discussed later in this chapter)
1. You need not always export the declaration, you can also export references, as with `multiply` in this example.
1. Any variables, functions, or classes that are not explicitly exported remain private to the module. In this example, `subtract()` is not exported and is therefore not accessible from outside the module.

Once you have a module with exports, you can access the functionality in another module by using the `import` keyword. An `import` statement has two parts: the identifiers you're importing and the module from which those identifiers should be imported. The basic form is as follows:

```js
import { identifier1, identifier2 } from "example.js";
```

The curly braces after `import` indicate the bindings to import from the given module. The keyword `from` is used to indicate the module from which to import the given bindings. The module is specified using a string representing the path to the module. Browsers use the same format as you might pass to `<script>`, which means you must include a file extension, whereas Node.js follows its traditional conventions of differentiating between local files and packages based on a filesystem prefix (for example, `example` for a package and `./example` for a local file) and does not require file extensions. Specific differences in how browsers and Node.js load modules are discussed later in this chapter.

I> Even though it looks similar, the list of bindings to import is not a destructured object.

When importing a binding from a module, the binding acts as if it were defined using `const`. That means you cannot define another variable with the same name, use the identifier prior to the `import` statement, or change its value.

Suppose that the first example in this section is in a module named `"example.js"`. You can import and use bindings from that module in a number of ways. You can just import one identifier:

```js
// import just one
import { sum } from "example.js";

console.log(sum(1, 2));     // 3

sum = 1;        // error
```

This example imports only the `sum()` function from the example module. Even though the example module exports more than just that one function, they are not exposed here. If you try to assign a new value to `sum`, the result is an error, as you cannot reassign imported bindings.

If you want to import multiple bindings from the example module, you can explicitly list them out:

```js
// import multiple
import { sum, multiply, magicNumber } from "example.js";
console.log(sum(1, magicNumber));   // 8
console.log(multiply(1, 2));        // 2
```

Here, three bindings are imported from the example module: `sum`, `multiply`, and `magicNumber`. They are then used as if they were locally defined.

There's also a special case that allows you to import the entire module as a single object. All of the exports are then available on that object as properties. For example:

```js
// import everything
import * as example from "example.js";
console.log(example.sum(1,
        example.magicNumber));          // 8
console.log(example.multiply(1, 2));    // 2
```

In this code, the entirety of the example module is loaded into an object called `example`. The named exports `sum()`, `multiple()`, and `magicNumber` are then accessible as properties on `example`.

Keep in mind that the code inside of a module will only ever be executed once, regardless of the number of times it's used in an `import` statement. Consider the following:

```js
import { sum } from "example.js";
import { multiply } from "example.js";
import { magicNumber } from "example.js";
```

Even though there are three `import` statements in this module, the code in `"example"` will only be executed once. The instantiated module is then kept in memory and reused whenever another `import` statement references it. It doesn't matter if the `import` statements are all in the module, or are spread across multiple modules -- they each will use the same module instance.

A> ### Syntax Limitation
A>
A> An important limitation of both `export` and `import` is that they must be used outside of other statements. For instance, this is a syntax error:
A>
A> ```js
A> if (flag) {
A>     export flag;    // syntax error
A> }
A> ```
A>
A> This example is a syntax error because `export` is inside of an `if` statement. Exports cannot be conditional or done dynamically in any way. Part of the benefit of module syntax is so the JavaScript engine can staticly determine what will be exported. As such, you can only use `export` at the top-level of a module. Similarly, you cannot use `import` inside of a statement:
A>
A> ```js
A> if (!flag) {
A>     import flag from "example.js";    // syntax error
A> }
A> ```
A>
A> The `export` and `import` keywords are designed to be static so that tools such as text editors can easily tell what information is available. For that reason, you can't dynamically export or import bindings, and that reason requires the syntax limitation of using `export` and `import` outside of either statements.

## Renaming Exports and Imports

Sometimes the original name of a variable, function, or class isn't what you want to use. It's possible to change the name of an export both during the export and during the import.

In the first case, suppose you have a function that you'd like to export with a different name. You can use the `as` keyword to specify the name that the function should be known as outside of the module:

```js
function sum(num1, num2) {
    return num1 + num2;
}

export { sum as add };
```

Here, the `sum()` function (`sum` is the *local name*) is exported as `add()` (`add` is the *exported name*). That means when another module wants to import this function, it will have to use the name `add` instead:

```js
import { add } from "example.js";
```

If the module importing the function wants to use a different name, it can also use `as`:

```js
import { add as sum } from "example.js";
console.log(typeof add);            // "undefined"
console.log(sum(1, 2));             // 3
```

This code imports the `add()` function (the *import name*) and renames it to `sum()` (the local name). That means there is no identifier named `add` in this module.

A> ### Imported Bindings
A>
A> A subtle but important point about the `import` statements is that they create read-only bindings to variables, functions, and classes rather than simply referencing them (as is the case with normal variables). That means even though you cannot change an imported identifier, it can still be changed by the module that exported it. For example, suppose you have this module:
A>
A> ```js
A> export var name = "Nicholas";
A> export function setName(newName) {
A>     name = newName;
A> }
A> ```
A>
A> When you import `name` and `setName()`, you can see that `setName()` is able to change the value of `name`:
A>
A> ```js
A> import { name, setName } from "example.js";
A>
A> console.log(name);       // "Nicholas"
A> setName("Greg");
A> console.log(name);       // "Greg"
A>
A> name = "Nicholas";       // error
A> ```
A>
A> The call to `setName("Greg")` goes back into the module from which `setName()` was exported and executes there, setting `name` to `"Greg"`. Note this change is automatically reflected on the imported `name` binding. That's because `name` is the local name for the exported *name* identifier so they are not the same thing.

## Exporting and Importing Defaults

The module syntax is really optimized for exporting and importing default values from modules. The default value for a module is a single variable, function, or class as specified by the `default` keyword. For example:

```js
export default function(num1, num2) {
    return num1 + num2;
}
```

This module exports a function as the default. The `default` keyword indicates that this is a default export and the function doesn't require a name because the module itself represents the function.

You can also specify an identifier as being the default export using the renaming syntax, such as:

```js
// equivalent to previous example
function sum(num1, num2) {
    return num1 + num2;
}

export { sum as default };
```

The `as default` specifies that `sum` should be the default export of the module. This syntax is equivalent to the previous example.

W> You can only have one default export per module. It is a syntax error to use the `default` keyword with multiple exports.

You can import a default value from a module using the following syntax:

```js
// import the default
import sum from "example.js";

console.log(sum(1, 2));     // 3
```

This import statement imports the default from the module `"example.js"`. Note that there are no curly braces used in this case, as would be with a non-default export. The local name `sum` is used to represent the function that the module exports. This syntax is the cleanest as it's anticipated to be the dominant form of import on the web, allowing you to use already-existing object.

For modules that export both a default and one or more non-defaults, you can import them with one statement. For instance, suppose you have this module:

```js
export let color = "red";

export default function(num1, num2) {
    return num1 + num2;
}
```

You can then import both `color` and the default function using the following:

```js
import sum, { color } from "example.js";

console.log(sum(1, 2));     // 3
console.log(color);         // "red"
```

The comma separates the default local name from the non-defaults (which are also surrounded by curly braces).

As with exporting defaults, importing defaults can also be accomplished using the renaming syntax:

```js
// equivalent to previous example
import { default as sum, color } from "example";

console.log(sum(1, 2));     // 3
console.log(color);         // "red"
```

In this code, the default export (`default`) is renamed to `sum` and the additional `color` export is also imported. This example is equivalent to the preceding example.

## Re-exporting

There may be a time when you'd like to re-export something that your module has imported (for instance, if you are creating a library out of several small modules). You can do this using the patterns already discussed in this chapter, such as:

```js
import { sum } from "example.js";
export { sum }
```

However, there's also a single statement that can accomplish the same thing:

```js
export { sum } from "example.js";
```

This form of `export` looks into the specified module for the declaration of `sum` and then exports it. Of course, you can also choose to export a different name for the same thing:

```js
export { sum as add } from "example.js";
```

Here, `sum` is imported from `"example.js"` and then exported as `add`.

If you'd like to export everything from another module, you can use the `*` pattern:

```js
export * from "example.js";
```

By exporting everything, you're including the default as well as any named exports, which may affect what you can export from your module. For instance, if `"example.js"` has a default export, you'll be unable to define a new default export when using this syntax.

## Importing Without Bindings

Some modules may not export anything, and instead, only make modifications to objects in the global scope. Even though top-level variables, functions, and classes inside of modules do not automatically end up in the global scope, that doesn't mean modules cannot access the global scope. The shared definitions of built-in objects such as `Array` and `Object` are accessible inside of a module and changes to those objects will be reflected in other modules.

For instance, suppose you want to add a method to all arrays called `pushAll()`, you may define a module like this:

```js
// module code without exports or imports
Array.prototype.pushAll = function(items) {

    // items must be an array
    if (!Array.isArray(items)) {
        throw new TypeError("Argument must be an array.");
    }

    // use built-in push() and spread operator
    return this.push(...items);
};
```

This is a valid module even though there are no exports or imports. This code can be used both as a module and a script. Since it doesn't export anything, you can use a simplified import to execute the module code without importing any bindings:

```js
import "example";

let colors = ["red", "green", "blue"];
let items = [];

items.pushAll(colors);
```

In this example, the module is imported and executed, so `pushAll()` is added to the array prototype. That means `pushAll()` is now available for use on all arrays inside of this module.

I> Imports without bindings are most likely to be used to create polyfills and shims.

## Loading Modules

While ECMAScript 6 defined the syntax for modules, it didn't define how to load them. This is part of the complexity of a specification that is supposed to be agnostic to implementation environments. Rather than trying to create a single specification that would work for all JavaScript environments, ECMAScript 6 chose to specify only the syntax and abstract out the loading mechanism to an internal operation called ` HostResolveImportedModule`, which was left undefined. It was then up to web browsers and Node.js to decide how to implement `HostResolveImportedModule` in such a way that it made sense for their environment.

### Using Modules in Web Browsers

Even before ECMAScript 6, web browsers had multiple ways of including JavaScript in an web application:

1. Loading files of JavaScript code using the `<script>` element with the `src` attribute specifying a location from which to load the code.
1. Embedding JavaScript code inline using the `<script>` element without the `src` attribute.
1. Loading files of JavaScript code to execute as a worker (such as a web worker or service worker).

In order to fully support modules, web browsers had to update each of these script loading mechanisms. These details are defined in HTML specification and summarized in the following sections.

#### Using Modules With `<script>`

The default behavior of the `<script>` element is to load JavaScript files as scripts (not modules). This happens when the `type` attribute is missing and also when the `type` attribute contains a JavaScript content type (such as `"text/javascript"`). The `<script>` element can then execute inline code or load the file specified in `src`. To support modules, a new value for `type` was added: `"module"`. When `type` is `"module"`, any inline code or code contained in the file specified by `src` is loaded as a module instead of a script. Here's a simple example:

```html
<!-- load a module JavaScript file -->
<script type="module" src="module.js"></script>

<!-- include a module inline -->
<script type="module">

import { sum } from "example.js";

let result = sum(1, 2);

</script>
```

The first `<script>` element in this example loads an external module file using the `src` attribute. The only difference from loading a script is that `type` is `"module"`. The second `<script>` element contains a module that is embedded directly in the web page. The variable `result` is not exposed globally because it exists only within the module (as defined by the `<script>` element) and is therefore not added to `window` as a property.

As you can see, including modules in web pages is fairly simple and similar to including scripts. However, there are some differences in how modules are loaded.

I> You may have noticed that `"module"` is not a content type like `"text/javascript"`. Module JavaScript files are served with the same content type as script JavaScript files, so it's not possible to differentiate solely based on content type. Also, browsers ignore `<script>` elements when the `type` is unrecognized, so browsers that don't support modules will automatically ignore `<script type="module">`, providing good backwards-compatibility.

#### Module Loading Sequence in Web Browsers

Modules are unique in that, unlike scripts, they may specify that other files must be loaded in order to execute correctly (using `import`). In order to support this, `<script type="module">` always acts as if the `defer` attribute is applied. The `defer` attribute is optional for loading script files but is always applied for loading module files. That means the module file begins downloading as soon as the HTML parser encounters `<script type="module">` with a `src` attribute but does not execute until after the document has been completely parsed. Also, modules are executed in the order in which they appear, so the first `<script type="module">` is always guaranteed to execute before the second, even if one of them contains inline code instead of specifying `src`. For example:

```html
<!-- this will execute first -->
<script type="module" src="module1.js"></script>

<!-- this will execute second -->
<script type="module">
import { sum } from "example.js";

let result = sum(1, 2);
</script>

<!-- this will execute third -->
<script type="module" src="module2.js"></script>
```

These three `<script>` elements execute in the order they are specified, so `module1.js` is guaranteed to execute before the inline module, and the inline module is guaranteed to execute before `module2.js`.

Complicating the matter is that each module may `import` from one or more other modules. That's why modules are parsed completely first to identify all `import` statements. Each `import` statement then triggers a fetch (either from the network or from the cache), and the module isn't executed until all of the `import` resources have first been loaded and executed.

All of the modules, both those explicitly included using `<script type="module">` and those implicitly included using `import`, are loaded and executed in order. In the preceding example, the complete loading sequence is:

1. Download and parse `module1.js`
1. Recursively download and parse `import` resources in `module1.js`
1. Parse the inline module
1. Recursively download and parse `import` resources the inline module
1. Download and parse `module2.js`
1. Recursively download and parse `import` resources in `module2.js`

Once loading is complete, nothing is executed until after the document has been completely parsed. Once document parsing is complete, then the following takes place:

1. Recursively execute `import` resources for `module1.js`
1. Execute `module1.js`
1. Recursively execute `import` resources for the inline module
1. Execute the inline module
1. Recursively execute `import` resources for `module2.js`
1. Execute `module2.js`

Notice that the inline module acts like the other two modules except that the code doesn't first have to be downloaded. Otherwise, the sequence of loading `import` resources and executing modules is exactly the same.

I> The `defer` attribute is ignored on `<script type="module">` because it already behaves as if `defer` is applied.

#### Asynchronous Module Loading in Web Browsers

You may already be familiar with the `async` attribute on the `<script>` element. When used with scripts, `async` causes the script file to be executed as soon as the file is completely downloaded and parsed. Additionally, the order of `async` scripts in the document does not affect the order in which the scripts are executed -- they are always executed as soon as they finish downloading without waiting for the containing document to finish parsing.

The `async` attribute can be applied to modules as well. When `async` is used on `<script type="module">`, the module is executed in a similar manner as a script. The only difference is that all of the `import` resources for the module are downloaded before the module itself is executed. So you are guaranteed that the module will have all of its resources downloaded in order to execute, but you cannot guarantee when the module will execute. Consider the following:

```html
<!-- no guarantee which one of these will execute first -->
<script type="module" async src="module1.js"></script>
<script type="module" async src="module2.js"></script>
```

In this example, there are two module files loaded asynchronously. It's not possible to tell which module will execute first simply by looking at this code. If `module1.js` finishes downloading first (including all of its `import` resources), then it will execute first; if `module2.js` finishes downloading first, then it will execute first. The only guarantees are that `module1.js` will have all of its `import` resources downloading before it is executed and the same for `module2.js`.

#### Loading Modules as Workers

Workers, such as web workers and service workers, execute JavaScript code outside of the web page context. Creating a new worker involves creating a new instance `Worker` (or another class) and passing in the location of JavaScript file. The default loading mechanism is to load files as scripts, such as:

```js
// load script.js as a script
let worker = new Worker("script.js");
```

To support loading modules, a second argument was added to these constructors in the HTML standard. The second argument is an object with a `type` property with a default value of `"script"`. You can set `type` to `"module"` in order to load module files:

```js
// load module.js as a module
let worker = new Worker("module.js", { type: "module" });
```

This example loads `module.js` as a module instead of a script by passing a second argument with `type` set to `"module"` (the `type` property is meant to mimic how the `type` attribute of `<script>` differentiates modules and scripts). The second argument is supported for all worker types in the browser.

Worker modules are generally the same as worker scripts, but there are a couple of exceptions:

* Worker scripts are limited to being loaded from the same origin as the web page in which they are referenced. Worker modules have this same default restriction, however, they can also load files that have appropriate Cross-Origin Resource Sharing (CORS) headers to allow access.
* Worker scripts can use the `self.importScripts()` method to load additional scripts into the worker. In worker modules, `self.importScripts()` always fails because you should use `import` instead.

### Using Modules in Node.js

TODO

## Summary

ECMAScript 6 adds modules to the language as a way to package up and encapsulate functionality. Modules behave differently than scripts, as they do not modify the global scope with their top-level variables, functions, and classes, and `this` is `undefined`. In order to work differently than scripts, modules must be loaded using a different mode.

You must export any functionality you'd like to make available to consumers of a module. Variables, functions, and classes can all be exported, and there is also one default export allowed per module. After exporting, another module can import all or some of the exported names. These names act as if defined by `let`, and so operate as block bindings that cannot be redeclared in the same module.

Modules need not export anything if they are manipulating something in the global scope. In that case, it's possible to import from such a module without introducing any bindings into the module scope.
