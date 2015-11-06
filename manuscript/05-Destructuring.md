# Destructuring

Objects and arrays are two of the most frequently used types in JavaScript. Thanks to the popularity of JSON as a data format, object and array literals have become an important part of JavaScript. It's quite common to define objects and arrays, and then systematically pull out the relevant pieces of information from those structures. In ECMAScript 5 and earlier, this could lead to a lot of code that looks the same, just to get certain data into local variables. For example:

```js
let options = {
        repeat: true,
        save: false
    };

// extract data from the object
let repeat = options.repeat,
    save = options.save;
```

This code extracts information from the `options` object and stores them in local variables of the same name. While this code looks simple, imagine if you had a large number of variables to assign, or if there was a nested data structure to traverse to find the information. This is where destructuring comes in.

## What is Destructuring?

ECMAScript 6 adds destructuring for both objects and arrays. *Destructuring* is the process of breaking down a data structure into smaller parts. Many languages implement destructuring using a minimal amount of syntax to make it simpler to use. The ECMAScript 6 implementation of destructuring makes use of syntax you're already familiar with: object and array literals.

### Object Destructuring

Object destructuring syntax uses an object literal on the left side of an assignment operation. For example:

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

In this code, the value of `node.type` is stored in a variable called `type` and the value of `node.name` is stored in a variable called `name`. This syntax is the same as the object literal property initializer shorthand introduced in Chapter 4. The identifiers `type` and `name` are both declarations of local variables and the property to read the value from on `options`.

A> #### Don't Forget the Initializer
A>
A>When using destructuring to declare variables using `var`, `let`, or `const`, you must supply an initializer (the value after the equals sign). The following lines of code will all throw syntax errors due to a missing initializer:
A>
A>```js
A>// syntax error!
A>var { type, name };
A>
A>// syntax error!
A>let { type, name };
A>
A>// syntax error!
A>const { type, name };
A>```
A>
A>While `const` always requires an initializer, even when using nondestructured variables, `var` and `let` only require initializers when using destructuring.

#### Destructuring Assignment

The object destructuring examples so far have used variable declarations. However, it's also possible to use destructuring in assignments. For instance, you may decide to change the value of variables after they are defined:

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

