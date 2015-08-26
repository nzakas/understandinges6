# Block Bindings

Traditionally, one of the tricky parts of JavaScript has been the way that `var` declarations work. In most C-based languages, variables are created at the spot where the declaration occurs. In JavaScript, however, this is not the case. Variables declared using `var` are *hoisted* to the top of the function (or global scope) regardless of where the actual declaration occurs. For example:

```js
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // other code

        return value;
    } else {

        // value exists here with a value of undefined

        return null;
    }

    // value exists here with a value of undefined
}
```

If you are unfamiliar with JavaScript, you might expect that the variable `value` is only defined if `condition` evaluates to true. In fact, the variable `value` is declared regardless. The JavaScript engine changes the function to look like this:

```js
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

The declaration of `value` is moved to the top (hoisted) while the initialization remains in the same spot. That means the variable `value` is actually still accessible from within the `else` clause, it just has a value of `undefined` because it hasn't been initialized.

It often takes new JavaScript developers some time to get used to declaration hoisting and this unique behavior can end up causing bugs. For this reason, ECMAScript 6 introduces block level scoping options to make the control of variable lifecycle a little more powerful.

## Let declarations

The `let` declaration syntax is the same as for `var`. You can basically replace `var` with `let` to declare a variable but keep its scope to the current code block. For example:

```js
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // other code

        return value;
    } else {

        // value doesn't exist here

        return null;
    }

    // value doesn't exist here
}
```

This function now behaves much closer to other C-based languages. The variable `value` is declared using `let` instead of `var`. That means the declaration is not hoisted to the top, and the variable `value` is destroyed once execution has flowed out of the `if` block. If `condition` evaluates to false, then `value` is never declared or initialized.

Perhaps one of the areas where developers most want block level scoping of variables is with `for` loops. It's not uncommon to see code such as this:

```js
for (var i=0; i < items.length; i++) {
    process(items[i]);
}

// i is still accessible here and is equal to items.length
```

In other languages, where block level scoping is the default, code like this works as intended. In JavaScript, the variable `i` is still accessible after the loop is completed because the `var` declaration was hoisted. Using `let` allows you to get the intended behavior:

```js
for (let i=0; i < items.length; i++) {
    process(items[i]);
}

// i is not accessible here
```

In this example, the variable `i` only exists within the `for` loop. Once the loop is complete, the variable is destroyed and is no longer accessible elsewhere.

A> ### Using let in loops
A>
A> The behavior of `let` inside of loops is slightly different than with other blocks. Instead of creating a variable that is used with each iteration of the loop, each iteration actually gets its own variable to use. This is to solve an old problem with JavaScript loops. Consider the following:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  var funcs = [];
A>
A>  for (var i=0; i < 10; i++) {
A>      funcs.push(function() { console.log(i); });
A>  }
A>
A>  funcs.forEach(function(func) {
A>      func();     // outputs the number "10" ten times
A>  });
A> ~~~~~~~~
A>
A> This code will output the number `10` ten times in a row. That's because the variable `i` is shared across each iteration of the loop, meaning the closures created inside the loop all hold a reference to the same variable. The variable `i` has a value of `10` once the loop completes, and so that's the value each function outputs.
A>
A> To fix this problem, developers use immediately-invoked function expressions (IIFEs) inside of loops to force a new copy of the variable to be created:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  var funcs = [];
A>
A>  for (var i=0; i < 10; i++) {
A>      funcs.push((function(value) {
A>          return function() {
A>              console.log(value);
A>          }
A>      }(i)));
A>  }
A>
A>  funcs.forEach(function(func) {
A>      func();     // outputs 0, then 1, then 2, up to 9
A>  });
A> ~~~~~~~~
A>
A> This version of the example uses an IIFE inside of the loop. The `i` variable is passed to the IIFE, which creates its own copy and stores it as `value`. This is the value used of the function for that iteration, so calling each function returns the expected value.
A>
A> A `let` declaration does this for you without the IIFE. Each iteration through the loop results in a new variable being created and initialized to the value of the variable with the same name from the previous iteration. That means you can simplify the process by using this code:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  var funcs = [];
A>
A>  for (let i=0; i < 10; i++) {
A>      funcs.push(function() { console.log(i); });
A>  }
A>
A>  funcs.forEach(function(func) {
A>      func();     // outputs 0, then 1, then 2, up to 9
A>  })
A> ~~~~~~~~
A>
A>  This code works exactly like the code that used `var` and an IIFE but is, arguably, cleaner.

Unlike `var`, `let` has no hoisting characteristics. A variable declared with `let` cannot be accessed until after the `let` statement. Attempting to do so results in a reference error:

```js
if (condition) {
    console.log(value);     // ReferenceError!
    let value = "blue";
}
```

In this code, the variable `value` is defined and initialized using `let`, but that statement is never executed because the previous line throws an error. The issue is that `value` exists in what has become known as the *temporal dead zone* (TDZ). The TDZ is never named explicitly in the specification, but is a term often used to describe the non-hoisting behavior of `let`.

When a JavaScript parser looks through the upcoming block to find variable declarations, it results in either hoisting the declaration (for `var`) or placing the declaration in the TDZ. Any attempt to access a variable in the TDZ results in a runtime error. Only once execution flows to the variable declaration is that variable removed from the TDZ and therefore safe to use.

The same is true anytime you attempt to use a variable declared with `let` inside of the same block prior to it being defined. Even the normally safe-to-use `typeof` operator isn't safe:

```js
if (condition) {
    console.log(typeof value);     // ReferenceError!
    let value = "blue";
}
```

Here, `typeof value` throws the same error as the previous example. You cannot use a `let` variable before its declaration within the same block. However, you can use `typeof` outside of the block:

```js
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

