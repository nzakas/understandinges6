# Symbols

ECMAScript 6 symbols began as a way to create private object members, a feature JavaScript developers have long wanted. Any property with a string name was easy to access regardless of the obscurity of the name. The initial "private names" feature aimed to create non-string property names. That way, normal techniques for detecting these private names wouldn't work.

The private names proposal eventually evolved into ECMAScript 6 symbols. While the implementation details remained the same (non-string values for property names), the goal of privacy was dropped. Instead, symbol properties are categorized separately, non-enumerable by default but still discoverable.

Symbols are a new primitive type, joining the existing primitive types: strings, numbers, booleans, `null`, and `undefined`. Symbols are unique among JavaScript primitives in that they do not have a literal form (such as `true` for boolean or `42` for numbers). The ECMAScript 6 standard uses a special notation to indicate symbols, prefixing the identifier with `@@`, such as `@@create`. This book uses this same convention for ease of understanding.

W> Despite the notation, symbols do not exactly map to strings beginning with "@@". Don't try to use string values where symbols are required.

## Creating Symbols

You can create a symbol by using the global `Symbol` function, such as:

```js
var firstName = Symbol();
var person = {};

person[firstName] = "Nicholas";
console.log(person[firstName]);     // "Nicholas"
```

In this example, the symbol `firstName` is created and used to assign a new property on `person`. That symbol must be used each time you want to access that same property. It's a good idea to name the symbol variable appropriately so you can easily tell what the symbol represents.

W> Because symbols are primitive values, `new Symbol()` throws an error when called. It is possible to create an instance of `Symbol` via `new Object(yourSymbol)`, but it's unclear when this capability would be useful.

The `Symbol` function accepts an optional argument that is the description of the symbol. The description itself cannot be used to access the property but is used for debugging purposes. For example:

```js
var firstName = Symbol("first name");
var person = {};

person[firstName] = "Nicholas";

console.log("first name" in person);        // false
console.log(person[firstName]);             // "Nicholas"
console.log(firstName);                     // "Symbol(first name)"
```

A symbol's description is stored internally in a property called `[[Description]]`. This property is read whenever the symbol's `toString()` method is called either explicitly or implicitly (as in this example). It is not otherwise possible to access `[[Description]]` directly from code. It's recommended to always provide a description to make both reading and debugging symbols easier.

A> ### Identifying Symbols
A>
A>Since symbols are primitive values, you can use the `typeof` operator to determine if a variable contains a symbol. ECMAScript 6 extends `typeof` to return `"symbol"` when used on a symbol. For example:
A>
A>```js
A>var symbol = Symbol("test symbol");
A>console.log(typeof symbol);         // "symbol"
A>```
A>
A>While there are other indirect ways of determining whether a variable is a symbol, `typeof` is the most accurate and preferred way of doing so.

## Using Symbols

You can use symbols anywhere you would use a computed property name. You've already seen the use of bracket notation in this chapter, but you can use symbols in computed object literal property names as well as with `Object.defineProperty()`, and `Object.defineProperties()`, such as:

```js
var firstName = Symbol("first name");

// use a computed object literal property
var person = {
    [firstName]: "Nicholas"
};

// make the property read only
Object.defineProperty(person, firstName, { writable: false });

var lastName = Symbol("last name");

Object.defineProperties(person, {
    [lastName]: {
        value: "Zakas",
        writable: false
    }
});

console.log(person[firstName]);     // "Nicholas"
console.log(person[lastName]);      // "Zakas"
```

This example first uses a computed object literal property to create the `firstName` symbol property. The property is created as nonenumerable, which is different from computed properties created using nonsymbol names. The following line then sets the property to be read only. Later, a readonly `lastName` symbol property is created using `Object.defineProperties()`. A computed object literal property is used once again, but this time, it's part of the second argument to `Object.defineProperties()`.

While symbols can be used in any place that computed property names are allowed, you'll need to have a system for sharing these symbols in order to use them effectively.

## Symbol Coercion

