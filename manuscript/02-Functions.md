# Functions

Functions are an important part of any programming language, and JavaScript functions hadn't changed much since the language was first introduced. This left a backlog of problems and nuanced behavior that made it easy to make mistakes or require more code just to achieve a very common behavior.

ECMAScript 6 functions made a big leap forward, taking into account years of complaints and asks from JavaScript developers. The result is a number of incremental improvements on top of ECMAScript 5 functions that make programming in JavaScript less error-prone and more powerful than ever before.

## Default Parameters

Functions in JavaScript are unique in that they allow any number of parameters to be passed regardless of the number of declared parameters in the function definition. This allows you to define functions that can handle different number of parameters, often by just filling in default values when ones are provided. In ECMAScript 5 and earlier, you would likely use the following pattern to accomplish this:

```js
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // the rest of the function

}
```

In this example, both `timeout` and `callback` are actually optional because they are given a default value if not provided. The logical OR operator (`||`) always returns the second operand when the first is falsy. Since named function parameters that are not explicitly provided are set to `undefined`, the logical OR operator is frequently used to provide default values for missing parameters. There is a flaw with this approach, however, in that a valid value for `timeout` might actually be `0`, but this could would replace it with `2000` because `0` is falsy.

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

Any parameters with a default value are considered to be optional parameters while those without default value are considered to be required parameters.

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

Perhaps the most interesting feature of default parameter arguments is that the default value need not be a primitive value. You can, for example, execute a function to retrieve the default parameter:

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
function sum(first) {
    let result = first,
        i = 1,
        len = arguments.length;

    while (i < len) {
        result += arguments[i];
        i++;
    }

    return result;
}
```

This function adds together all of the parameters that are passed to it so you can call `sum(1)` or `sum(1,2,3,4)` and it will still work. There are couple of things to notice about this function. First, it's not at all obvious that the function is capable of handling more than one parameter. You could add in several more named parameters, but you would always fall short of indicating that this function can take any number of parameters. Second, because the first parameter is named and used directly, you have to start looking in the `arguments` object at index 1 instead of starting at index 0. Remembering to use the appropriate indices with `arguments` isn't necessarily difficult, but it's one more thing to keep track of. ECMAScript 6 introduces rest parameters to help with these issues.

Rest parameters are indicated by three dots (`...`) preceding a named parameter. That named parameter then becomes an `Array` containing the rest of the parameters (which is why these are called "rest" parameters). For example, `sum()` can be rewritten using rest parameters like this:

```js
function sum(first, ...numbers) {
    let result = first,
        i = 0,
        len = numbers.length;

    while (i < len) {
        result += numbers[i];
        i++;
    }

    return result;
}
```

In this version of the function, `numbers` is a rest parameter that contains all parameters after the first one (unlike `arguments`, which contains all parameters including the first one). That means you can iterate over `numbers` from beginning to end without worry. As a bonus, you can tell by looking at the function that it is capable of handling any number of parameters.

I> The `sum()` method doesn't actually need any named parameters. You could, in theory, use only rest parameters and have it continue to work exactly the same. However, in that case, the rest parameters would effectively be the same as `arguments`, so you're not really getting any additional benefit.

The only restriction on rest parameters is that no other named arguments can follow in the function declaration. For example, this causes syntax error:

```js
// Syntax error: Can't have a named parameter after rest parameters
function sum(first, ...numbers, last) {
    let result = first,
        i = 0,
        len = numbers.length;

    while (i < len) {
        result += numbers[i];
        i++;
    }

    return result;
}
```

Here, the parameter `last` follows the rest parameter `numbers` and causes a syntax error.

Rest parameters were designed to replace `arguments` in ECMAScript. Originally ECMAScript 4 did away with `arguments` and added rest parameters to allow for an unlimited number of arguments to be passed to functions. Even though ECMAScript 4 never came into being, the idea was kept around and reintroduced in ECMAScript 6 despite `arguments` not being removed from the language.

## The Spread Operator

Closely related to rest parameters is the spread operator. Whereas rest parameters allow you to specify multiple independent arguments should be combined into an array, the spread operator allows you to specify an array that should be be split and have its items passed in as separate arguments to a function. Consider the `Math.max()` method, which accepts any number of arguments and returns the one with the highest value. It's basic usage is as follows:

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

## Arrow Functions

One of the most interesting new parts of ECMAScript 6 are arrow functions. Arrow functions are, as the name suggests, functions defined with a new syntax that uses an "arrow" (`=>`). However, arrow functions behave differently than traditional JavaScript functions in a number of important ways:

* **Lexical `this` binding** - The value of `this` inside of the function is determined by where the arrow function is defined not where it is used.
* **Not `new`able** - Arrow functions cannot be used a constructors and will throw an error when used with `new`.
* **Can't change `this`** - The value of `this` inside of the function can't be changed, it remains the same value throughout the entire lifecycle of the function.
* **No `arguments` object** - You can't access arguments through the `arguments` object, you must use named arguments or other ES6 features such as rest arguments.

There are a few reasons why these differences exist. First and foremost, `this` binding is a common source of error in JavaScript. It's very easy to lose track of the `this` value inside of a function and can easily result in unintended consequences. Second, by limiting arrow functions to simply executing code with a single `this` value, JavaScript engines can more easily optimize these operations (as opposed to regular functions, which might be used as a constructor or otherwise modified).

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
var sum = (a, b) => a + b;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3

var newSum = sum.bind(null, 1, 2);

console.log(newSum());                  // 3
```

In this example, the `sum()` function is called using `call()` and `apply()` to pass arguments as you would with any function. The `bind()` method is used to create `newSum()`, which has its two arguments bound to `1` and `2` so that they don't need to be passed directly.

Arrow functions are appropriate to use anywhere you're currently using an anonymous function expression, such as with callbacks.

## Summary

Functions haven't undergone a huge change in ECMAScript 6, but rather, a series of incremental changes that make them easier to work with.

Default function parameters allow you to easily specify what value to use when a particular argument isn't passed. Prior to ECMAScript 6, this would require some extra code inside of the function to both check for the presence of arguments and assign a different value.

Rest parameters allow you to specify an array into which all remaining parameters should be placed. Using a real array and letting you indicate which parameters to include makes rest parameters a much more flexible solution than `arguments`.

The spread operator is a companion to rest parameters, allowing you to destructure an array into separate parameters when calling a function. Prior to ECMAScript 6, the only ways to pass individual parameters that were contained in an array were either manually specifying each parameter or using `apply()`. With the spread operator, you can easily pass an array to any function without worrying about the `this` binding of the function.

The biggest change to functions in ECMAScript 6 was the addition of arrow functions. Arrow functions are designed to be used in places where anonymous function expressions have traditionally been used. Arrow functions have a more concise syntax, lexical `this` binding, and no `arguments` object. Additionally, arrow functions can't change their `this` binding and so can't be used as constructors.
