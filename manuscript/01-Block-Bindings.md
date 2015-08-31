# Block Bindings

Traditionally, one of the tricky parts of JavaScript has been the way that `var` declarations work. In most C-based languages, variables (or *bindings*) are created at the spot where the declaration occurs. In JavaScript, however, this is not the case. Variables declared using `var` are *hoisted* to the top of the function (or global scope, if declared outside of a function) regardless of where the actual declaration occurs. For example:

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

I> Since `let` declarations are *not* hoisted to the top of the enclosing block, you may want to always place `let` declarations first in the block so that they are available to the entire block.

Perhaps one of the areas where developers most want block level scoping of variables is with `for` loops. It's not uncommon to see code such as this:

```js
for (var i=0; i < 10; i++) {
    process(items[i]);
}

// i is still accessible here
console.log(i);                     // 10
```

In other languages, where block level scoping is the default, code like this works as intended. In JavaScript, the variable `i` is still accessible after the loop is completed because the `var` declaration was hoisted. Using `let` allows you to get the intended behavior:

```js
for (let i=0; i < 10; i++) {
    process(items[i]);
}

// i is not accessible here - throws an error
console.log(i);
```

In this example, the variable `i` only exists within the `for` loop. Once the loop is complete, the variable is destroyed and is no longer accessible elsewhere.

## Constant declarations

Another new way to define variables is to use the `const` declaration syntax. Variables declared using `const` are considered to be *constants*, so the value cannot be changed once set. For this reason, every `const` variable must be initialized. For example:

```js
// Valid constant
const maxItems = 30;

// Syntax error: missing initialization
const name;
```

Constants, like `let` declarations, are block-level declarations. That means constants are destroyed once execution flows out of the block in which they were declared, and declarations are not hoisted to the top of the block. For example:

```js
if (condition) {
    const maxItems = 5;

    // more code
}

// maxItems isn't accessible here
```

In this code, the constant `maxItems` is declared within an `if` statement. Once the statement finishes executing, `maxItems` is destroyed and is not accessible outside of that block.

Also similar to `let`, an error is thrown whenever a `const` declaration is made with an identifier for an already-defined variable in the same scope. It doesn't matter if that variable was declared using `var` (for global or function scope) or `let` (for block scope). For example:

```js
var message = "Hello!";
let age = 25;

// Each of these would throw an error given the previous declarations
const message = "Goodbye!";
const age = 30;
```

The big difference between `let` and `const` is that attempting to assign to a previously defined constant will throw an error in both strict and non-strict modes:

```js
const maxItems = 5;

maxItems = 6;      // throws error
```

Keep in mind that `const` prevents modification of the binding and not of the value itself. That means `const` declarations for objects do not prevent modification of those objects. For example:

```js
const person = {
    name: "Nicholas";
};

// works
person.name = "Greg";

// throws an error
person = {
    name: "Greg"
};
```

Here, the binding `person` is created with an initial value of an object with one property. It's possible to change `person.name` without causing an error because you are changing what `person` contains and not changing the value that `person` is bound to. If you then attempt to assign a value to `person` (attempting to change the binding), an error is thrown. This subtlety in how `const` works with objects is easy to misunderstand. Remember, `const` prevents modification of the binding, not modification of the bound value.

W> Several browsers implement pre-ECMAScript 6 versions of `const`. Implementations range from being simply a synonym for `var` (allowing the value to be overwritten) to actually defining constants but only in the global or function scope. For this reason, be especially careful with using `const` in a production system. It may not be providing you with the functionality you expect.

## The Temporal Dead Zone

Unlike `var`, `let` and `const` has no hoisting characteristics. A variable declared with either cannot be accessed until after the declaration. Attempting to do so results in a reference error:

```js
if (condition) {
    console.log(value);     // ReferenceError!
    let value = "blue";
}
```

In this code, the variable `value` is defined and initialized using `let`, but that statement is never executed because the previous line throws an error. The issue is that `value` exists in what has become known as the *temporal dead zone* (TDZ). The TDZ is never named explicitly in the specification, but is a term often used to describe the non-hoisting behavior of `let`.

When a JavaScript engine looks through the upcoming block to find variable declarations, it results in either hoisting the declaration (for `var`) or placing the declaration in the TDZ (for `let` and `const`, discussed later). Any attempt to access a variable in the TDZ results in a runtime error. Only once execution flows to the variable declaration is that variable removed from the TDZ and therefore safe to use.

The same is true anytime you attempt to use a variable declared with `let` inside of the same block prior to it being defined. Even the normally safe `typeof` operator isn't safe:

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

I> Although the examples in this section use `let`, the same is true for `const`.

## Use in Loops

The characteristics of `var` have long made using variables inside of loops problematic. Consider the following:

```js
var funcs = [];

for (var i=0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // outputs the number "10" ten times
});
```

This code will output the number `10` ten times in a row. That's because the variable `i` is shared across each iteration of the loop, meaning the functions created inside the loop all hold a reference to the same variable. The variable `i` has a value of `10` once the loop completes, and so that's the value each function outputs.

