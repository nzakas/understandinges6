# Functions

Functions are an important part of any programming language, and JavaScript functions hadn't changed much since the language was first introduced. This left a backlog of problems and nuanced behavior that made it easy to make mistakes or require more code just to achieve a very common behavior.

ECMAScript 6 functions made a big leap forward, taking into account years of complaints and asks from JavaScript developers. The result is a number of incremental improvements on top of ECMAScript 5 functions that make programming in JavaScript less error-prone and more powerful than ever before.

## Default Parameters

Functions in JavaScript are unique in that they allow any number of parameters to be passed regardless of the number of declared parameters in the function definition. This allows you to define functions that can handle different number of parameters, often by just filling in default values when ones aren't provided. In ECMAScript 5 and earlier, you would likely use the following pattern to accomplish this:

```js
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // the rest of the function

}
```

In this example, both `timeout` and `callback` are actually optional because they are given a default value if not provided. The logical OR operator (`||`) always returns the second operand when the first is falsy. Since named function parameters that are not explicitly provided are set to `undefined`, the logical OR operator is frequently used to provide default values for missing parameters. There is a flaw with this approach, however, in that a valid value for `timeout` might actually be `0`, but this would replace it with `2000` because `0` is falsy.

Other ways of determining if any parameters are missing include checking `arguments.length` for the number of parameters that were passed or directly inspecting each parameter to see if it is not `undefined`.

ECMAScript 6 makes it easier to provide default values for parameters by providing initializations that are used when the parameter isn't formally passed. For example:

```js
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // the rest of the function

}
```

Here, only the first parameter is expected to be passed all the time. The other two parameters have default values, which makes the body of the function much smaller because you don't need to add any code to check for a missing value. When `makeRequest()` is called with all three parameters, then the defaults are not used. For example:

```js
// uses default timeout and callback
makeRequest("/foo");

// uses default callback
makeRequest("/foo", 500);

// doesn't use defaults
makeRequest("/foo", 500, function(body) {
    doSomething(body);
});
```

Any parameters with a default value are considered to be optional parameters while those without a default value are considered to be required parameters.

It's possible to specify default values for any arguments, including those that appear before arguments without default values. For example, this is fine:

```js
function makeRequest(url, timeout = 2000, callback) {

    // the rest of the function

}
```

In this case, the default value for `timeout` will only be used if there is no second argument passed in or if the second argument is explicitly passed in as `undefined`. For example:

```js
// uses default timeout
makeRequest("/foo", undefined, function(body) {
    doSomething(body);
});

// uses default timeout
makeRequest("/foo");

// doesn't use default timeout
makeRequest("/foo", null, function(body) {
    doSomething(body);
});
```

In the case of default parameter values, the value of `null` is considered to be valid and the default value will not be used.

Perhaps the most interesting feature of default parameter values is that the default value need not be a primitive value. You can, for example, execute a function to retrieve the default parameter:

```js
function getCallback() {
    return function() {
        // some code
    };
}

function makeRequest(url, timeout = 2000, callback = getCallback()) {

    // the rest of the function

}
```

Here, if the last argument isn't provided, the function `getCallback()` is called to retrieve the correct default value. This opens up a lot of interesting possibilities to dynamically inject information into functions.

## Rest Parameters

Since JavaScript functions can be passed any number of parameters, it's not always necessary to define each parameter specifically. Early on, JavaScript provided the `arguments` object as a way of inspecting all function parameters that were passed without necessarily defining each one individually. While that worked fine in most cases, it can become a little cumbersome to work with. For example:

```js
function pick(object) {
    let result = Object.create(null);

    for (let i = 1, len = arguments.length; i < len; i++) {
        result[arguments[i]] = object[arguments[i]];
    }

    return result;
}

let book = {
    title: "Understanding ECMAScript 6",
    author: "Nicholas C. Zakas",
    year: 2015
};

let bookData = pick(book, "author", "year");

console.log(bookData.author);   // "Nicholas C. Zakas"
console.log(bookData.year);     // 2015
```

This function mimics the `pick()` method from Underscore. The first argument is the object from which to copy properties and every other argument is the name of a property that should be copied on the result. There are couple of things to notice about this function. First, it's not at all obvious that the function is capable of handling more than one parameter. You could add in several more named parameters, but you would always fall short of indicating that this function can take any number of parameters. Second, because the first parameter is named and used directly, you have to start looking in the `arguments` object at index 1 instead of starting at index 0. Remembering to use the appropriate indices with `arguments` isn't necessarily difficult, but it's one more thing to keep track of. ECMAScript 6 introduces rest parameters to help with these issues.

