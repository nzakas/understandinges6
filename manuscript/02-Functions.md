# Functions

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

TODO: Intro

## Default Parameters

Functions in JavaScript are unique in that they allow any number of parameters to be passed regardless of the number of declared parameters in the function definition. This allows you to define functions that can handle different number of parameters, often by just filling in default values when ones are provided. In ECMAScript 5 and earlier, you would likely use the following pattern to accomplish this:

```js
function combineText(start, middle, end) {

    middle = middle || "";
    end = end || "";

    return start + middle + end;
}
```

The logical OR operator (`||`) always returns the second operand when the first is falsy. Since named function parameters that are not explicitly provided are set to `undefined`, the logical OR operator is frequently used to provide default values for missing parameters. Other ways of determining if any parameters are missing include checking `arguments.length` for the number of parameters that were passed or directly inspecting each parameter to see if it is `undefined`.

ECMAScript 6 makes it easier to provide default values for parameters by providing initializations that are used when the parameter isn't formally passed. For example:

```js
function combineText(start, middle = "", end = "") {
    return start + middle + end;
}
```

Here, only the first parameter is expected to be passed all the time. The other two parameters have default values of an empty string, which makes the body of the function much smaller because you don't need to add any code to check for a missing value. When `combineText()` is called with all three parameters, then the defaults are not used.

Any parameters with a default value are considered to be optional parameters while those without default value are considered to be required parameters.

It's possible to specify default values for any arguments, including those that appear before arguments without default values. For example, this is fine:

```js
function combineText(start, middle = "", end) {
    return start + middle + end;
}
```

In this case, the default value for `middle` will only be used if there is no second argument passed in or if the second argument is explicitly passed in as `undefined`. For example:

```js
// uses default
combineText("Hello", undefined, "Goodbye");

// uses default
combineText("Hello");

// doesn't use default
combineText("Hello", null, "Goodbye");
```

In the case of default parameter values, the value of `null` is considered to be valid and the default value will not be used.
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

One of the most interesting new parts of ECMAScript 6 are arrow functions. Arrow functions are, as the name suggests, functions defined with a new syntax that uses an "arrow" (`=>`) as part of the syntax. However, arrow functions behave differently than traditional JavaScript functions in a number of important ways:

* **Lexical `this` binding** - The value of `this` inside of the function is determined by where the arrow function is defined not where it is used.
* **Not `new`able** - Arrow functions cannot be used a constructors and will throw an error when used with `new`.
* **Can't change `this`** - The value of `this` inside of the function can't be changed, it remains the same value throughout the entire lifecycle of the function.
* **No `arguments` object** - You can't access arguments through the `arguments` object, you must use named arguments or other ES6 features such as rest arguments.

There are a few reasons why these differences exist. First and foremost, `this` binding is a common source of error in JavaScript. It's very easy to lose track of the `this` value inside of a function and can easily result in unintended consequences. Second, by limiting arrow functions to simply executing code with a single `this` value, JavaScript engines can more easily optimize these operations (as opposed to regular functions, which might be used as a constructor or otherwise modified).

## Syntax

The syntax for arrow functions comes in many flavors depending upon what you are trying to accomplish. All variations begin with function arguments, followed by the arrow, followed by the body of the function. Both the arguments and the body can take different forms depending on usage. For example, the following arrow function takes a single argument and simply returns it:

    var reflect = value => value;

    // effectively equivalent to:

    var reflect = function(value) {
        return value;
    };

When there is only one argument for an arrow function, that one argument can be used directly without any further syntax. The arrow comes next and the expression to the right of the arrow is evaluated and returned. Even though there is no explicit `return` statement, this arrow function will return the first argument that is passed in.

If you are passing in more than one argument, then you must include parentheses around those arguments. For example:

    var sum = (num1, num2) => num1 + num2;

    // effectively equivalent to:

    var sum = function(num1, num2) {
        return num1 + num2;
    };

The `sum()` function simply adds two arguments together and returns the result. The only difference is that the arguments are enclosed in parentheses with a comma separating them (same as traditional functions).

When you want to provide a more traditional function body, perhaps consisting of more than one expression, then you need to wrap the function body in braces and explicitly define a return value, such as:

    var sum = (num1, num2) => { return num1 + num2; }

    // effectively equivalent to:

    var sum = function(num1, num2) {
        return num1 + num2;
    };

You can more or less treat the inside of the curly braces as the same as in a traditional function with the exception that `arguments` is not available.