To fix this problem, developers use immediately-invoked function expressions (IIFEs) inside of loops to force a new copy of the variable to be created:

```js
var funcs = [];

for (var i=0; i < 10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value);
        }
    }(i)));
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
});
```

This version of the example uses an IIFE inside of the loop. The `i` variable is passed to the IIFE, which creates its own copy and stores it as `value`. This is the value used of the function for that iteration, so calling each function returns the expected value.

### Let Declarations in Loops

A `let` declaration simplifies loops by effectively mimicking what the IIFE does in the previous example. Each iteration through the loop results in a new variable being created and initialized to the value of the variable with the same name from the previous iteration. That means you can omit the IIFE altogether and get the results you expect:

```js
var funcs = [];

for (let i=0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
})
```

This code works exactly like the code that used `var` and an IIFE but is, arguably, cleaner. The `let` declaration creates a new variable `i` each time through the loop, so each function created inside the loop gets its own copy of `i`. Each copy of `i` has the value it was assigned at the beginning of the loop iteration in which it was created. The same is true for `for-in` and `for-of` (discussed in Chapter 7) loops:

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

for (let key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

In this example, `for-in` loop shows the same behavior as the `for` loop. Each time through the loop, a new `key` binding is created and so each function has its own copy. The result is that each function outputs a different value (if `var` was used, they would all output `"c"`).

I> It's important to understand that the behavior of `let` declarations in loops is a specially-defined behavior in the specification and is not necessarily related to the non-hoisting characteristics of `let`. In fact, early implementions of `let` did not have this behavior as it was added later on in the process.

### Constant Declarations in Loops

The specification doesn't explicitly disallow `const` declarations in loops, however, there are different behaviors based on the type of loop you're using. For a normal `for` loop, you can use `const` in the initializer, but the loop will throw a warning if you attempt to change the value. For example:

```js
var funcs = [];

// throws an error after one iteration
for (const i=0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}
```

In this code, the `i` variable is declared as a constant. The first iteration of the loop, where `i` is 0, executes successfully. An error is thrown when `i++` executes because it's attempting to modify a constant. As such, you can only use `const` in the loop initializer if you are not modifying it.

When used in a `for-in` or `for-of` loop, `const` behaves the same as `let`. So the following will not cause an error:

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

// doesn't cause an error
for (const key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

This code functions almost exactly the same as when using `let`. The only difference is that the value of `key` cannot be changed inside of the loop. The `for-in` and `for-of` loops work with `const` because the initializer creates a new binding each time through the loop rather than attempting to modify the value of an existing binding (as was the case with the `for` loop example).

## Global Block Bindings

There is the potential for naming collisions when using `let` or `const` in the global scope because the global object has predefined properties. In many JavaScript environments, global variables are assigned as properties on the global object, and global object properties are accessed transparently as non-qualified identifiers (such as `name` or `location`). Using a block binding declaration to define a variable that shares a name with a property of the global object can produce an error because global object properties may be nonconfigurable. Since block bindings disallow redefinition of an identifier in the same scope, it's not possible to shadow nonconfigurable global properties. Attempting to do so will result in an error. For example:

```js
let RegExp = "Hello!";          // ok
let undefined = "Hello!";       // throws error
```

The first line of this example redefines the global `RegExp` as a string. Even though this would be problematic, there is no error thrown. The second line throws an error because `undefined` is a nonconfigurable own property of the global object. Since its definition is locked down by the environment, the `let` declaration is illegal.

It's unusual to use `let` or `const` in the global scope, but if you do, it's important to understand this situation.

## Emerging Best Practices

While ECMAScript 6 was in development, there was widespread belief you should use `let` by default instead of `var` for variable declarations. For many, `let` behaves exactly the way they thought `var` should have behaved, and so the direct replacement makes logical sense. You would then use `const` for variables that needed modification protection.

However, as more developers migrated to ECMAScript 6, an alternate approach gained popularity: use `const` by default and only use `let` when you know a variable's value needs to change. The rationale is that most variables should not change their value after initialization because unexpected value changes are a source of bugs. This idea has gained a significant amount of traction and is worth exploring in your code as you adopt ECMAScript 6.

## Summary

The `let` and `const` block bindings introduce lexical scoping to JavaScript. These declarations are not hoisted and only exist within the block in which they are declared. That means behavior that is more like other languages and less likely to cause unintentional errors, as variables can now be declared exactly where they are needed. As a side effect, you cannot access variables before they are declared, even with safe operators such as `typeof`. Attempting to access a block binding before its declaration results in an error due to the binding's presence in the temporal dead zone (TDZ).

In many cases, `let` and `const` behave in a manner that is similar to `var`, however, this is not true for loops. For both `let` and `const`, `for-in` and `for-of` loops create a new binding with each iteration through the loop. That means functions created inside the loop body can access the loop bindings values as they were during that iteration rather than as they were after the last iteration (the behavior with `var`). The same is true for `let` declarations in `for` loops, while attempting to use `const` declarations in a `for` loop may result in an error.

The current best practice for block bindings is to use `const` by default and only use `let` when you know a variable's value needs to change. This ensures a basic level of immutability in code that can help prevent certain types of errors.
