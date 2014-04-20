# Objects

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

A lot of ECMAScript 6 focused on improving the utility of objects. The focus makes sense given that nearly every value in JavaScript is represented by some type of object. Additionally, the number of objects used in an average Javascript program continues to increase, meaning that developers are writing more objects all the time. With more objects comes the necessity to use them more effectively.

ECMAScript 6 improves objects in a number of ways, from simple syntax to new ways of manipulating and interacting with objects.

## Object Literal Extensions

One of the most popular patterns in JavaScript is the object literal. It's the syntax upon which JSON is built and can be seen in nearly every JavaScript file on the Internet. The reason for the popularity is clear: a succinct syntax for creating objects that otherwise would take several lines of code to accomplish. ECMAScript 6 recognized the popularity of the object literal and extends the syntax in several ways to make object literals more powerful and even more succinct.

### Property Initializer Shorthand

In ECMAScript 5 and earlier, object literals were simply collections of name-value pairs. That meant there could be some duplication when property values are being initialized. For example:

```js
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
}
```

The `createPerson()` function creates an object whose property names are the same as the function parameter names. The result is what appears to be duplication of `name` and `age` even though each represents a different aspect of the process.

In ECMAScript 6, you can eliminate the duplication that exists around property names and local variables by using the property initializer shorthand. When the property name is going to be the same as the local variable name, you can simply include the name without a colon and value. For example, `createPerson()` can be rewritten as follows:

```js
function createPerson(name, age) {
    return {
        name,
        age
    };
}
```

When a property in an object literal only has a name and no value, the JavaScript engine looks into the surrounding scope for a variable of the same name. If found, that value is assigned to the same name on the object literal. So in this example, the object literal property `name` is assigned the value of the local variable `name`.

The purpose of this extension is to make object literal initialization even more succinct than it already was. Assigning a property with the same name as a local variable is a very common pattern in JavaScript and so this extension is a welcome addition.

### Method Initializer Shorthand

ECMAScript 6 also improves syntax for assigning methods to object literals. In ECMAScript 5 and earlier, you must specify a name and then the full function definition to add a method to an object. For example:

```js
var person = {
    name: "Nicholas",
    sayName: function() {
        console.log(this.name);
    }
};
```

In ECMAScript 6, the syntax is made more sustained by eliminating the colon and the `function` keyword. you can then rewrite this example as follows:

```js
var person = {
    name: "Nicholas",
    sayName() {
        console.log(this.name);
    }
};
```

This shorthand syntax creates a method on the `person` object just as the previous example did. There is no difference aside from saving you some keystrokes.

### Computed Property Names

JavaScript objects have long had computed property names through the use of square brackets instead of dot notation. The square brackets allow you to specify property names using variables and string literals that may contain characters that would be a syntax error if used in an identifier. For example:

```js
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

Both of the property names in this example have a space, making it impossible to reference those names using dot notation. However, bracket notation allows any string value to be used as a property name.

In ECMAScript 5, you could use string literals as property names in object literals, such as:

```js
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas"
```

If you could provide the string literal inside of the object literal property definition then you were all set. If, however, the property name was contained in a variable or had to be calculated, then there was no way to define that property using an object literal.

ECMAScript 6 adds computed property names to object literal syntax by using the same square bracket notation that has been used to reference computed property names in object instances. For example:

```js
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

The square brackets inside of the object literal indicate that the property name is computed, so its contents are evaluated as a string. That means you can also include expressions such as:

```js
var suffix = " name";

var person = {
    ["first" + suffix]: "Nicholas",
    ["last" + suffix]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person["last name"]);          // "Zakas"
```

Anything you would be inside of square brackets while using bracket notation on object instances will also work for computed property names inside of object literals.

## Symbols

ECMAScript 6 symbols began as a way to create private object members, a feature JavaScript developers have long wanted. The focus was around creating properties that were not identified by string names. Any property with a string name was easy picking to access regardless of the obscurity of the name. The initial "private names" feature aimed to create non-string property names. That way, normal techniques for detecting these private names wouldn't work.

The private names proposal eventually evolved into ECMAScript 6 symbols. While the implementation details remained the same (non-string values for property identifiers), TC-39 dropped the requirement that these properties be private. Instead, the properties would be categorized separately, being non-enumerable by default by still discoverable.

Symbols are actually a new kind of primitive value, joining strings, numbers, booleans, `null`, and `undefined`. They are unique among JavaScript primitives in that they do not have a literal form. The ECMAScript 6 standard uses a special notation to indicate symbols, prefixing the identifier with `@@`, such as `@@create`. This book uses this same convention for ease of understanding.

W> Despite the notation, symbols do not exactly map to strings beginning with "@@". Don't try to use string values where symbols are required.

### Creating Symbols

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

### Identifying Symbols

Since symbols are primitive values, you can use the `typeof` operator to identify them. ECMAScript 6 extends `typeof` to return `"symbol"` when used on a symbol. For example:

```js
var symbol = Symbol("test symbol");
console.log(typeof symbol);         // "symbol"
```

While there are other indirect ways of determining whether a variable is a symbol, `typeof` is the most accurate and preferred way of doing so.

### Using Symbols

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

With computer property names in object literals, symbols are very easy to work with.

### Sharing Symbols

You may find that you want different parts of your code to use the same symbols. For example, suppose you have two different object types in your application that should use the same symbol property to represent a unique identifier. Keeping track of symbols across files or large codebases can be difficult and error-prone. That's why ECMAScript 6 provides a global symbol registry that you can access at any point in time.

When you want to create a symbol to be shared, use the `Symbol.for()` method instead of calling `Symbol()`. The `Symbol.for()` method accepts a single parameter, which is a string identifier for the symbol you want to create (this value doubles as the description). For example:

```js
var uid = Symbol.for("uid");
var object = {};

object[uid] = "12345";

console.log(object[uid]);       // "Nicholas"
console.log(uid);               // "Symbol(uid)"
```

The `Symbol.for()` method first searches the global symbol registry to see if a symbol with the key `"uid"` exists. If so, then it returns the already existing symbol. If no such symbol exists, then a new symbol is created and registered into the global symbol registry using the specified key. The new symbol is then returned. That means subsequent calls to `Symbol.for()` using the same key will return the same symbol:

```js
var uid = Symbol.for("uid");
var object = {};

object[uid] = "12345";

console.log(object[uid]);       // "Nicholas"
console.log(uid);               // "Symbol(uid)"

var uid2 = Symbol.for("uid");

console.log(uid === uid2);      // true
console.log(object[uid2]);      // "Nicholas"
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

Notice that both `uid` and `uid2` return the key `"uid"`. The symbol `uid3` doesn't exist in the global symbol registry, so it has no key associated with it and so `Symbol.keyFor()` returns `undefined`.

W> The global symbol registry is a shared environment, just like the global scope. That means you can't make assumptions about what is or is not already present in that environment. You should use namespacing of symbol keys to reduce the likelihood of naming collisions when using third-party components. For example, jQuery might prefix all keys with `"jquery."`, such as `"jquery.element"`.

### Finding Object Symbols

TODO

## Object Destructuring

TODO

## Object.assign()

TODO

## ~~__proto__~~ Object.setPrototypeOf()

TODO

## super

TODO
