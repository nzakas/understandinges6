# Destructuring

JavaScript developers spend a lot of time pulling data out of objects and arrays. It's not uncommon to see code such as this:

```js
var options = {
        repeat: true,
        save: false
    };

// later

var localRepeat = options.repeat,
    localSave = options.save;
```

Frequently, object properties are stored into local variables for more succinct code and easier access. ECMAScript 6 makes this easy by introducing *destructuring assignment*, which systematically goes through an object or array and stores specified pieces of data into local variables.

W> If the right side value of a destructuring assignment evaluates to `null` or `undefined`, an error is thrown.

## Object Destructuring

Object destructuring assignment syntax uses an object literal on the left side of an assignment operation. For example:

```js
var options = {
        repeat: true,
        save: false
    };

// later

var { repeat: localRepeat, save: localSave } = options;

console.log(localRepeat);       // true
console.log(localSave);         // false
```

In this code, the value of `options.repeat` is stored in a variable called `localRepeat` and the value of `options.save` is stored in a variable called `localSave`. These are both specified using the object literal syntax where the key is the property to find on `options` and the value is the variable in which to store the property value.

I> If the property with the given name doesn't exist on the object, then the local variable gets a value of `undefined`.

If you want to use the property name as the local variable name, you can omit the colon and the identifier, such as:

```js
var options = {
        repeat: true,
        save: false
    };

// later

var { repeat, save } = options;

console.log(repeat);        // true
console.log(save);          // false
```

Here, two local variables called `repeat` and `save` are created. They are initialized with the value of `options.repeat` and `options.save`, respectively. This shorthand is helpful when there's no need to have different variable names.

Destructuring can also handle nested objects, such as the following:

```js
var options = {
        repeat: true,
        save: false,
        rules: {
            custom: 10,
        }
    };

// later

var { repeat, save, rules: { custom }} = options;

console.log(repeat);        // true
console.log(save);          // false
console.log(custom);        // 10
```

In this example, the `custom` property is embedded in another object. The extra set of curly braces allows you to descend into a nested object and pull out its properties.

A> ### Syntax Gotcha
A>
A> If you try use destructuring assignment without a `var`, `let`, or `const`, you may be surprised by the result:
A>
A> {:lang="js"}
A> ~~~~~~~~
A> // syntax error
A> { repeat, save, rules: { custom }} = options;
A> ~~~~~~~~
A>
A> This causes a syntax error because the opening curly brace is normally the beginning of a block and blocks can't be part of assignment expressions.
A>
A> The solution is to wrap the entire expression in parentheses:
A>
A> {:lang="js"}
A> ~~~~~~~~
A> // no syntax error
A> ({ repeat, save, rules: { custom }} = options);
A> ~~~~~~~~
A>
A> This now works without any problems.

## Array Destructuring

Similarly, you can destructure arrays using array literal syntax on the left side of an assignment operation. For example:

```js
var colors = [ "red", "green", "blue" ];

// later

var [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

In this example, array destructuring pulls out the first and second values in the `colors` array. Keep in mind that the array itself isn't changed in any way.

Similar to object destructuring, you can also nest array destructuring. Just use another set of square brackets to descend into a subarray:

```js
var colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// later

var [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

Here, the `secondColor` variable refers to the `"green"` value inside of the `colors` array. That item is contained within a second array, so the extra square brackets around `secondColor` in the destructuring assignment is necessary.

## Mixed Destructuring

It's possible to mix objects and arrays together in a destructuring assignment expression using a mix of object and array literals. For example:

```js
var options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

var { repeat, save, colors: [ firstColor, secondColor ]} = options;

console.log(repeat);            // true
console.log(save);              // false
console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

This example extracts two property values, `repeat` and `save`, and then two items from the `colors` array, `firstColor` and `secondColor`. Of course, you could also choose to retrieve the entire array:

```js
var options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

var { repeat, save, colors } = options;

console.log(repeat);                        // true
console.log(save);                          // false
console.log(colors);                        // "red,green,blue"
console.log(colors === options.colors);     // true
```

This modified example retrieves `options.colors` and stores it in the `colors` variable. Notice that `colors` is a direct reference to `options.colors` and not a copy.

Mixed destructuring is very useful for pulling values out of JSON configuration structures without navigating the entire structure.

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


## Summary

Destructuring makes it easier to work with objects and arrays in JavaScript. Using syntax that's already familiar to many developers, object literals and array literals, you can now pick data structures apart to get at just the information you're interested in.

Destructured parameters use the destructuring syntax to make options objects more transparent when used as function parameters. The actual data you're interested in can be listed out along with other named parameters.