Rest parameters are indicated by three dots (`...`) preceding a named parameter. That named parameter then becomes an `Array` containing the rest of the parameters (which is why these are called "rest" parameters). For example, `pick()` can be rewritten using rest parameters like this:

```js
function pick(object, ...keys) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

In this version of the function, `keys` is a rest parameter that contains all parameters after the first one (unlike `arguments`, which contains all parameters including the first one). That means you can iterate over `keys` from beginning to end without worry. As a bonus, you can tell by looking at the function that it is capable of handling any number of parameters.

The only restriction on rest parameters is that no other named arguments can follow in the function declaration. For example, this causes syntax error:

```js
// Syntax error: Can't have a named parameter after rest parameters
function pick(object, ...keys, last) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

Here, the parameter `last` follows the rest parameter `keys` and causes a syntax error.

Rest parameters were designed to replace `arguments` in ECMAScript. Originally ECMAScript 4 did away with `arguments` and added rest parameters to allow for an unlimited number of arguments to be passed to functions. Even though ECMAScript 4 never came into being, the idea was kept around and reintroduced in ECMAScript 6 despite `arguments` not being removed from the language.

## Destructured Parameters

In Chapter 1, you learned about destructuring assignment. Destructuring can also be used outside of the context of an assignment expression and perhaps the most interesting such case is with destructured parameters.

It's common for functions that take a large number of optional parameters to use an options object as one or more parameters. For example:

```js
function setCookie(name, value, options) {

    options = options || {};

    var secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;

    // ...
}

setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

There are many `setCookie()` functions in JavaScript libraries that look similar to this. The `name` and `value` are required but everything else is not. And since there is no priority order for the other data, it makes sense to have an options object with named properties rather than extra named parameters. This approach is okay, although it makes the expected input for the function a bit opaque.

Using destructured parameters, the previous function can be rewritten as follows:

```js
function setCookie(name, value, { secure, path, domain, expires }) {

    // ...
}

setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

The behavior of this function is similar to the previous example, the biggest difference is the third argument uses destructuring to pull out the necessary data. Doing so makes it clear which parameters are really expected, and the destructured parameters also act like regular parameters in that they are set to `undefined` if they are not passed.

One quirk of this pattern is that the destructured parameters throw an error when the argument isn't provided. If `setCookie()` is called with just two arguments, it results in a runtime error:

```js
// Error!
setCookie("type", "js");
```

This code throws an error because the third argument is missing (`undefined`). To understand why this is an error, it helps to understand that destructured parameters are really just a shorthand for destructured assignment. The JavaScript engine is actually doing this:

```js
function setCookie(name, value, options) {

    var { secure, path, domain, expires } = options;

    // ...
}
```

Since destructuring assignment throws an error when the right side expression evaluates to `null` or `undefined`, the same is true when the third argument isn't passed.

You can work around this behavior by providing a default value for the destructured parameter:

```js
function setCookie(name, value, { secure, path, domain, expires } = {}) {

    // ...
}
```

This example now works exactly the same as the first example in this section. Providing the default value for the destructured parameter means that `secure`, `path`, `domain`, and `expires` will all be `undefined` if the third argument to `setCookie()` isn't provided.

I> It's recommended to always provide the default value for destructured parameters to avoid these types of errors.

## The Spread Operator

Closely related to rest parameters is the spread operator. Whereas rest parameters allow you to specify that multiple independent arguments should be combined into an array, the spread operator allows you to specify an array that should be split and have its items passed in as separate arguments to a function. Consider the `Math.max()` method, which accepts any number of arguments and returns the one with the highest value. It's basic usage is as follows:

```js
let value1 = 25,
    value2 = 50;

console.log(Math.max(value1, value2));      // 50
```

When you're dealing with just two values, as in this example, `Math.max()` is very easy to use. The two values are passed in and the higher value is returned. But what if you have been tracking values in an array, and now you want to find the highest value? The `Math.max()` method doesn't allow you to pass in an array, so in ECMAScript 5 and earlier, you'd be stuck either searching the array yourself or using `apply()`:

```js
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100
```

While possible, using `apply()` in this manner is a bit confusing - it actually seems to obfuscate the true meaning of the code with additional syntax.

The ECMAScript 6 spread operator makes this case very simple. Instead of calling `apply()`, you can pass in the array and prefix it with the same `...` pattern that is used with rest parameters. The JavaScript engine then splits up the array into individual arguments and passes them in:

```js
let values = [25, 50, 75, 100]

// equivalent to
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100
```

Now the call to `Math.max()` looks a bit more conventional and avoids the complexity of specifying a `this`-binding for a simple mathematical operation.

You can mix and match the spread operator with other arguments as well. Suppose you want the smallest number returned from `Math.max()` to be 0 (just in case negative numbers sneak into the array). You can pass that argument separately and still use the spread operator for the other arguments:

```js
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0
```

In this example, the last argument passed to `Math.max()` is `0`, which comes after the other arguments are passed in using the spread operator.

The spread operator for argument passing makes using arrays for function arguments much easier. You'll likely find it to be a suitable replacement for the `apply()` method in most circumstances.

## The name Property

Identifying functions can be challenging in JavaScript given the various ways a function can be defined. Additionally, the prevalence of anonymous function expressions makes debugging a bit more difficult, often resulting in stack traces that are hard to read and decipher. For these reasons, ECMAScript 6 adds the `name` property to all functions.

All functions in an ECMAScript 6 program will have an appropriate value for their `name` property while all others will have an empty string. For example:

```js
function doSomething() {
    // ...
}

var doAnotherThing = function() {
    // ...
};

console.log(doSomething.name);          // "doSomething"
console.log(doAnotherThing.name);       // "doAnotherThing"
```

In this code, `doSomething()` has a `name` property equal to `"doSomething"` because it's a function declaration. The anonymous function expression `doAnotherThing()` has a `name` of `"doAnotherThing"` due to the variable to which it is assigned.

While appropriate names for function declarations and function expressions are easy to find, ECMAScript 6 goes further to ensure that all functions have appropriate names:

```js
var doSomething = function doSomethingElse() {
    // ...
};

var person = {
    get firstName() {
        return "Nicholas"
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);      // "doSomethingElse"
console.log(person.sayName.name);   // "sayName"
console.log(person.firstName.name); // "get firstName"
```

In this example, `doSomething.name` is `"doSomethingElse"` because the function expression itself has a name and that name takes priority over the variable to which the function was assigned. The `name` property of `person.sayName()` is `"sayName"`, as the value was interpreted from the object literal. Similarly, `person.firstName` is actually a getter function, so its name is `"get firstName"` to indicate this difference (setter functions are prefixed with `"set"` as well).

There are a couple of other special cases for function names. Functions created using `bind()` will have their name prefixed with `"bound"` and functions created using the `Function` constructor have a name of `"anonymous"`:

```js
var doSomething = function() {
    // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous"
```

The `name` of a bound function will always be the `name` of the function being bound prefixed with the `"bound "`, so the bound version of `doSomething()` is `"bound doSomething"`.

## new.target, [[Call]], and [[Construct]]

In ECMAScript 5 and earlier, functions serve the double purpose of being callable with or without `new`. When used with `new`, the `this` value inside of a function is a new object and that new object is returned. For example:

```js
function Person(name) {
    this.name = name;
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");

console.log(person);        // "[Object object]"
console.log(notAPerson);    // "undefined"
```

Calling `Person()` without `new` results in `undefined` (and a `name` property being set on the global object in non-strict mode). It's fairly obvious from the code that the intent is to use `Person` with `new` to create a new object. The confusion over the dual roles of functions led to some changes in ECMAScript 6.

First, the specification defines two different internal-only methods that every function has: `[[Call]]` and `[[Construct]]`. When a function is called without `new`, the `[[Call]]` method is executed, which essentially executes the body of the function as it appears in the code. When a function is called with `new`, that's when the `[[Construct]]` method is called. The `[[Construct]]` method is responsible for creating a new object, called the *new target*, and then executing the function body with `this` set to the new target. Functions that have a `[[Construct]]` method are called *constructors*.

I> Keep in mind that not all functions have `[[Construct]]`, and therefore not all function can be called with `new`. Arrow functions, discussed later in this chapter, do not have a `[[Construct]]` method.

The most popular way to determine if a function was called with `new` in ECMAScript 5 is to use `instanceof`, for example:

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");  // throws error
```

Here, the `this` value is checked to see if it's an instance of the constructor, and if so, it continues as normal. If `this` isn't an instance of `Person`, then an error is thrown. This works because the `[[Construct]]` method creates a new instance of `Person` and assigns it to `this`. Unfortunately, this approach is not completely reliable because `this` can be an instance of `Person` without using `new`, for example:

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // works!
```