Type coercion is a significant part of JavaScript, and there's a lot of flexibility in the language's ability to coerce one data type into another. Symbols, however, are quite inflexible when it comes to coercion because there exists no logical equivalent of a symbol in other types. Specifically, symbols cannot be coerced into strings or numbers so that they cannot accidentally be used as properties that would otherwise be expected to behave as symbols.

The examples in this chapter have used `console.log()` to indicate the output for symbols, and that works because `console.log()` calls `String()` on its arguments. You can use `String()` directly to get the same result. For instance:

```js
var uid = Symbol.for("uid"),
    desc = String(uid);

console.log(desc);              // "Symbol(uid)"
```

The `String()` function calls `uid.toString()` and the symbol's string description is returned. If you try to concatenate the symbol directly with a string, however, an error is thrown:

```js
var uid = Symbol.for("uid"),
    desc = uid + "";            // error!
```

Concatenating `uid` with an empty string requires that `uid` first be coerced into a string. An error is thrown when the coercion is detected, preventing its use in this manner.

Similarly, you cannot coerce a symbol to a number. All mathematical operators cause an error when applied to a symbol. For example:

```js
var uid = Symbol.for("uid"),
    sum = uid / 1;            // error!
```

This example attempts to divide the symbol by 1, which causes an error. Errors are thrown regardless of the mathematical operator used (logical operators do not throw an error because all symbols are considered equivalent to `true`, just like any other non-empty value in JavaScript).

## Sharing Symbols

You may find that you want different parts of your code to use the same symbols. For example, suppose you have two different object types in your application that should use the same symbol property to represent a unique identifier. Keeping track of symbols across files or large codebases can be difficult and error-prone. That's why ECMAScript 6 provides a global symbol registry that you can access at any point in time.

When you want to create a symbol to be shared, use the `Symbol.for()` method instead of calling `Symbol()`. The `Symbol.for()` method accepts a single parameter, which is a string identifier for the symbol you want to create (this value is also used as the description). For example:

```js
var uid = Symbol.for("uid");
var object = {};

object[uid] = "12345";

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"
```

The `Symbol.for()` method first searches the global symbol registry to see if a symbol with the key `"uid"` exists. If so, then it returns the already existing symbol. If no such symbol exists, then a new symbol is created and registered into the global symbol registry using the specified key. The new symbol is then returned. That means subsequent calls to `Symbol.for()` using the same key will return the same symbol:

```js
var uid = Symbol.for("uid");
var object = {
    [uid]: "12345"
};

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"

var uid2 = Symbol.for("uid");

console.log(uid === uid2);      // true
console.log(object[uid2]);      // "12345"
console.log(uid2);              // "Symbol(uid)"
```

In this example, `uid` and `uid2` contain the same symbol and so they can be used interchangeably. The first call to `Symbol.for()` creates the symbol and second call retrieves the symbol from the global symbol registry.

Another unique aspect of shared symbols is that you can retrieve the key associated with a symbol in the global symbol registry by using `Symbol.keyFor()`, for example:

```js
var uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));    // "uid"

var uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));   // "uid"

var uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));   // undefined
```

Notice that both `uid` and `uid2` return the key `"uid"`. The symbol `uid3` doesn't exist in the global symbol registry, so it has no key associated with it and `Symbol.keyFor()` returns `undefined`.

W> The global symbol registry is a shared environment, just like the global scope. That means you can't make assumptions about what is or is not already present in that environment. You should use namespacing of symbol keys to reduce the likelihood of naming collisions when using third-party components. For example, jQuery might prefix all keys with `"jquery."`, such as `"jquery.element"`.

## Retrieving Object Symbols

You may be familiar with the `Object.keys()` and `Object.getOwnPropertyNames()` methods for retrieving all property names in an object, with the former returning all enumerable property names and the latter returning all properties regardless of enumerability. Neither of these methods return symbol properties to preserve their ECMAScript 5 functionality. Instead, the `Object.getOwnPropertySymbols()` method was added to allow you to retrieve property symbols from an object.

