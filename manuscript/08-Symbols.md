# Symbols

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

ECMAScript 6 symbols began as a way to create private object members, a feature JavaScript developers have long wanted. The focus was around creating properties that were not identified by string names. Any property with a string name was easy picking to access regardless of the obscurity of the name. The initial "private names" feature aimed to create non-string property names. That way, normal techniques for detecting these private names wouldn't work.

The private names proposal eventually evolved into ECMAScript 6 symbols. While the implementation details remained the same (non-string values for property identifiers), TC-39 dropped the requirement that these properties be private. Instead, the properties would be categorized separately, being non-enumerable by default but still discoverable.

Symbols are actually a new kind of primitive value, joining strings, numbers, booleans, `null`, and `undefined`. They are unique among JavaScript primitives in that they do not have a literal form. The ECMAScript 6 standard uses a special notation to indicate symbols, prefixing the identifier with `@@`, such as `@@create`. This book uses this same convention for ease of understanding.

W> Despite the notation, symbols do not exactly map to strings beginning with "@@". Don't try to use string values where symbols are required.

## Creating Symbols

You can create a symbol by using the `Symbol` function, such as:

```js
var firstName = Symbol();
var person = {};

person[firstName] = "Nicholas";
console.log(person[firstName]);     // "Nicholas"
```

In this example, the symbol `firstName` is created and used to assign a new property on `person`. That symbol must be used each time you want to access that same property. It's a good idea to name the symbol variable appropriately so you can easily tell what the symbol represents.

W> Because symbols are primitive values, `new Symbol()` throws an error when called. It's not possible to create an instance of `Symbol`, which also differentiates it from `String`, `Number`, and `Boolean`.

The `Symbol` function accepts an optional argument that is the description of the symbol. The description itself cannot be used to access the property but is used for debugging purposes. For example:

```js
var firstName = Symbol("first name");
var person = {};

person[firstName] = "Nicholas";

console.log("first name" in person);        // false
console.log(person[firstName]);             // "Nicholas"
console.log(firstName);                     // "Symbol(first name)"
```

A symbol's description is stored internally in a property called `[[Description]]`. This property is read whenever the symbol's `toString()` method is called either explicitly or implicitly (as in this example). It is not otherwise possible to access `[[Description]]` directly from code. It's recommended to always provide a description to make both reading and debugging code using symbols easier.

## Identifying Symbols

Since symbols are primitive values, you can use the `typeof` operator to identify them. ECMAScript 6 extends `typeof` to return `"symbol"` when used on a symbol. For example:

```js
var symbol = Symbol("test symbol");
console.log(typeof symbol);         // "symbol"
```

While there are other indirect ways of determining whether a variable is a symbol, `typeof` is the most accurate and preferred way of doing so.

## Using Symbols

You can use symbols anywhere you would use a computed property name. You've already seen the use of bracket notation in the previous sections, but you can use symbols in computed object literal property names as well as with `Object.defineProperty()`, and `Object.defineProperties()`, such as:

```js
var firstName = Symbol("first name");
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

With computed property names in object literals, symbols are very easy to work with.

## Sharing Symbols

You may find that you want different parts of your code to use the same symbols. For example, suppose you have two different object types in your application that should use the same symbol property to represent a unique identifier. Keeping track of symbols across files or large codebases can be difficult and error-prone. That's why ECMAScript 6 provides a global symbol registry that you can access at any point in time.

When you want to create a symbol to be shared, use the `Symbol.for()` method instead of calling `Symbol()`. The `Symbol.for()` method accepts a single parameter, which is a string identifier for the symbol you want to create (this value doubles as the description). For example:

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

## Finding Object Symbols

Similar to other properties on objects, you can access symbol properties using the `Object.getOwnPropertySymbols()` method. This method works exactly the same as `Object.getOwnPropertyNames()` except that the returned values are symbols rather than strings. Since symbols technically aren't property names, they are omitted from the result of `Object.getOwnPropertyNames()`.

The return value of `Object.getOwnPropertySymbols()` is an array of symbols that represent own properties. For example:

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

In this code, `object` has a single symbol property. The array returned from `Object.getOwnPropertySymbols()` is an array containing just that symbol.

I> All objects start off with zero own symbol properties (although they do have some inherited symbol properties).

## Well-Known Symbols

In addition to the symbols you defined, there are some predefined symbols as well (called *well-known* symbols in the specification). These symbols represent common behaviors in JavaScript that were previously considered internal-only operations. Each well-known symbol is represented by a property on `Symbol`, such as `Symbol.create` for the `@@create` symbol.

A central theme for both ECMAScript 5 and ECMAScript 6 was exposing and defining some of the "magic" parts of JavaScript - the parts that couldn't be emulated by a developer. ECMAScript 6 follows this tradition by exposing even more of the previously internal-only logic of the language. It does so primarily through the use of symbol prototype properties that define the basic behavior of certain objects.

I> Overwriting a method defined with a well-known symbol changes an ordinary object to an exotic object because this changes some internal default behavior.

### @@toStringTag

Once of the most interesting problems in JavaScript has been the availability of multiple global execution environments. This occurs in web browsers when a page includes an iframe, as the page and the iframe each have their own execution environments. In most cases, this isn't a problem, as data can be passed back and forth between the environments with little cause for concern. The problem arises when trying to identify what type of an object you're dealing with.

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

ECMAScript 6 explains this behavior through the `@@toStringTag` symbol. This symbol represents a method on each object that defines what value should be produced when `Object.prototype.toString.call()` is called on it. So the value returned for arrays is explained by having the `@@toStringTag` method return `"Array"`. Likewise, you can define that value for your own objects:

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = function() {
    return "Person";
};

var me = new Person("Nicholas");

console.log(me.toString());                         // "[object Person]"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

In this example, a `@@toStringTag` method is defined on `Person.prototype` to provide the default behavior for creating a string representation. Since `Person.prototype` inherits `Object.prototype.toString()`, the value returned from `@@toStringTag` is also used when calling `me.toString()`. However, you can still define your own `toString()` that provides a different behavior without affecting the use of `Object.prototype.toString.call()`:

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = function() {
    return "Person";
};

Person.prototype.toString = function() {
    return this.name;
};

var me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

This code defines `Person.prototype.toString()` to return the value of the `name` property. Since `Person` instances no longer inherit `Object.prototype.toString()`, calling `me.toString()` exhibits a different behavior.

I> All objects inherit `@@toStringTag` from `Object.prototype` unless otherwise specified. This default method returns `"Object"`.

### @@toPrimitive

JavaScript frequently attempts to convert objects into primitive values implicitly when certain operations are applied. For instance, when you compare a string to an object using double equals (`==`), the object is converted into a primitive value before comparing. Exactly what value should be used was previously an internal operation that is exposed in ECMAScript 6 through the `@@toPrimitive` method.

The `@@toPrimitive` method is defined on the prototype of each standard type and prescribes the exact behavior. When a primitive conversion is needed, `@@toPrimitive` is called with a single argument, referred to as `hint` in the specification. The `hint` argument is `"default"`, specifying that the operation has no preference as to the type, `"string"`, indicating a string should be returned, or `"number"`, if a number is necessary to perform the operation. Most standard objects treat `"default"` as equivalent to `"number"` (except for `Date`, which treats `"default"` as `"string"`).

TODO

### @@isConcatSpreadable

TODO

### @@species

TODO

### @@unscopeables

TODO

Only applied to `with` statement object records - does not refer to other scopes.

### @@iterator

As you learned in chapter 6, iterators are an important part of ECMAScript 6, and many built-in types have default iterators defined. These default iterators are defined by the `@@iterator` symbol on their prototype. The `for-of` loop uses an object's `@@iterator` property to retrieve an iterator when the given value isn't already one. You can get a reference to the default iterator for arrays using `Symbol.iterator`, such as:

```js
// retrieve the default generator for arrays
let arrayGenerator = Array.prototype[Symbol.iterator];

console.log(typeof arrayGenerator);                     // "function"
console.log(Array.prototype.values === arrayGenerator); // true
```

In this example, `arrayGenerator` is a generator function used to create the default array iterator. This generator is the same one that is used for `Array.prototype.values`.

The `@@iterator` symbol can also be used to define the default iterator for your objects. For example:

```js
let collection = {
    items: [],
    *[Symbol.iterator]() {
        yield *this.items.values();
    }

};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}

// Output:
// 1
// 2
// 3
```

This code defines a default iterator for a variable called `collection` using object literal method shorthand and a computed property using `Symbol.iterator`. The generator then delegates to the `values()` iterator of `this.items`. The `for-of` loop then uses the generator to create an iterator and execute the loop.

You can also define a default iterator using classes, such as:

```js
class Collection {

    constructor() {
        this.items = [];
    }

    *[Symbol.iterator]() {
        yield *this.items.values();
    }
}

var collection = new Collection();
collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}

// Output:
// 1
// 2
// 3
```

This example mirrors the previous one with the exception that a class is used instead of an object literal.

Default iterators can be added to any object by assigning a generator to `Symbol.iterator`. It doesn't matter if the property is an own or prototype property, as `for-of` normal prototype chain lookup applies.

### @@hasInstance

TODO

## Summary

TODO