The call to `Person.call()` passes the `person` variable as the first argument, which means `this` is set to `person` inside of the `Person` function. To the function, there's no way to distinguish this from being called with `new`.

To solve this problem, ECMAScript 6 introduces the `new.target` *metaproperty*. When a function's `[[Construct]]` method is called, `new.target` is filled with the target of the `new` operator, which is typically the constructor of the newly created object instance that will become `this` inside the function body. If `[[Call]]` is executed, then `new.target` is `undefined`. That means you can now safely detect if a function is called with `new` by checking that `new.target` is defined:

```js
function Person(name) {
    if (typeof new.target !== "undefined") {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // error!
```

By using `new.target` instead of `this instanceof Person`, the `Person` constructor is now correctly throwing an error when used without `new`.

You can also check that `new.target` was called with a specific constructor, for instance:

```js
function Person(name) {
    if (typeof new.target === Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

function AnotherPerson(name) {
    Person.call(this, name);
}

var person = new Person("Nicholas");
var anotherPerson = new AnotherPerson("Nicholas");  // error!
```

In this example, `new.target` must be `Person` in order to work correctly. When `new AnotherPerson("Nicholas")` is called, `new.target` is set to `AnotherPerson`, so the subsequent call to `Person.call(this, name)` will throw an error even though `new.target` is defined.

W> Using `new.target` outside of a function is a syntax error.

## Block-Level Functions

In ECMAScript 3 and earlier, a function declaration occurring inside of a block (a *block-level function*) was technically a syntax error, but many browsers still supported it. Unfortunately, each browser that allowed the syntax behaved in a slightly different way, so it is considered a best practice to avoid function declarations inside of blocks (the best alternative is to use a function expression).

In an attempt to reign in this incompatible behavior, ECMAScript 5 strict mode introduced an error whenever a function declaration was used inside of a block. For example:

```js
"use strict";

if (true) {

    // Throws a syntax error in ES5, not so in ES6
    function doSomething() {
        // ...
    }
}
```

In ECMAScript 5, this code throws a syntax error. In ECMAScript 6, the `doSomething()` function is considered a block-level declaration and can be accessed and called within the same block in which it was defined. For example:

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "undefined"
```

Block level functions are hoisted to the top of the block in which they are defined, so `typeof doSomething` returns `"function"` even though it appears before the function declaration in the code. Once the `if` block is finished executing, `doSomething()` no longer exists.

Block level functions are a similar to `let` function expressions in that the function definition is removed once execution flows out of the block in which it's defined. The key difference is that block level functions are hoisted to the top of the containing block while `let` function expressions are not hoisted. For example:

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // throws error

    let doSomething = function () {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);
```

Here, code execution stops when `typeof doSomething` is executed because the `let` statement hasn't been executed yet.

Whether you want to use block level functions or `let` expressions depends on whether or not you want the hoisting behavior.

ECMAScript 6 also allows block-level functions in nonstrict mode, but the behavior is slightly different. Instead of hoisting these declarations to the top of the block, they are hoisted all the way to the containing function or global environment. For example:

```js
// ECMAScript 6 behavior
if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "function"
```

In this example, `doSomething()` is hoisted into the global scope so that it still exists outside of the `if` block. ECMAScript 6 standardized this behavior to remove the incompatible browser behaviors that previously existed. ECMAScript 6 runtimes will all behave in the same way.

## Arrow Functions

One of the most interesting new parts of ECMAScript 6 are arrow functions. Arrow functions are, as the name suggests, functions defined with a new syntax that uses an "arrow" (`=>`). However, arrow functions behave differently than traditional JavaScript functions in a number of important ways:

* **Lexical `this` binding** - The value of `this` inside of the function is determined by where the arrow function is defined not where it is used.
* **Not `new`able** - Arrow functions do not have a `[[Construct]]` method and therefore cannot be used as constructors. Arrow functions throw an error when used with `new`.
* **Can't change `this`** - The value of `this` inside of the function can't be changed, it remains the same value throughout the entire lifecycle of the function.
* **No `arguments` object** - You can't access arguments through the `arguments` object, you must use named arguments or other ES6 features such as rest arguments.

There are a few reasons why these differences exist. First and foremost, `this` binding is a common source of error in JavaScript. It's very easy to lose track of the `this` value inside of a function, which can result in unintended consequences. Second, by limiting arrow functions to simply executing code with a single `this` value, JavaScript engines can more easily optimize these operations (as opposed to regular functions, which might be used as a constructor or otherwise modified).

I> Arrow functions also have a `name` property that follows the same rule as other functions.