// assign different values using structuring
({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

In this example, `type` and `name` are initialized with values when declared. The next line uses destructuring assignment to change those values by reading from `node`. Note that you must put parentheses around a destructuring assignment statement. That's because an opening curly brace is expected to a be a block statement, and a block statement cannot appear on the left side of an assignment. The parentheses signal that the next curly brace is not a block statement and should be interpreted as an expression, allowing the assignment to complete.

#### Default Values

If you specify a property name that doesn't exist on the object, then the local variable is assigned a value of `undefined`. For example:

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // undefined
```

This code defines an additional local variable, `value`, and attempts to assign it a value. However, there is no corresponding `value` property on `node`, so it's assigned the value of `undefined`.

You can optionally define a different value to use when a specified property doesn't exist. To do so, insert an equals sign (`=`) after the property name and specify the default value. For example:

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value = true } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // true
```

In this example, the variable `value` is given a default value of `true` that is used only if the property is missing or has the value of `undefined`. Since there is no `node.value`, the variable `value` uses the default value. This works similar to default parameter values as discussed in Chapter 3.

#### Assigning to Different Local Variable Names

To this point, each example has used the object property name as the local variable name, so the value of `options.repeat` was stored in `repeat`. That works well when you want to use the same name, but what if you don't? There is an extended syntax that allows you to assign to a local variable with a different name, and that syntax looks like the object literal nonshorthand property initializer syntax. Here's an example:

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type: localType, name: localName } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "foo"
```

This code uses destructuring assignment to declare two variables, `localType` and `localName`, that contain the values from `node.type` and `node.name`, respectively. The syntax `type: localType` says to read the property named `type` and store it in the variable `localType`. This syntax is effectively the opposite of traditional object literal syntax where the name is on the left of the colon and the value is on the right. In this case, the name is on the right of the colon and the location of the value to read is on the left.

You can add default values when using a different variable name, as well. The equals sign and default value are placed after the local variable name. For example:

```js
let node = {
        type: "Identifier"
    };

let { type: localType, name: localName = "bar" } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "bar"
```

Here, the `localName` variable has a default value of `"bar"`. The default value is used because there is no `node.name` property, so `localName` has a value of `"bar"`.

So far, you've seen how to deal with destructuring of an object whose properties are primitive values. Object destructuring can also be used to retrieve values in nested object structures.

#### Nested Object Destructuring

By using a syntax similar to object literals, you can navigate into a nested object structure to retrieve just the information you want. Here's an example:

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

let { loc: { start }} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
```

The destructuring pattern in this example uses curly braces to indicate that the pattern should descend into the property named `loc` and look for the property named `start`. Remember from the last section that whenever there's a colon in a destructuring pattern, it means the identifier before the colon is giving a location to inspect and the right side is the one that assigns a value. When there's a curly brace after the colon, that indicates another level of pattern.

You can go one step further and use a different name for the local variable as well:

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

// extract node.loc.start
let { loc: { start: localStart }} = node;

console.log(localStart.line);   // 1
console.log(localStart.column); // 1
```

In this version of the code, `node.loc.start` is stored in a new local variable `localStart`. Destructuring patterns can be nested to an arbitrary level of depth, with all capabilities available at each level.

While object destructuring is very powerful and has a lot of options, array destructuring offers some unique capabilities that allow you to extract information from arrays.

### Array Destructuring

Array destructuring works in a way that is very similar to object destructuring, with the exception that it uses array literal syntax instead of object literal syntax. The destructuring operates on positions within an array rather than the named properties that are available in objects. For example:

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

In this example, array destructuring pulls out the first and second values in the `colors` array and stores them in `firstColor` and `secondColor`. The values stores in these variables, `"red"` and `"green"`, are chosen because of the position in the array (the actual variables names can be anything). Any items not explicitly mentioned in the destructuring pattern are ignored. Keep in mind that the array itself isn't changed in any way.

You can also omit items in the destructuring pattern and only provide variable names for the items you're interested in. For example, suppose you just want the third value of an array. You don't need to supply variable names for the first and second items:

```js
let colors = [ "red", "green", "blue" ];

let [ , , thirdColor ] = colors;

console.log(thirdColor);        // "blue"
```

This code uses a destructuring assignment to retrieve the third item in `colors`. The commas preceding `thirdColor` in the pattern are placeholders for the array items that come before it. By using this approach, you can easily pick out values from any number of slots in the middle of an array without needing to provide variable names for them.

W> Similar to object destructuring, you must always provide an initializer when using array destructuring with `var`, `let`, or `const`.

#### Destructuring Assignment

You can use array destructuring in the context of an assignment, but unlike object destructuring, there is no need to wrap the expression in parentheses. For example:

```js
let colors = [ "red", "green", "blue" ],
    firstColor = "black",
    secondColor = "purple";

[ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

The destructured assignment in this example works in a similar manner to the example from the previous section. The only difference is that `firstColor` and `secondColor` have already been defined. But there's a little bit more to array destructuring assignment than may be obvious.

Array destructuring assignment has a very unique use case in that it's possible to swap the value of two variables. Value swapping is a common operation in sorting algorithms, and the ECMAScript 5 way of swapping variables involves a third, temporary variable:

```js
// Swapping variables in ECMAScript 5
let a = 1,
    b = 2,
    tmp;

tmp = a;
a = b;
b = tmp;

console.log(a);     // 2
console.log(b);     // 1
```

The intermediate variable `tmp` is necessary in order to swab the values of `a` and `b`. Using array destructuring assignment, however, there's no need for that extra variable. Here's how you can swap variables in ECMAScript 6:

```js
// Swapping variables in ECMAScript 6
let a = 1,
    b = 2;

[ a, b ] = [ b, a ];

console.log(a);     // 2
console.log(b);     // 1
```

The array destructuring assignment in this example looks like a mirror image. The left side of the assignment (before the equals sign) is the destructuring pattern just like you've seen in the previous examples. The right side is an array literal that is temporarily created for the purposes of doing the swap. The destructuring happens on the temporary array, which has the values of `b` and `a` copied into its first and second positions. The effect is that the variables have swapped values.

#### Default Values

Another similarity with object destructuring is the ability to specify a default value for any position in the array. The default value is used when the given position either doesn't exist or has the value `undefined`. For example:

```js
let colors = [ "red" ];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

In this code, the `colors` array has only one item, so there is nothing for `secondColor` to match. The default value `"green"` is used instead of the default (`undefined`).

#### Nested Destructuring

You can destructure nested arrays in a manner similar to object destructuring nested objects. By inserting another array pattern into the overall pattern, the destructuring will descend into a nested array. For example:

```js
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// later

let [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

Here, the `secondColor` variable refers to the `"green"` value inside of the `colors` array. That item is contained within a second array, so the extra square brackets around `secondColor` in the destructuring pattern is necessary. As with objects, you can nest arrays arbitrarily deep.

#### Rest Items

You learned about rest parameters for functions in Chapter 3 and array destructuring has a similar concept called *rest items*. Rest items use the `...` syntax to assign the remaining items in an array to a particular variable. Here's an example:

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, ...restColors ] = colors;

console.log(firstColor);        // "red"
console.log(restColors.length); // 2
console.log(restColors[0]);     // "green"
console.log(restColors[0]);     // "blue"
```

In this code, the first item in `colors` in assigned to `firstColor`, and the rest are assigned into a new array in `restColors`. The `restColors` array, therefore, has two items: `"green"` and `"blue"`. Rest items are useful for extracting certain items from an array and keeping the rest available, but there's also another helpful use.

A glaring omission from JavaScript objects is the ability to easily create a clone. In ECMAScript 5, developers frequently used the `concat()` method as an easy way to clone an array. For example:

```js
// cloning an array in ECMAScript 5
var colors = [ "red", "green", "blue" ];
var clonedColors = colors.concat();

console.log(clonedColors);      //"[red,green,blue]"
```

While the `concat()` method is intended to concatenate two arrays together, calling it without an argument returns a clone of the array. In ECMAScript 6, you can use rest items to achieve the same thing:

```js
// cloning an array in ECMAScript 6
let colors = [ "red", "green", "blue" ];
let [ ...clonedColors ] = colors;

console.log(clonedColors);      //"[red,green,blue]"
```

In this example, rest items are used to copy values from `colors` into `clonedColors`. While it's a matter of perception as to whether this or `concat()` makes the developer's intent clearer, this is a useful ability to be aware of.



## Mixed Destructuring

It's possible to mix objects and arrays together in a destructuring assignment expression using a mix of object and array literals. For example:

```js
let options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

let { repeat, save, colors: [ firstColor, secondColor ]} = options;

console.log(repeat);            // true
console.log(save);              // false
console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

This example extracts two property values, `repeat` and `save`, and then two items from the `colors` array, `firstColor` and `secondColor`. Of course, you could also choose to retrieve the entire array:

```js
let options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

let { repeat, save, colors } = options;

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

    let secure = options.secure,
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

    let { secure, path, domain, expires } = options;

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