The return value of `Object.getOwnPropertySymbols()` is an array of symbols for own properties. For example:

```js
var uid = Symbol.for("uid");
var object = {
    [uid]: "12345"
};

var symbols = Object.getOwnPropertySymbols(object);

console.log(symbols.length);        // 1
console.log(symbols[0]);            // "Symbol(uid)"
console.log(object[symbols[0]]);    // "12345"
```

In this code, `object` has a single symbol property `uid`. The array returned from `Object.getOwnPropertySymbols()` is an array containing just that symbol.

All objects start off with zero own symbol properties, however, objects can inherit symbol properties from their prototypes. ECMAScript 6 predefines several such properties.


## Well-Known Symbols

A central theme for both ECMAScript 5 and ECMAScript 6 was exposing and defining some of the "magic" parts of JavaScript - the parts that couldn't be emulated by a developer. ECMAScript 6 follows this tradition by exposing even more of the previously internal-only logic of the language. It does so primarily through the use of symbol prototype properties that define the basic behavior of certain objects.

ECMAScript 6 has predefined symbols called *well-known* symbols that represent common behaviors in JavaScript that were previously considered internal-only operations. Each well-known symbol is represented by a property on `Symbol`, such as `Symbol.create` for the `@@create` symbol.

The well-known symbols are:

* `@@hasInstance` - a method used by `instanceof` to determine an object's inheritance.
* `@@isConcatSpreadable` - a Boolean value indicating if use with `Array.prototype.concat()` should flatten the collection's elements.
* `@@iterator` - a method that returns an iterator (covered in Chapter 7 - Iterators and Generators).
* `@@match` - a method used by `String.prototype.match()` to compare strings.
* `@@replace` - a method used by `String.prototype.replace()` to replace substrings.
* `@@search` - a method used by `String.prototype.search()` to locate substrings.
* `@@species` - the constructor from which derived objects are made (covered in Chapter 8 - Classes).
* `@@split` - a method used by `String.prototype.split()` to split up strings.
* `@@toPrimitive` - a method that returns a primitive value representation of the object.
* `@@toStringTag` - a string used by `Object.prototype.toString()` to create an object description.
* `@@unscopables` - an object whose properties are the names of object properties that should not be included in a `with` statement.

Some of the well-known symbols are discussed below while others are discussed throughout the book to keep them in the correct context.

I> Overwriting a method defined with a well-known symbol changes an ordinary object to an exotic object because this changes some internal default behavior.

### @@hasInstance

The `@@hasInstance` symbol is the property on functions that determines whether or not a given object is an instance of that function. The symbol is represented in code by `Symbol.hasInstance` and the symbol property is defined on `Function.prototype` so that all functions inherit the default behavior for `instanceof`. The property itself is defined as nonwritable and onconfigurable as well as nonenumerable to ensure it doesn't get overwritten by mistake. The `@@hasInstance` method accepts a single argument, the value to check, and returns true if the value is an instance of the function.

To understand how `@@hasInstance` works, consider the following code:

```js
obj instanceof Array;
```

This code is equivalent to:

```js
Array[Symbol.hasInstance](obj);
```

Essentially, ECMAScript 6 redefined the `instanceof` operator as shorthand syntax for this method call. And now that there's a method call involved, you can actually change how `instanceof` works.

For instance, suppose you want to define a function that claims no object as an instance. You can do so by hardcoding the return value of `@@hasInstance` to `false`, such as:

```js
function MyObject() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});

var obj = new MyObject();

console.log(obj instanceof MyObject);       // false
```

This example uses `Object.defineProperty()` to overwrite the `@@hasInstance` method with a new function. (You must use `Object.defineProperty()` to overwrite a nonwritable property.) The function always returns `false`, so even though `obj` is actually an instance of `MyObject`, the `instanceof` operator returns `false`.

Of course, you can also inspect the value and decide whether or not a value should be considered an instance based on any arbitrary condition. For instance, maybe numbers with values between 1 and 100 are to be considered an instance of a special number type:

