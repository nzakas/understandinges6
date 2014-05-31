# Objects

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

A lot of ECMAScript 6 focused on improving the utility of objects. The focus makes sense given that nearly every value in JavaScript is represented by some type of object. Additionally, the number of objects used in an average Javascript program continues to increase, meaning that developers are writing more objects all the time. With more objects comes the necessity to use them more effectively.

ECMAScript 6 improves objects in a number of ways, from simple syntax to new ways of manipulating and interacting with objects.

## Object Categories

The ECMAScript 6 specification introduced some new terminology to help distinguish between categories of objects. JavaScript has long been littered with a mix of terminology used to describe objects found in the standard as opposed to those that are added by execution environments such as the browser. ECMAScript 6 takes the time to clearly define each category of object, and it's important to understand this terminology to have a good understanding of the language as a whole. The object categories are:

* *Ordinary objects* are objects that have all of the default internal behaviors for objects in JavaScript.
* *Exotic objects* are objects whose internal behavior is different than the default in some way.
* *Standard objects* are objects defined by ECMAScript 6, such as `Array`, `Date`, etc. Standard objects may be ordinary or exotic.
* *Built-in objects* are objects that are present in a JavaScript execution environment when a script begins to execute. All standard objects are built-in objects.

These terms are used throughout the book to explain the various objects defined by ECMAScript 6.

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

In ECMAScript 6, the syntax is made more succinct by eliminating the colon and the `function` keyword. You can then rewrite the previous example as:

```js
var person = {
    name: "Nicholas",
    sayName() {
        console.log(this.name);
    }
};
```

This shorthand syntax creates a method on the `person` object just as the previous example did. There is no difference aside from saving you some keystrokes, so `sayName()` is assigned an anonymous function expression and has all of the same characteristics as the function defined in the previous example.

I> The `name` property of a method created using this shorthand is the name used before the parentheses. In the previous example, the `name` property for `person.sayName()` is `"sayName"`.

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
console.log(person["last name"]);       // "Zakas"
```

Anything you would be inside of square brackets while using bracket notation on object instances will also work for computed property names inside of object literals.

## Object.assign()

One of the most popular patterns for object composition is *mixins*, in which one object receives properties and methods from another object. Many JavaScript libraries have a mixin method similar to this:

```js
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });

    return receiver;
}
```

The `mixin()` function iterates over the own properties of `supplier` and copies them onto `receiver`. This allows the `receiver` to gain new behaviors without inheritance. For example:

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
}

var myObject = {}
mixin(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

In this example, `myObject` receives behavior from `EventTarget.prototype`. This gives `myObject` the ability to publish events and let others subscribe to them using `emit()` and `on()`, respectively.

This pattern became popular enough that ECMAScript 6 added `Object.assign()`, which behaves the same way. The difference in name is to reflect the actual operation that occurs. Since the `mixin()` method uses the assignment operator (`=`), it cannot copy accessor properties to the receiver as accessor properties. The name `Object.assign()` was chosen to reflect this distinction.

You can use `Object.assign()` anywhere the `mixin()` function would have been used:

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
}

var myObject = {}
Object.assign(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

The `Object.assign()` method accepts any number of suppliers, and the receiver receives the properties in the order in which the suppliers are specified. That means the second supplier might overwrite a value from the first supplier on the receiver. For example:

```js
var receiver = {};

Object.assign(receiver, {
        type: "js",
        name: "file.js"
    }, {
        type: "css"
    }
);

console.log(receiver.type);     // "css"
console.log(receiver.name);     // "file.js"
```

The value of `receiver.type` is `"css"` because the second supplier overwrote the value of the first.

The `Object.assign()` method isn't a big addition to ECMAScript 6, but it does formalize a common function that is found in many JavaScript libraries.


## __proto__, Object.setPrototypeOf()

TODO

## super

toMethod()

TODO

## Reflection Methods

TODO

### Object.getOwnPropertyDescriptors()

TODO

### Object.getPropertyNames()

TODO

### Object.getPropertyDescriptor()

TODO

## Summary

TODO