Because curly braces are used to denote the function's body, an arrow function that wants to return an object literal outside of a function body must wrap the literal in parentheses. For example:

    var getTempItem = id => ({ id: id, name: "Temp" });

    // effectively equivalent to:

    var getTempItem = function(id) {

        return {
            id: id,
            name: "Temp"
        };
    };

Wrapping the object literal in parentheses signals that the braces are an object literal instead of the function body.

### Lexical this Binding

One of the most common areas of error in JavaScript is the binding of `this` inside of functions. Since the value of `this` can change inside of a single function depending on the context in which it's called, it's possible to mistakenly affect one object when you meant to affect another. Consider the following example:

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

In this code, the object `PageHandler` is designed to handle interactions on the page. The `init()` method is called to set up the interactions and that method in turn assigns an event handler to call `this.doSomething()`. However, this code doesn't work as intended. The call to `this.doSomething()` is broken because `this` is a reference to the element object (in this case `document`) that was the target of the event, instead of being bound to `PageHandler`. If you tried to run this code, you will get an error when the event handler fires because `this.doSomething()` doesn't exist on the target `document` object.

You can bind the value of `this` to `PageHandler` explicitly using the `bind()` method on the function:

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

Now the code works as expected, but may look a little bit strange. By calling `bind(this)`, you're actually creating a new function whose `this` is bound to the current `this`, which is `PageHandler`. The code now works as you would expect even though you had to create an extra function to get the job done.

Arrow functions have implicit `this` binding, which means that the value of `this` inside of an arrow function is always the same as the value of `this` in the scope in which the arrow function was defined. For example:

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

The event handler in this example is an arrow function that calls `this.doSomething()`. The value of `this` is the same as it is within `init()`, so this version of the example works similarly to the one using `bind()`. Even though the `doSomething()` method doesn't return a value, it is still the only statement executed necessary for the function body and so there is no need to include braces.

Arrow functions are designed to be "throwaway" functions and so cannot be used to define new types. This is evident by the missing `prototype` property that regular functions have. If you try to use the `new` operator with an arrow function, you'll get an error:

    var MyType = () => {},
        object = new MyType();  // error - you can't use arrow functions with 'new'

Also, since the `this` value is statically bound to the arrow function, you cannot change the value of `this` using `call()`, `apply()`, or `bind()`.

The concise syntax for arrow functions makes them ideal for use with array processing. For example, if you want to sort an array using a custom comparator, you typically write something like this:

    var result = values.sort(function(a, b) {
        return a - b;
    });

That's a lot of syntax for a very simple procedure. Compare that to the more terse arrow function version:

    var result = values.sort((a, b) => a - b);

The array methods that accept callback functions such as `sort()`, `map()`, and `reduce()` all can benefit from simpler syntax with arrow functions to change what would appear to be more complex processes into simpler code.

Generally speaking, arrow functions are designed to be used in places where anonymous functions have traditionally been used. They are not really designed to be kept around for long periods of time, hence the inability to use arrow functions as constructors. Arrow functions are best used for callbacks that are passed into other functions, as seen in the examples in this section.

### Still Functions

Despite the different syntax, arrow functions are still functions and are identified as such:

```js
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true
```

Both `typeof` and `instanceof` behave the same with arrow functions as they do with other functions.

Also like other functions, you can still use `call()`, `apply()`, and `bind()`, although the `this`-binding of the function will not be affected.

TODO: Example



--------------

TODO: Duplicate content below, figure out how to incorporate.


## Other things to know

Arrow functions are different than traditional functions, but do share some common characteristics. For example:

* The `typeof` operator returns "function" for arrow functions.
* Arrow functions are still instances of `Function`, so `instanceof` works the same way.
* The methods `call()`, `apply()`, and `bind()` are still usable with arrow functions, though they do not augment the value of `this`.

The biggest difference is that arrow functions can't be used with `new`—attempting to do so results in an error being thrown.

## Conclusion

Arrow functions are an interesting new feature in ECMAScript 6, and one of the features that is pretty solidified at this point in time. As passing functions as arguments has become more popular, having a concise syntax for defining these functions is a welcome change to the way we've been doing this forever. The lexical `this` binding solves a major painpoint for developers and has the added bonus of improving performance through JavaScript engine optimizations. If you want to try out arrow functions, just fire up the latest version of Firefox, which is the first browser to ship an implementation in their official release.