```js
function SpecialNumer() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return (v instanceof Number) && (v >=1 && v <= 100);
    }
});

var two = new Number(2),
    zero = new Number(0);

console.log(two instanceof SpecialNumber);    // true
console.log(zero instanceof SpecialNumber);   // false
```

This code defines a `@@hasInstance` method that returns true if the value is an instance of `Number` and also has a value between 1 and 100. Note that the left operand to `instanceof` must be an object to trigger the call to `@@hasInstance`, as nonobjects cause `instanceof` to simply return `false` all the time. This code allows `SpecialNumber` to claim `two` as an instance even though there is no directly relationship between them.

W> You can also overwrite the default `@@hasInstance` for all builtin functions such as `Date` and `Error`. This isn't recommended as the effects on your code can be unexpected and confusing. It's a good idea to only overwrite `@@hasInstance` on your own functions and only when necessary.

### @@isConcatSpreadable

JavaScript arrays have a `concat()` method that is designed to concatenate two arrays together, for example:

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ]);

console.log(colors2.length);    // 4
console.log(colors2);           // ["red","green","blue","black"]
```

This code concatenates a new array to the end of `colors1` and creates `colors2`, a single array with all items from both arrays. However, `concat()` can also accept nonarray arguments and, in that case, those arguments are simply added to the end of the array. For example:

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ], "brown");

console.log(colors2.length);    // 5
console.log(colors2);           // ["red","green","blue","black","brown"]
```

Here, the extra argument `"brown"` is passed to `concat()` and it becomes the fifth item in `colors2`. Why is an array argument treated differently than a string argument? The specification says that arrays are automatically split into their individual items and all other types are not. Prior to ECMAScript 6, there was no way to adjust this behavior.

The `@@isConcatSpreadable` property is a boolean value indicating that an object has a `length` property and numeric keys, and that its numeric property values should be added individually to the result of `concat()`. The symbol is represented in code by `Symbol.isConcatSpreadable` but unlike other well-known symbols, this symbol property doesn't appear on any standard objects by default. Instead, it's available as a way to augment how `concat()` works on certain types of objects, effectively short-circuiting the default behavior. That means you can define any type to behave like arrays in `concat()`, such as:

```js
let collection = {
    0: "Hello",
    1: "world",
    length: 2,
    [Symbol.isConcatSpreadable]: true
};

let messages = [ "Hi" ].concat(collection);

console.log(messages.length);    // 3
console.log(messages);           // ["hi","Hello","world"]
```

The `collection` object in this example is setup to look like an array: it has a `length` property and two numeric keys. The `@@isConcatSpreadable` property is set to `true` to indicate that the property values should be added as individual items to an array. When `collection` is passed to `concat()`, the resulting array has `"Hello"` and `"world"` as separate items after `"hi"`.

I> You can also set `@@isConcatSpreadable` to `false` on array subclasses to prevent items from being separated using `concat()`. Subclassing is discussed in Chapter 8.

### @@match, @@replace, @@search, and @@split

There has always been a close relationship between strings and regular expressions in JavaScript. The string type, in particular, has several methods accept regular expressions as arguments:

* `match(regex)` - determines if the given string matches a regular expression
* `replace(regex, replacement)` - replaces regular expression matches with a replacement
* `search(regex)` - locates a regular expression match inside the string
* `split(regex)` - splits a string into an array on a regular expression match

Prior to ECMAScript 6, the way these methods interacted with regular expressions was hidden from developers. That meant there was no way to mimic what regular expressions did using developer-defined objects. ECMAScript 6 defined four symbols that correspond to these four methods, effectively outsourcing the native behavior to the `RegExp` builtin object.

The `@@match`, `@@replace`, `@@search`, and `@@split` symbols represent methods on the regular expression argument that should be called on the first argument to the string methods `match()`, `replace()`, `search()`, and `split()`, respectively. The four symbol properties are defined on `RegExp.prototype` as the default implementation that the string methods should use. Knowing this, you can create an object to use with the string methods in a way that is similar to regular expressions. To do, you can use the following symbol in code:

