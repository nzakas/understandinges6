# Modules

One of the most error-prone and confusing aspects of JavaScript has long been the "shared everything" approach to loading code. Whereas other languages have concepts such as packages, JavaScript lagged behind, and everything defined in every file shared the same global scope. As web applications became more complex and the amount of JavaScript used grew, the "shared everything" approach began to show problems with naming collisions, security concerns, and more. One of the goals of ECMAScript 6 was to solve this problem and bring some order into JavaScript applications. That's where modules come in.

## What are Modules?

*Modules* are JavaScript files that are loaded in a special mode (as opposed to *scripts*, which are loaded in the original way JavaScript worked). At the time of my writing, neither browsers nor Node.js have a way to natively load ECMAScript 6 modules, but both have indicated that there will need to be some sort of opt-in to do so. The reason this opt-in is necessary is because module files have very different semantics than non-module files:

1. Module code automatically runs in strict mode and there's no way to opt-out of strict mode.
1. Variables created in the top level of a module are not automatically added to the shared global scope. They exist only within the top-level scope of the module.
1. The value of `this` in the top level of a module is `undefined`.
1. Modules do not allow HTML-style comments within the code (a leftover feature from the early browser days).
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
1. Both function and class declarations require a name. You cannot export anonymous functions or classes using this syntax (unless using the `default` keyword discussed later in this chapter)
1. You need not always export the declaration, you can also export references, as with `multiply` in this example.
1. Any variables, functions, or classes that are not explicitly exported remain private to the module. In this example, `subtract()` is not exported and is therefore not accessible from outside the module.

An important limitation of `export` is that it must be used in the top-level of the module. For instance, this is a syntax error:

```js
if (flag) {
    export flag;    // syntax error
}
```

This example is a syntax error because `export` is inside of an `if` statement. Exports cannot be conditional or done dynamically in any way. Part of the benefit of module syntax is so the JavaScript engine can staticly determine what will be exported. As such, you can only use `export` at the top-level of a module.

W> If you are using a transpiler like Babel.js, you may find that `export` can be used anywhere. This only works when code is converted to ECMAScript 5 and will not work with a native ECMAScript 6 module system.

Once you have a module with exports, you can access the functionality in another module by using the `import` keyword. An `import` statement has two parts: the identifiers you're importing and the module from which those identifiers should be imported. The basic form is as follows:

```js
import { identifier1, identifier2 } from "module";
```

The curly braces after `import` indicate the identifiers to import from the given module. The keyword `from` is used to indicate the module from which to import the given identifiers. The module is specified using a string. At the time of my writing, it is still undecided what module identifiers will look like. They may end up being full file paths (such as "../mymodule.js"), file paths without extensions (such as "../mymodule"), or something else. This likely won't be determined until browsers and Node.js begin implementing modules natively.

I> Even though it looks similar, the list of identifiers to import is not a destructured object.

When importing an identifier from a module, the identifier acts as if it were defined using `const`. That means you cannot define another variable with the same name, use the identifier prior to the `import` statement, or change its value.

Suppose that the first example in this section is in a module named `"example"`. You can import and use identifiers from that module in a number of ways. You can just import one identifier:

```js
// import just one
import { sum } from "example";

console.log(sum(1, 2));     // 3

sum = 1;        // error
```

This example imports only `sum()` from the example module. Even though the example module exports more than just that one function, they are not exposed here. If you try to assign a new value to `sum`, the result is an error, as you cannot reassign imported identifiers.

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

Keep in mind that the code inside of a module will only ever be executed once, regardless of the number of times it's used in an `import` statement. Consider the following:

```js
import { sum } from "example";
import { multiply } from "example";
import { magicNumber } from "example";
```

Even though there are three `import` statements in this module, the code in `"example"` will only be executed once. The instantiated module is then kept in memory and reused whenever another `import` statement references it. It doesn't matter if the `import` statements are all in the module, or are spread across multiple modules - they each will use the same module instance.

## Renaming Exports and Imports

Sometimes the original name of a variable, function, or class isn't what you want to use. It's possible to change the name of an export both during the export and when the identifier is being imported.

In the first case, suppose you have a function that you'd like to export with a different name. You can use the `as` keyword to specify the name that the function should be known as outside of the module:

```js
function sum(num1, num2) {
    return num1 + num2;
}

export { sum as add };
```

Here, the `sum()` function (`sum` is the *local name*) is exported as `add()` (`add` is the *exported name*). That means when another module wants to import this function, it will have to use the name `add` instead:

```js
import { add } from "example";
```

If the module importing the function wants to use a different name, it can also use `as`:

```js
import { add as sum } from "example";
console.log(typeof add);            // "undefined"
console.log(sum(1, 2));             // 3
```

This code imports the `add()` function (the *import name*) and renames it to `sum()` (the local name). That means there is no identifier named `add` in this module.

A> ### Imported Bindings
A>
A> A subtle but important point about the `import` statements is that they create bindings to variables, functions, and classes rather than simply referencing them. That means even though you cannot change an imported identifier, it can still change on its own. For example, suppose you have this module:
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
A> import { name, setName } from "example";
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
import sum from "example";

console.log(sum(1, 2));     // 3
```

This import statement imports the default from the module `"example"`. Note that there are no curly braces used in this case, as would be with a non-default export. The local name `sum` is used to represent the function that the module exports. This syntax is the cleanest as it's anticipated to be the dominant form of import on the web, allowing you to use already-existing object, such as:

```js
import $ from "jquery";
```

For modules that export both a default and one or more non-defaults, you can import them with one statement. For instance, suppose you have this module:

```js
export let color = "red";

export default function(num1, num2) {
    return num1 + num2;
}
```

You can then import both `color` and the default function using the following:

```js
import sum, { color } from "example";

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

In this code, the default export (`default`) is renamed to `sum` and the additional `color` export is also imported. This example is equivalent to the previous example.

## Re-exporting

There may be a time when you'd like to re-export something that your module has imported. You can do this using the patterns already discussed in this chapter, such as:

```js
import { sum } from "example";
export { sum }
```

However, there's also a single statement that can accomplish the same thing:

```js
export { sum } from "example";
```

This form of `export` looks into the specified module for the declaration of `sum` and then exports it. Of course, you can also choose to export a different name for the same thing:

```js
export { sum as add } from "example";
```

Here, `sum` is imported from `"example"` and then exported as `add`.

If you'd like to export everything from another module, you can use the `*` pattern:

```js
export * from "example";
```

By exporting everything, you're including the default as well as any named exports, which may affect what you can export from your module. For instance, if `"example"` has a default export, you'll be unable to define a new default export when using this syntax.

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

## Summary

ECMAScript 6 adds modules to the language as a way to package up and encapsulate functionality. Modules behave differently than scripts, as they do not modify the global scope with their top-level variables, functions, and classes, and `this` is `undefined`. In order to work differently than scripts, modules must be loaded using a different mode.

You must export any functionality you'd like to make available to consumers of a module. Variables, functions, and classes can all be exported, and there is also one default export allowed per module. After exporting, another module can import all or some of the exported names. These names act as if defined by `let`, and so operate as block bindings that cannot be redeclared in the same module.

Modules need not export anything if they are manipulating something in the global scope. In that case, it's possible to import from such a module without introducing any bindings into the module scope.
