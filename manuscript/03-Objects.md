# Objects

A lot of ECMAScript 6 focused on improving the utility of objects. The focus makes sense given that nearly every value in JavaScript is represented by some type of object. Additionally, the number of objects used in an average JavaScript program continues to increase, meaning that developers are writing more objects all the time. With more objects comes the necessity to use them more effectively.

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

Anything you would put inside of square brackets while using bracket notation on object instances will also work for computed property names inside of object literals.

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
};

var myObject = {};
mixin(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

In this example, `myObject` receives behavior from `EventTarget.prototype`. This gives `myObject` the ability to publish events and let others subscribe to them using `emit()` and `on()`, respectively.

This pattern became popular enough that ECMAScript 6 added `Object.assign()`, which behaves the same way. The difference in name is to reflect the actual operation that occurs. Since the `mixin()` method uses the assignment operator (`=`), it cannot copy accessor properties to the receiver as accessor properties. The name `Object.assign()` was chosen to reflect this distinction.

I> Similar methods in various libraries may have other names. Some popular alternate names for the same basic functionality are `extend()` and `mix()`.

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

A> ### Working with Accessor Properties
A>
A> Keep in mind that you cannot create accessor properties on the receiver by using a supplier with accessor properties. Since `Object.assign()` uses the assignment operator, an accessor property on a supplier will become a data property on the receiver. For example:
A>
A> ```js
A> var receiver = {},
A>     supplier = {
A>         get name() {
A>             return "file.js"
A>         }
A>     };
A>
A> Object.assign(receiver, supplier);
A>
A> var descriptor = Object.getOwnPropertyDescriptor(receiver, "name");
A>
A> console.log(descriptor.value);      // "file.js"
A> console.log(descriptor.get);        // undefined
A> ```
A>
A> In this code, the `supplier` has an accessor property called `name`. After using `Object.assign()`, `receiver.name` exists as a data property with the value of `"file.js"`. That's because `supplier.name` returned "file.js" at the time `Object.assign()` was called.

## Duplicate Object Literal Properties

ECMAScript 5 strict mode introduced a check for duplicate object literal properties that would throw an error if a duplicate was found. For example:

```js
var person = {
    name: "Nicholas",
    name: "Greg"        // syntax error in ES5 strict mode
};
```

When running in ECMAScript 5 strict mode, this example results in a syntax error on the second `name` property.

In ECMAScript 6, the duplicate property check has been removed. Both strict and nonstrict mode code no longer check for duplicate properties and instead take the last property of the given name as the actual value.

```js
var person = {
    name: "Nicholas",
    name: "Greg"        // not an error in ES6
};

console.log(person.name);       // "Greg"
```

In this example, the value of `person.name` is `"Greg"` because that was the last value assigned to the property.

## Changing Prototypes

Prototypes are the foundation of inheritance and JavaScript and so ECMAScript 6 continues to make prototypes more powerful. ECMAScript 5 added the `Object.getPrototypeOf()` method for retrieving the prototype of any given object. ECMAScript 6 adds the reverse operation, `Object.setPrototypeOf()`, which allows you to change the prototype of any given object.

Normally, the prototype of an object is specified at the time of its creation, either by using a constructor or via `Object.create()`. Prior to ECMAScript 6, there was no standard way to change an object's prototype after it had already been created. In a way, `Object.setPrototypeOf()` changes one of the biggest assumptions about objects in JavaScript to this point, which is that an object's prototype remains unchanged after creation.

The `Object.setPrototypeOf()` method accepts two arguments, the object whose prototype should be changed and the object that should become the first argument's prototype. For example:

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// prototype is person
let friend = Object.create(person);
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true

// set prototype to dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

This code defines two base objects: `person` and `dog`. Both objects have a method `getGreeting()` that outputs something to the console. The object `friend` starts out inheriting from `person`, meaning that `getGreeting()` will output `"Hello"`. When the prototype is changed to be `dog` instead, `person.getGreeting()` outputs `"Woof"` because the original relationship to `person` is broken.

The actual value of an object's prototype is stored in an internal-only property called `[[Prototype]]`. The `Object.getPrototypeOf()` method returns the value stored in `[[Prototype]]` and `Object.setPrototypeOf()` changes the value stored in `[[Prototype]]`. However, these aren't the only ways to work with the value of `[[Prototype]]`.

Even before ECMAScript 5 was finished, several JavaScript engines already implemented a custom property called `__proto__` that could be used to both get and set the prototype of an object. Effectively, `__proto__` was an early precursor to both `Object.getPrototypeOf()` and `Object.setPrototypeOf()`. It was unrealistic to expect all of the JavaScript engines to remove this property, so ECMAScript 6 formalized the behavior of `__proto__`.

In ECMAScript 6 engines, `Object.prototype.__proto__` is defined as an accessor property whose `get` method calls `Object.getPrototypeOf()` and whose `set` method calls `Object.setPrototypeOf()`. This means that there is no real difference between using `__proto__` and the other methods except that `__proto__` allows you to set the prototype of an object literal directly. For example:

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// prototype is person
let friend = {
    __proto__: person
};
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true
console.log(friend.__proto__ === person);               // true

// set prototype to dog
friend.__proto__ = dog;
console.log(friend.getGreeting());                      // "Woof"
console.log(friend.__proto__ === dog);                  // true
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

This example is functionally equivalent to the previous. The call to `Object.create()` has been replaced with an object literal that assigns a value to `__proto__`. The only real difference between creating an object using `Object.create()` or an object literal with `__proto__` is that the former requires you to specify full property descriptors for any additional object properties while the latter is just a standard object literal.

W> The `__proto__` property is special in a number of ways:
W>
W> 1. You can only specify it once in an object literal. If you specify two `__proto__` properties, then an error is thrown. This is the only object literal property that has this restriction.
W> 1. The computed form `["__proto__"]` acts like a regular property and does not set or return the current object's prototype. All rules related to object literal properties apply in this form, as opposed to the non-computed form, for which there are exceptions.
W>
W> It's best to be careful when using `__proto__` to make sure you don't get caught by these differences.

## Super References

As previously mentioned, prototypes are very important for JavaScript and a lot of work went into making them easier to use in ECMAScript 6. Among the improvements is the introduction of `super` references to more easily access functionality on an object's prototype. For example, if you want to override a method on an object instance such that it also calls the prototype method of the same name, you would need to do the following in ECMAScript 5:

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// prototype is person
let friend = {
    __proto__: person,
    getGreeting() {
        // same as this.__proto__.getGreeting.call(this)
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};

console.log(friend.getGreeting());                      // "Hello, hi!"
console.log(Object.getPrototypeOf(friend) === person);  // true
console.log(friend.__proto__ === person);               // true

// set prototype to dog
friend.__proto__ = dog;
console.log(friend.getGreeting());                      // "Woof, hi!"
console.log(friend.__proto__ === dog);                  // true
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

In this example, `getGreeting()` on `friend` calls the prototype method of the same name. The `Object.getPrototypeOf()` method is used to ensure the method is always getting the accurate prototype and then an additional string is appended. The additional `.call(this)` ensures that the `this` value inside the prototype method is set correctly.

Needing to remember to use `Object.getPrototypeOf()` and `.call(this)` to call a method on the prototype is a bit involved, so ECMAScript 6 introduced `super`.

At it's simplest, `super` acts as a pointer to the current object's prototype, effectively acting like `Object.getPrototypeOf(this)`. So you can simplify the `getGreeting()` method by rewriting it as:

```js
let friend = {
    __proto__: person,
    getGreeting() {
        // same as Object.getPrototypeOf(this).getGreeting.call(this)
        // or this.__proto__.getGreeting.call(this)
        return super.getGreeting() + ", hi!";
    }
};
```

The call to `super.getGreeting()` is the same as `Object.getPrototypeOf(this).getGreeting.call(this)` or `this.__proto__.getGreeting.call(this)`. Similarly, you can call any method on an object's prototype by using a `super` reference.

If you're calling a prototype method with the exact same name, then you can also call `super` as a function, for example:

```js
let friend = {
    __proto__: person,
    getGreeting() {
        // same as Object.getPrototypeOf(this).getGreeting.call(this)
        // or this.__proto__.getGreeting.call(this)
        // or super.getGreeting()
        return super() + ", hi!";
    }
};
```

Calling `super` in this manner tells the JavaScript engine that you want to use the prototype method with the same name as the current method. So `super()` actually does a lookup using the containing function's `name` property (discussed in Chapter 2) to find the correct method.

W> `super` references can only be used inside of functions and cannot be used in the global scope. Attempting to use `super` in the global scope results in a syntax error.

### Methods

Prior to ECMAScript 6, there was no formal definition of a "method" - methods were just object properties that contained functions instead of data. ECMAScript 6 formally defines a method as a function that has an internal `[[HomeObject]]` property containing the object to which the method belongs. Consider the following:

```js
let person = {

    // method
    getGreeting() {
        return "Hello";
    }
};

// not a method
function shareGreeting() {
    return "Hi!";
}
```

This example defines `person` with a single method called `getGreeting()`. The `[[HomeObject]]` for `getGreeting()` is `person` by virtue of assigning the function directly to an object. The `shareGreeting()` function, on the other hand, has no `[[HomeObject]]` specified because it wasn't assigned to an object when it was created. In most cases this difference isn't important, but it becomes very important when using `super`.

Any reference to `super` uses the `[[HomeObject]]` to determine what to do. The first step is to call `Object.getPrototypeOf()` on the `[[HomeObject]]` to retrieve a reference to the prototype. Then, the prototype is searched for a function with the same name as the executing function. Last, the `this`-binding is set and the method is called. If a function has no `[[HomeObject]]`, or has a different one than expected, then this process won't work. For example:

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// prototype is person
let friend = {
    __proto__: person,
    getGreeting() {
        return super() + ", hi!";
    }
};

function getGlobalGreeting() {
    return super.getGreeting() + ", yo!";
}

console.log(friend.getGreeting());  // "Hello, hi!"

getGlobalGreeting();                      // throws error
```

Calling `friend.getGreeting()` returns a string while calling `getGlobalGreeting()` throws an error for improper use of `super`. Since the `getGlobalGreeting()` function has no `[[HomeObject]]`, it's not possible to perform a lookup. Interestingly, the situation doesn't change if `getGlobalGreeting()` is later assigned as a method on `friend`:

```js
// prototype is person
let friend = {
    __proto__: person,
    getGreeting() {
        return super() + ", hi!";
    }
};

function getGlobalGreeting() {
    return super.getGreeting() + ", yo!";
}

console.log(friend.getGreeting());  // "Hello, hi!"

// assign getGreeting to the global function
friend.getGreeting = getGlobalGreeting;

friend.getGreeting();               // throws error
```

Here the global `getGlobalGreeting()` function is used to overwrite the previously-defined `getGreeting()` method on `friend`. Calling `friend.getGreeting()` at that point results in an error as well. The value of `[[HomeObject]]` is only set when the function is first created, so even assigning onto an object doesn't fix the problem.

## Summary

Objects are at the center of programming in JavaScript, and ECMAScript 6 has made some helpful changes to objects that both make them easier to deal with and more powerful.

ECMAScript 6 makes several changes to object literals. Shorthand property definitions make it easier to assign properties whose names are the same as in-scope variables. Computed property names allow you to specify non-literal values as property names, which is something you've already been able to do in other areas of the language. Shorthand methods let you type a lot fewer characters in order to define methods on object literals by completely omitting the `function` keyword and colon. A loosening of the strict mode check for duplicate object literal property names was introduced as well, meaning you can now have two properties with the same name in a single object literal without an error being thrown.

The `Object.assign()` method makes it easier to change multiple properties on a single object at once. This can be very useful if you use the mixin pattern.

It's now possible to modify an object's prototype after it's already created using `Object.setPrototypeOf()`. ECMAScript 6 also defines the behavior of the `__proto__` property, which is an accessor property whose getter calls `Object.getPrototypeOf()` and whose setter calls `Object.setPrototypeOf()`.

The `super` keyword can now be used to call methods on an object's prototype. It can be used either standalone as a method, such as `super()`, or as a reference to the prototype itself, such as `super.getGreeting()`. In both cases, the `this`-binding is setup automatically to work with the current value of `this`. You can change how `super` is evaluated inside of a function by calling `toMethod()` and specifying a new `[[HomeObject]]`. This method returns a new copy of the function whose `[[HomeObject]]` is the argument that was passed in.