* `Symbol.match` - a function that accepts a string argument and returns an array of matches or `null` if no match is found.
* `Symbol.replace` - a function that accepts a string argument and a replacement string, and returns a string.
* `Symbol.search` - a function that accepts a string argument and returns the numeric index of the match or -1 if no match is found.
* `Symbol.split` - a function that accepts a string argument and returns an array containing pieces of the string split on the match.

Here's an example:

```js
// effectively equivalent to /^.{10}$/
let hasLengthOf10 = {
    [Symbol.match] = function(value) {
        return value.length === 10 ? [value.substring(0, 10)] : null;
    },
    [Symbol.replace] = function(value, replacement) {
        return value.length === 10 ? replacement + value.substring(10) : value;
    },
    [Symbol.search] = function(value) {
        return value.length === 10 ? 0 : -1;
    },
    [Symbol.split] = function(value) {
        return value.length === 10 ? ["", ""] : [value];
    }
};

let message1 = "Hello world",   // 11 characters
    message2 = "Hello John";    // 10 characters


let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10);

console.log(match1);            // null
console.log(match2);            // ["Hello John"]

let replace1 = message1.replace(hasLengthOf10),
    replace2 = message2.replace(hasLengthOf10);

console.log(replace1);          // "Hello world"
console.log(replace2);          // "Hello John"

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10);

console.log(search1);           // -1
console.log(search2);           // 0

let split1 = message1.split(hasLengthOf10),
    split2 = message2.split(hasLengthOf10);

console.log(split1);            // ["Hello world"]
console.log(split2);            // ["", ""]
```

Here, `hasLengthOf10` is an object intended to work like a regular expression that matches whenever the string length is exactly 10. Each of the four methods is implemented using the appropriate symbols and then the corresponding methods on two strings are called. The first string, `message1`, has 11 characters and so will not match; the second string, `message2`, has 10 characters and so will match. Despite not being a regular expression, `hasLengthOf10` is passed to each string method and used correctly due to the additional methods.

While this is a simple example, the ability to perform more complex matches than are currently possible with regular expressions opens up a lot of possibilities.

### @@toPrimitive

JavaScript frequently attempts to convert objects into primitive values implicitly when certain operations are applied. For instance, when you compare a string to an object using double equals (`==`), the object is converted into a primitive value before comparing. Exactly what value should be used was previously an internal operation that is exposed in ECMAScript 6 through the `@@toPrimitive` method.

The `@@toPrimitive` method is defined on the prototype of each standard type and prescribes the exact behavior. When a primitive conversion is needed, `@@toPrimitive` is called with a single argument, referred to as `hint` in the specification. The `hint` argument is one of the following string values:

* `"number"` - a number should be returned
* `"string"` - a string should be returned
* `"default"` - the operation has no preference as to the type

For most standard objects, the behavior of number mode is:

1. Call `valueOf()`, and if the result is a primitive value, return it.
1. Otherwise, call `toString()`, and if the result is a primitive value, return it.
1. Otherwise, throw an error.

Similarly, for most standard objects, the behavior of string mode is:

1. Call `toString()`, and if the result is a primitive value, return it.
1. Otherwise, call `valueOf()`, and if the result is a primitive value, return it.
1. Otherwise, throw an error.

In many cases, standard objects treat default mode as equivalent to number mode (except for `Date`, which treats default mode as equivalent to string mode). By defining `@@toPrimitive`, you can override these default coercion behaviors.

I> Default mode is only used for `==`, `+`, and when passing a single argument to the `Date` constructor. Most operations use string or number mode.

To override the default behaviors, use `Symbol.toPrimitive` and assign a function as its value. For example:

```js
function Temperature(degrees) {
    this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {

    switch (hint) {
        case "string":
            return this.degrees + "\u00b0"; // degrees symbol

        case "number":
            return this.degrees;

        case "default":
            return this.degrees + " degrees";
    }
};

var freezing = new Temperature(32);

console.log(freezing + "!");            // "32 degrees!"
console.log(freezing / 2);              // 16
console.log(String(freezing));          // "32°"
```