## Syntax

The syntax for arrow functions comes in many flavors depending upon what you are trying to accomplish. All variations begin with function arguments, followed by the arrow, followed by the body of the function. Both the arguments and the body can take different forms depending on usage. For example, the following arrow function takes a single argument and simply returns it:

```js
var reflect = value => value;

// effectively equivalent to:

var reflect = function(value) {
    return value;
};
```

When there is only one argument for an arrow function, that one argument can be used directly without any further syntax. The arrow comes next and the expression to the right of the arrow is evaluated and returned. Even though there is no explicit `return` statement, this arrow function will return the first argument that is passed in.

If you are passing in more than one argument, then you must include parentheses around those arguments. For example:

```js
var sum = (num1, num2) => num1 + num2;

// effectively equivalent to:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

The `sum()` function simply adds two arguments together and returns the result. The only difference is that the arguments are enclosed in parentheses with a comma separating them (same as traditional functions).

If there are no arguments to the function, then you must include an empty set of parentheses:

```js
var getName = () => "Nicholas";

// effectively equivalent to:

var getName = function() {
    return "Nicholas";
};
```

When you want to provide a more traditional function body, perhaps consisting of more than one expression, then you need to wrap the function body in braces and explicitly define a return value, such as:

```js
var sum = (num1, num2) => {
    return num1 + num2;
};

// effectively equivalent to:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

You can more or less treat the inside of the curly braces as the same as in a traditional function with the exception that `arguments` is not available.

If you want to create a function that does nothing, then you need to include curly braces:

```js
var doNothing = () => {};

// effectively equivalent to:

var doNothing = function() {};
```

Because curly braces are used to denote the function's body, an arrow function that wants to return an object literal outside of a function body must wrap the literal in parentheses. For example:

```js
var getTempItem = id => ({ id: id, name: "Temp" });

// effectively equivalent to:

var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
};
```

Wrapping the object literal in parentheses signals that the braces are an object literal instead of the function body.

### Immediately-Invoked Function Expressions (IIFEs)

A popular use of functions in JavaScript is immediately-invoked function expressions (IIFEs). IIFEs allow you to define an anonymous function and call it immediately without saving a reference. This pattern comes in handy when you want to create a scope that is shielded from the rest of a program. For example:

```js
let person = function(name) {

    return {
        getName() {
            return name;
        }
    };

}("Nicholas");

console.log(person.getName());      // "Nicholas"
```

In this code, the IIFE is used to create an object with a `getName()` method. The method uses the `name` argument as the return value, effectively making `name` a private member of the returned object.

You can accomplish the same thing using arrow functions so long as you wrap the arrow function in parentheses:

```js
let person = ((name) => {

    return {
        getName() {
            return name;
        }
    };

})("Nicholas");

console.log(person.getName());      // "Nicholas"
```

Note that the location of the parentheses is around just the arrow function definition, and does not include `("Nicholas")`. This is different from a formal function, where the parentheses can be placed outside of the passed-in parameters as well as just as around the function definition.

### Lexical this Binding

One of the most common areas of error in JavaScript is the binding of `this` inside of functions. Since the value of `this` can change inside of a single function depending on the context in which it's called, it's possible to mistakenly affect one object when you meant to affect another. Consider the following example:

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // error
        }, false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

In this code, the object `PageHandler` is designed to handle interactions on the page. The `init()` method is called to set up the interactions and that method in turn assigns an event handler to call `this.doSomething()`. However, this code doesn't work as intended. The call to `this.doSomething()` is broken because `this` is a reference to the element object (in this case `document`) that was the target of the event, instead of being bound to `PageHandler`. If you tried to run this code, you will get an error when the event handler fires because `this.doSomething()` doesn't exist on the target `document` object.

You can bind the value of `this` to `PageHandler` explicitly using the `bind()` method on the function:

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // no error
        }).bind(this), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

Now the code works as expected, but may look a little bit strange. By calling `bind(this)`, you're actually creating a new function whose `this` is bound to the current `this`, which is `PageHandler`. The code now works as you would expect even though you had to create an extra function to get the job done.

Arrow functions have implicit `this` binding, which means that the value of `this` inside of an arrow function is always the same as the value of `this` in the scope in which the arrow function was defined. For example:

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

The event handler in this example is an arrow function that calls `this.doSomething()`. The value of `this` is the same as it is within `init()`, so this version of the example works similarly to the one using `bind()`. Even though the `doSomething()` method doesn't return a value, it is still the only statement executed necessary for the function body and so there is no need to include braces.

