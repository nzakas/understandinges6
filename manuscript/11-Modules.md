# Modules

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

One of the most error-prone and confusing aspects of JavaScript has long been the "shared everything" approach to loading code. Whereas other languages have concepts such as packages, JavaScript lagged behind, and everything defined in every file shared the same global scope. As web applications became more complex and the amount of JavaScript used grew, the "shared everything" approach began to show problems with naming collisions, security concerns, and more. One of the goals of ECMAScript 6 was to solve this problem and bring some order into JavaScript applications. That's where modules come in.

## What are Modules?

Modules are JavaScript files that are loaded in a special module mode. At the time of my writing, neither browsers nor Node.js have a way to natively load ECMAScript 6 modules, but both have indicated that there will need to be some sort of opt-in to do so. The reason this opt-in is necessary is because module files have very different semantics than non-module files:

1. Module code automatically runs in strict mode and there's no way to opt-out of strict mode.
1. Variables created in the top level of a module are not automatically added to the shared global scope. They exist only within the top-level scope of the module.
1. The value of `this` in the top level of a module is `undefined`.
1. Does not allow HTML-style comments within the code (a leftover feature from the early browser days).
1. Modules must export anything that should be available to code outside of the module.

These differences may seem small at first glance, however, they represent a significant change in how JavaScript code is loaded and evaluated.

Module JavaScript files are created just like any other JavaScript file: in a text editor and typically with the `.js` extension. The only difference during development is that you use some different syntax.

## Basic Exporting and Importing

The `export` keyword is used to expose parts of published code to other modules. In the simplest case, you can place `export` in front of any variable, function, or class declaration to export it from the module. For example:

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

// define a function
function multiply(num1, num2) {
    return num1 * num2;
}

// export later
export multiply;
```

There are a few things to notice in this example:

1. Every declaration is exactly the same as it would otherwise be without the `export` keyword.
1. Both function and class declarations require a name. You cannot export anonymous functions or classes using this syntax.
1. You need not always export the declaration, you can also export references, as with `multiply` in this example.
1. Any variables, functions, or classes that not explicitly exported remain private to the module. In this example, `subtract()` is not exported and is therefore not accessible from outside the module.

An important limitation of `export` is that it must be used in the top-level of the module. For instance, this is a syntax error:

```js
if (flag) {
    export flag;    // syntax error
}
```

This example is a syntax error because `export` is inside of an `if` statement. Exports cannot be condition or done dynamically in any way. Part of the benefit of module syntax is so the JavaScript engine can staticly determine what will be exported. As such, you can only use `export` at the top-level of a module.

W> If you are using a transpiler like Babel.js, you may find that `export` can be used anywhere. This only works when code is converted to ECMAScript 5 and will not work with a native ECMAScript 6 module system.

Once you have a module with exports, you can access the functionality in another module by using the `import` keyword. An `import` statement has two parts: the identifiers you're importing and the module from which those identifiers should be imported. The basic form is as follows:

```js
import { identifier1, identifier2 } from "module";
```

The curly braces after `import` indicate the identifiers to import from the given module. The keyword `from` is used to indicate the module from which to import the given identifiers. The module is specified using a string. At the time of my writing, it is still undecided what module identifiers will look like. They may end up being full file paths (such as "../mymodule.js"), file paths without extensions (such as "../mymodule"), or something else. This likely won't be determined until browsers and Node.js begin implementing modules natively.

N> Even though it looks similar, the list of identifiers to import is not a destructured object.

When importing an identifier from a module, the identifier acts as if were defined using `let`. That means you cannot define another variable with the same name nor can you use the identifier prior to the `import` statement.

Suppose that the first example in this section is in a module named `"example"`. You can import and use identifiers from that module in a number of ways. You can just import one identifier:

```js
// import just one
import { sum } from "example";

console.log(sum(1, 2));     // 3
```

This example imports only `sum()` from the example module. Even though the example module exports more than just that one function, they are not exposed here.

If you want to import multiple identifiers from the example module, you can explicitly list them out:

```js
// import multiple
import { sum, multiply, magicNumber } from "example";
console.log(sum(1, magicNumber));   // 8
console.log(multiply(1, 2));        // 2
```

Here, three identifiers are imported from the example module: `sum`, `multiply`, and `magicNumber`. They are then used as if they were locally defined.

There's also a special case that allows you to import the entire module as a single object. All of the exports are then available on that object as properties. For example:

```js
// import everything
import * as example from "example";
console.log(example.sum(1,
        example.magicNumber));          // 8
console.log(example.multiply(1, 2));    // 2
```

In this code, the entirety of the example module is loaded into an object called `example`. The named exports `sum()`, `multiple()`, and `magicNumber` are then accessible as properties on `example`.

## Renaming Exports and Imports

Sometimes the original name of a variable, function, or class isn't what you want to use. It's possible to change the name of an export both during the export and when the identifier is being imported.

In the first case, suppose you have a function that you'd like to export with a different name. You can use the `as` keyword to specify the name that the function should be known as outside of the module:

```js
function sum(num1, num2) {
    return num1 + num2;
}

export { sum as add };
```

Here, the `sum()` function is exported under the `add()`. That means when another module wants to import this function, it will have to use the name `add` instead:

```js
import { add } from "example";
```

If the module importing the function wants to use a different name, it can also use `as`:

```js
import { add as sum } from "example";
console.log(typeof add);            // "undefined"
console.log(sum(1, 2));             // 3
```

This code imports the `add()` function and renames it to `sum()`. That means there is no identifier named `add` in this module.