This example defines a `Temperature` constructor and overrides the default `@@toPrimitive` method on the prototype. A different value is returned depending on `hint`, where string mode returns the temperature with the Unicode degrees symbol, number mode returns just the numeric value, and default mode appends the word "degrees" after the number. Each of the log statements triggers a different mode: the `+` operator triggers default mode, the `/` operator triggers number mode, and the `String()` function triggers string mode. While it's possible to return different values for all three modes, it's much more common to set to the default mode to be the same as string or number mode.

### @@toStringTag

One of the most interesting problems in JavaScript has been the availability of multiple global execution environments. This occurs in web browsers when a page includes an iframe, as the page and the iframe each have their own execution environments. In most cases, this isn't a problem, as data can be passed back and forth between the environments with little cause for concern. The problem arises when trying to identify what type of an object you're dealing with.

The canonical example of this is passing an array from the iframe into the containing page or vice-versa. Now in a different execution environment, `instanceof Array` returns `false` because the array was created with a constructor from a different environment.

Developers soon found a good way to identify arrays. It was discovered that by calling the standard `toString()` method on the object, a predictable string was always returned. Thus, many JavaScript libraries began including a function that works similar to this:

```js
function isArray(value) {
    return Object.prototype.toString.call(value) === "[object Array]";
}

console.log(isArray([]));   // true
```

This may look a bit roundabout, but in reality it was found to work quite well in all browsers. The `toString()` method on arrays isn't very useful for this purpose because it returns a string representation of the items it contains. The `toString()` method on `Object.prototype`, however, had this quirk where it included some internally-defined name in the result. By using this method on an object, you could retrieve what the JavaScript environment thought the data type was.

Developers quickly realized that since there was no way to change this behavior, it was possible to use the same approach to distinguish between native objects and those created by developers. The most important case of this was the ECMAScript 5 `JSON` object.

Prior to ECMAScript 5, many used Douglas Crockford's `json2.js`, which created a global `JSON` object. As browsers started to implement the `JSON` global object, it became necessary to tell whether the global `JSON` was provided by the JavaScript environment itself or through some other library. Using the same technique, many created functions like this:

```js
function supportsNativeJSON() {
    return typeof JSON !== "undefined" &&
        Object.prototype.toString.call(JSON) === "[object JSON]";
}
```

Here, the same characteristic that allowed developers to identify arrays across iframe boundaries also provided a way to tell if `JSON` was the native one or not. A non-native `JSON` object would return `[object Object]` while the native version returned `[object JSON]`. From that point on, this approach became the de facto standard for identifying native objects.

ECMAScript 6 explains this behavior through the `@@toStringTag` symbol. This symbol represents a property on each object that defines what value should be produced when `Object.prototype.toString.call()` is called on it. So the value returned for arrays is explained by having the `@@toStringTag` property equal `"Array"`. Likewise, you can define that value for your own objects:

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

var me = new Person("Nicholas");

console.log(me.toString());                         // "[object Person]"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

In this example, a `@@toStringTag` property is defined on `Person.prototype` to provide the default behavior for creating a string representation. Since `Person.prototype` inherits `Object.prototype.toString()`, the value returned from `@@toStringTag` is also used when calling `me.toString()`. However, you can still define your own `toString()` that provides a different behavior without affecting the use of `Object.prototype.toString.call()`:

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

Person.prototype.toString = function() {
    return this.name;
};

var me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

This code defines `Person.prototype.toString()` to return the value of the `name` property. Since `Person` instances no longer inherit `Object.prototype.toString()`, calling `me.toString()` exhibits a different behavior.

I> All objects inherit `@@toStringTag` from `Object.prototype` unless otherwise specified. The default property value is `"Object"`.

There is no restriction on which values can be used for `@@toStringTag` on developer-defined objects. For example, there's nothing preventing you from using `"Array"` as the value of `@@toStringTag`, such as:

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Array";