The variable `value` isn't in the TDZ when the `typeof` operation executes because it occurs outside of the block in which `value` is declared. That means there is no `value` binding and `typeof` simply returns `"undefined"`.

If an identifier has already been defined in the block, then using the identifier in a `let` declaration causes an error to be thrown. For example:

```js
var count = 30;

// Syntax error
let count = 40;
```

In this example, `count` is declared twice, once with `var` and once with `let`. Because `let` will not redefine an identifier that already exists in the same scope, the declaration throws an error. No error is thrown if a `let` declaration creates a new variable in a scope with the same name as a variable in the containing scope, such as:

```js
var count = 30;

// Does not throw an error
if (condition) {

    let count = 40;

    // more code
}
```

Here, the `let` declaration will not throw an error because it is creating a new variable called `count` within the `if` statement. This new variable shadows the global `count`, preventing access to it from within the `if` block.

A> ## Global let Declarations
A>
A> There is the potential for naming collisions when using `let` in the global scope because the global object has predefined properties. Using `let` to define a variable that shares a name with a property of the global object can produce an error because global object properties may be nonconfigurable. Since `let` doesn't allow redefinition of the same identifier in the same scope, it's not possible to shadow nonconfigurable global properties. Attempting to do so will result in an error. For example:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  let RegExp = "Hello!";          // ok
A>  let undefined = "Hello!";       // throws error
A> ~~~~~~~~
A>
A> The first line of this example redefines the global `RegExp` as a string. Even though this would be problematic, there is no error thrown. The second line throws an error because `undefined` is a nonconfigurable own property of the global object. Since its definition is locked down by the environment, the `let` declaration is illegal.
A>
A> It's unusual to use `let` in the global scope, but if you do, it's important to understand this situation.

The long term intent is for `let` to replace `var`, as the former behaves more like variable declarations in other languages. If you are writing JavaScript that will execute only in an ECMAScript 6 or higher environment, you may want to try using `let` exclusively and leaving `var` for other scripts that require backwards compatibility.

I> Since `let` declarations are *not* hoisted to the top of the enclosing block, you may want to always place `let` declarations first in the block so that they are available to the entire block.

## Constant declarations

Another new way to define variables is to use the `const` declaration syntax. Variables declared using `const` are considered to be *constants*, so the value cannot be changed once set. For this reason, every `const` variable must be initialized. For example:

```js
// Valid constant
const MAX_ITEMS = 30;

// Syntax error: missing initialization
const NAME;
```

Constants are also block-level declarations, similar to `let`. That means constants are destroyed once execution flows out of the block in which they were declared, and declarations are not hoisted to the top of the block. For example:

```js
if (condition) {
    const MAX_ITEMS = 5;

    // more code
}

// MAX_ITEMS isn't accessible here
```

In this code, the constant `MAX_ITEMS` is declared within an `if` statement. Once the statement finishes executing, `MAX_ITEMS` is destroyed and is not accessible outside of that block.

Also similar to `let`, an error is thrown whenever a `const` declaration is made with an identifier for an already-defined variable in the same scope. It doesn't matter if that variable was declared using `var` (for global or function scope) or `let` (for block scope). For example:

```js
var message = "Hello!";
let age = 25;

// Each of these would cause an error given the previous declarations
const message = "Goodbye!";
const age = 30;
```

The big difference between `let` and `const` is that attempting to assign to a previously defined constant will throw an error in both strict and non-strict modes:

```js
const MAX_ITEMS = 5;

MAX_ITEMS = 6;      // throws error
```

W> Several browsers implement pre-ECMAScript 6 versions of `const`. Implementations range from being simply a synonym for `var` (allowing the value to be overwritten) to actually defining constants but only in the global or function scope. For this reason, be especially careful with using `const` in a production system. It may not be providing you with the functionality you expect.

## Summary

The `let` and `const` block bindings introduce lexical scoping to JavaScript. These declarations are not hoisted and only exist within the block in which they are declared. That means behavior that is more like other languages and less likely to cause unintentional errors, as variables can now be declared exactly where they are needed. It's expected that the majority of JavaScript code going forward will use `let` and `const` exclusively, effectively making `var` a deprecated part of the language.