Arrow functions are designed to be "throwaway" functions and so cannot be used to define new types. This is evident by the missing `prototype` property that regular functions have. If you try to use the `new` operator with an arrow function, you'll get an error:

```js
var MyType = () => {},
    object = new MyType();  // error - you can't use arrow functions with 'new'
```

Also, since the `this` value is statically bound to the arrow function, you cannot change the value of `this` using `call()`, `apply()`, or `bind()`.

The concise syntax for arrow functions makes them ideal for use with array processing. For example, if you want to sort an array using a custom comparator, you typically write something like this:

```js
var result = values.sort(function(a, b) {
    return a - b;
});
```

That's a lot of syntax for a very simple procedure. Compare that to the more terse arrow function version:

```js
var result = values.sort((a, b) => a - b);
```

The array methods that accept callback functions such as `sort()`, `map()`, and `reduce()` all can benefit from simpler syntax with arrow functions to change what would appear to be more complex processes into simpler code.

Generally speaking, arrow functions are designed to be used in places where anonymous functions have traditionally been used. They are not really designed to be kept around for long periods of time, hence the inability to use arrow functions as constructors. Arrow functions are best used for callbacks that are passed into other functions, as seen in the examples in this section.

### Lexical arguments Binding

Even though arrow functions don't have their own `arguments` object, it's possible for them to access the `arguments` object from a containing function. That `arguments` object is then available no matter where the arrow function is executed later on. For example:

```js
function createArrowFunctionReturningFirstArg() {
    return () => arguments[0];
}

var arrowFunction = createArrowFunctionReturningFirstArg(5);

console.log(arrowFunction());       // 5
```

Inside of `createArrowFunctionReturningFirstArg()`, `arguments[0]` is referenced by the created arrow function. That reference contains the first argument passed to `createArrowFunctionReturningFirstArg()`. When the arrow function is later executed, it returns `5`, which was the first argument passed in to `createArrowFunctionReturningFirstArg()`. Even though the arrow function is no longer in the scope of the function that created it, `arguments` remains accessible as a lexical binding.

### Identifying Arrow Functions

Despite the different syntax, arrow functions are still functions and are identified as such:

```js
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true
```

Both `typeof` and `instanceof` behave the same with arrow functions as they do with other functions.

Also like other functions, you can still use `call()`, `apply()`, and `bind()`, although the `this`-binding of the function will not be affected. Here are some examples:

```js
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3

var boundSum = sum.bind(null, 1, 2);

console.log(boundSum());                // 3
```

In this example, the `sum()` function is called using `call()` and `apply()` to pass arguments as you would with any function. The `bind()` method is used to create `boundSum()`, which has its two arguments bound to `1` and `2` so that they don't need to be passed directly.

Arrow functions are appropriate to use anywhere you're currently using an anonymous function expression, such as with callbacks.

## Summary

Functions haven't undergone a huge change in ECMAScript 6, but rather, a series of incremental changes that make them easier to work with.

Default function parameters allow you to easily specify what value to use when a particular argument isn't passed. Prior to ECMAScript 6, this would require some extra code inside of the function to both check for the presence of arguments and assign a different value.

Rest parameters allow you to specify an array into which all remaining parameters should be placed. Using a real array and letting you indicate which parameters to include makes rest parameters a much more flexible solution than `arguments`.

Destructured parameters use the destructuring syntax to make options objects more transparent when used as function parameters. The actual data you're interested in can be listed out along with other named parameters.

The spread operator is a companion to rest parameters, allowing you to destructure an array into separate parameters when calling a function. Prior to ECMAScript 6, the only ways to pass individual parameters that were contained in an array were either manually specifying each parameter or using `apply()`. With the spread operator, you can easily pass an array to any function without worrying about the `this` binding of the function.

The addition of the `name` property helps to more easily identify functions for debugging and evaluation purposes. Additionally, ECMAScript 6 formally defines the behavior of block-level functions so they are no longer a syntax error in strict mode.

The behavior of a function has been defined by `[[Call]]`, normal function execution, and `[[Construct]]`, when a function is called with `new`. The `new.target` metaproperty allows you to determine if a function was called using `new` or not.

The biggest change to functions in ECMAScript 6 was the addition of arrow functions. Arrow functions are designed to be used in places where anonymous function expressions have traditionally been used. Arrow functions have a more concise syntax, lexical `this` binding, and no `arguments` object. Additionally, arrow functions can't change their `this` binding and so can't be used as constructors.