Person.prototype.toString = function() {
    return this.name;
};

var me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Array]"
```

Here, the result of calling `Object.prototype.toString()` is `"[object Array]"`, which is the same as you would get from an actual array. This highlights the fact that `Object.prototype.toString()` is no longer a completely reliable way of identifying an object's type.

It's possible to change the string tag for native objects by assigning to `@@toStringTag` on their prototype. For example:

```js
Array.prototype[Symbol.toStringTag] = "Magic";

var values = [];

console.log(Object.prototype.toString.call(values));    // "[object Magic]"
```

Even though `@@toStringTag` is overwritten for arrays in this example, the call to `Object.prototype.toString()` results in `"[object Magic]"`. While it's recommended not to change built-in objects in this way, there's nothing in the language that forbids it.

### @@unscopables

The `with` statement has long been one of the most controversial parts of JavaScript. Originally designed to avoid repetitive typing, the `with` statement later became roundly criticized for making code harder to understand and for negative performance implications. As a result, the `with` statement is not allowed in strict mode (which also affects classes and modules, which are strict mode by default without an opt-out). While there is no future for the `with` statement, ECMAScript 6 still supports `with` in nonstrict mode and, as such, had to find ways to allow code to continue to work properly using `with`.

To understand the complexity of this task, consider the following code:

```js
var values = [1, 2, 3],
    colors = ["red", "green", "blue"],
    color = "black";

with(colors) {
    push(color);
    push(...values);
}

console.log(colors);    // ["red", "green", "blue", "black", 1, 2, 3]
```

In this example, the two calls to `push()` inside the `with` statement are equivalent to `colors.push()` because the `with` statement added `push` as a local binding. The `color` reference refers to the variable created outside of `with`, as does the `values` reference.

For ECMAScript 6, a `values` method was added to arrays (discussed in Chapter 7 - Iterators and Generators). That would mean the `values` reference in the last example should now refer not the local variables `values`, but to the array's `values` method, and that would break the code. This is why the `@@unscopables` symbol exists.

The `@@unscopables` symbol is used on `Array.prototype` to indicate which properties should not create bindings inside of a `with` statement. When present, `@@unscopables` is an object whose keys are the identifiers to omit from `with` statement bindings and whose values are `true` to enforce the block. Here's the default for arrays:

```js
// built into ECMAScript 6 by default
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null), {
    copyWithin: true,
    entries: true,
    fill: true,
    find, true,
    findIndex: true,
    keys: true,
    values: true
});
```

The `@@unscopables` object has a `null` prototype (created by `Object.create(null)`) and contains all of the new array methods in ECMAScript 6 (these methods are covered in detail in Chapter 7 - Iterators and Generators and Chapter 9 - Arrays). Bindings for these methods are not created inside of a `with` statement, allowing old code to continue working without any problem.

In general, you shouldn't need to define `@@unscopables` for your objects unless you use the `with` statement and are making changes to an existing object in your code base.

## Summary

Symbols are a new type of primitive value in JavaScript and are used to create nonenumerable properties that require the symbol to access. While not truly private, these properties are harder to accidentally change or overwrite and are therefore suitable for functionality that needs a level of protection from developers.

You can provide descriptions for symbols that allow for easier identifier of symbol values. There is a global symbol registry that allows for the use of shared symbols in different parts of code by using the same description. In this way, the same symbol can be used for the same reason in multiple places.

Symbols are not returned in methods like `Object.keys()` or `Object.getOwnPropertyNames()`, so a new method, `Object.getOwnPropertySymbols()` was added to allow for retrieval of symbol properties. You can still make changes to symbol properties by using `Object.defineProperty()` and `Object.defineProperties()`.

Well-known symbols define previously internal-only functionality for standard objects and use globally-available symbol constants, such as `Symbol.hasInstance`. These symbols use the prefix `@@` in the specification and allow developers the opportunity to modify standard object behavior in a variety of ways.
