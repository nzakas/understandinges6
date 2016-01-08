# Classes

Ever since JavaScript was first created, many developers have been confused by its lack of classes. Most formal object-oriented programming languages support classes and classical inheritance as the primary way of defining similar and related objects. From pre-ECMAScript 1 all the way through ECMAScript 5, this point of confusion led many libraries to create utilities designed to make JavaScript look like it had support for classes.

While there are some JavaScript developers who feel strongly that the language doesn't need classes, the fact that so many libraries were created specifically for this purpose led to the inclusion of classes in ECMAScript 6. However, ECMAScript 6 classes aren't exactly the same as classes in other languages. There's a uniqueness about them that embraces the dynamic nature of JavaScript.

## Class-Like Structures in ECMAScript 5

Before exploring classes, it's helpful to understand the underlying mechanisms that classes use. In ECMAScript 5 and earlier, there were no classes, and the closest equivalent was creating a constructor and then assigning methods to its prototype. This approach is called creating a custom type. For example:

```js
function PersonType(name) {
    this.name = name;
}

PersonType.prototype.sayName = function() {
    console.log(this.name);
};

let person = new PersonType("Nicholas");
person.sayName();   // outputs "Nicholas"

console.log(person instanceof PersonType);  // true
console.log(person instanceof Object);      // true
```

In this code, `PersonType` is a constructor function that creates a single property called `name`. The `sayName()` method is assigned to the prototype so the same function is shared by all instances of `PersonType`. Then, a new instance of `PersonType` is created via the `new` operator, and the resulting `person` object is considered an instance of `PersonType` and of `Object` (through prototypal inheritance).

This same basic pattern underlies a lot of the class-mimicking JavaScript libraries. And that's where ECMAScript 6 classes start.

## Class Declarations

The simplest class form is the one that looks similar to other languages: the class declaration. Class declarations begin with the `class` keyword followed by the name of the class. The rest of the syntax looks similar to concise methods in object literals without requiring commas between them. For example, here's the class equivalent of the previous example:

```js
class PersonClass {

    // equivalent of the PersonType constructor
    constructor(name) {
        this.name = name;
    }

    // equivalent of PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
}

let person = new PersonClass("Nicholas");
person.sayName();   // outputs "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function"
```

The class declaration `PersonClass` behaves quite similarly to `PersonType` from the previous example. Instead of defining a function as the constructor, class declarations allow you to define the constructor directly inside of the class using the special `constructor` method name. Since class methods use the concise syntax, there's no need to use the `function` keyword. All other method names have no special meaning, so you can add as many as you want.

I> Own properties, properties that occur on the instance rather than the prototype, can only be created inside of a class constructor or method. In the previous example, `name` is an own property. It's recommended to create all possible own properties inside of the constructor function so there's a single place that's responsible for all of them.

An interesting aspect of class declarations is that they are just syntactic sugar on top of the existing custom type declarations. The `PersonClass` declaration actually creates a function that has the behavior of the `constructor` method, which is why `typeof PersonClass` is `"function"`. Similarly, the `sayName()` method ends up as a method on `PersonClass.prototype`, similar to `PersonType.prototype` in the earlier example. These similarities allow you to mix custom types and classes without worrying too much about which you're using.

Despite the similarities, there are some important differences to keep in mind:

1. Class declarations, unlike function declarations, are not hoisted. Class declarations act like `let` declarations and so exist in the temporal dead zone until execution reaches the declaration.
1. All code inside of class declarations runs in strict mode automatically. There's no way to opt-out of strict mode inside of classes.
1. All methods are non-enumerable. This is a significant change from custom types, where you need to use `Object.defineProperty()` to make a method non-enumerable.
1. All methods have no `[[Construct]]` internal method and so throw an error if you try to call them with `new`.
1. Calling the class constructor without `new` throws an error.
1. Attempting to overwrite the class name within a class method throws an error.

With all of this in mind, the `PersonClass` declaration from the previous example is directly equivalent to the following:

```js
// direct equivalent of PersonClass
let PersonType2 = (function() {

    "use strict";

    const PersonType2 = function(name) {

        // make sure the function was called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonType2.prototype, "sayName", {
        value: function() {

            // make sure the method wasn't called with new
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new.");
            }

            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonType2;
}());
```

The first thing to notice in this code is that there are two `PersonType2` declarations, a `let` declaration in the outer scope and a `const` declaration inside of the IIFE. This is how class methods are forbidden from overwriting the class name while code outside of the class is allowed to do so. The constructor function checks `new.target` to ensure that it's being called with `new`, otherwise an error is thrown. Next, the `sayName()` method is defined as nonenumerable and it also checks `new.target` to ensure that it wasn't called with `new`. The final step is to return the constructor function.

From this example, you can see that while it is possible to do everything that classes do without using new syntax, the class syntax makes all of the functionality a lot simpler than it would be otherwise.

A> ### Constant Class Names
A>
A> The name of a class is specified as if using `const`, but only inside of the class itself. That means you can overwrite the class name outside of the class but not inside a class method. For example:
A>
A> ```js
A> class Foo {
A>    constructor() {
A>        Foo = "bar";    // throws an error when executed
A>    }
A> }
A>
A>// but this is okay
A> Foo = "baz";
A> ```
A>
A> In this code, the `Foo` inside of the class constructor is a separate binding than the `Foo` outside of the class. The internal `Foo` is defined as if it's a `const` and so cannot be overwritten, which means and error is thrown when an attempt is made to do so; the external `Foo` is defined as if it's a `let` declaration and so its value may be overwritten at any time.

## Class Expressions

Classes and functions are similar in that they have two forms: declarations and expressions. Function and class declarations begin with an appropriate keyword (`function` or `class`, respectively) followed by an identifier. Functions have an expression form that doesn't require an identifier after `function`, and similarly, classes have an expression form that doesn't require an identifier after `class`.

These *class expressions* are designed to be used in variable declarations or passed into functions as arguments. Here's the class expression equivalent of the previous examples:

```js
// class expressions do not require identifiers after "class"
let PersonClass = class {

    // equivalent of the PersonType constructor
    constructor(name) {
        this.name = name;
    }

    // equivalent of PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

let person = new PersonClass("Nicholas");
person.sayName();   // outputs "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function"
```

Aside from the syntax, class expressions are exactly equivalent to class declarations.

I> Whether you use class declarations or class expressions is purely a matter of style. Unlike function declarations and function expressions, both class declarations and class expressions are not hoisted, and so the choice has little bearing on the runtime behavior of the code.

### Named Class Expressions

The previous section used an anonymous class expression in the example, but you can also name class expressions just like you can name function expressions. To do so, include an identifier after the `class` keyword:

```js
let PersonClass = class PersonClass2 {

    // equivalent of the PersonType constructor
    constructor(name) {
        this.name = name;
    }

    // equivalent of PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

console.log(typeof PersonClass);        // "function"
console.log(typeof PersonClass2);       // "undefined"
```

In this example, the class expression is given a name, `PersonClass2`. The `PersonClass2` identifier exists only within the class definition so that it can be used inside of methods. Outside of the class, there is no `PersonClass2` binding, so `typeof PersonClass2` is `"undefined"`. To understand why this is, here's the equivalent declaration without using classes:

```js
// direct equivalent of PersonClass named class expression
let PersonClass = (function() {

    "use strict";

    const PersonClass2 = function(name) {

        // make sure the function was called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonClass2.prototype, "sayName", {
        value: function() {
            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonClass2;
}());
```

Creating a named class expression slightly changes what's happening in the JavaScript engine. For class declarations the outer binding (defined with `let`) has the same name as the inner binding (defined with `const`). Named class expressions use their name as the `const` definition, so in this case, `PersonClass2` is defined for use only inside of the class.

While the behavior of named class expressions is different from that of named function expressions, there are still a lot of similarities between the two. Both can be used as values, and that opens up a lot of possibilities.

## Classes as First-Class Citizens

In programming, something is said to be a *first-class citizen* when it can be used as a value, meaning it can be passed into a function, returned from a function, and assigned to a variable. JavaScript functions are first class citizens, sometimes just called first class functions, and that's part of what makes JavaScript unique. ECMAScript 6 continues this tradition by making classes first-class citizens as well. That allows classes to be used in a lot of different ways. For example, they can be passed into functions as arguments:

```js
function createObject(classDef) {
    return new classDef();
}

let obj = createObject(class {

    sayHi() {
        console.log("Hi!");
    }
});

obj.sayHi();        // "Hi!"
```

In this example, an anonymous class expression is passed into `createObject()`. An instance is then created by using `new` and that object is returned.

Another interesting use of class expressions is to create singletons by immediately invoking the class constructor. To do so, you must use `new` with a class expression and include parentheses at the end. For example:

```js
let person = new class PersonClass {

    constructor(name) {
        this.name = name;
    }

    sayName() {
        console.log(this.name);
    }

}("Nicholas");

person.sayName();       // "Nicholas"
```

Here, the anonymous class expression is created and then executed immediately. This pattern allows you to use the class syntax for creating singletons without leaving a class reference available for inspection (remember that `PersonClass` only creates a binding inside of the class, not outside). The parentheses at the end are the indicator that you're calling a function while also allowing you to pass in an argument.

The examples in this chapter so far have focused on classes with methods. But you can also create accessor properties on classes using a syntax similar to object literals.

## Accessor Properties

While own properties should be created inside of class constructors, classes allow you to define accessor properties on the prototype. To create a getter, use the keyword `get` followed by a space followed by an identifier; to create a setter, do the same using the keyword `set`. For example:

```js
class CustomHTMLElement {

    constructor(element) {
        this.element = element;
    }

    get html() {
        return this.element.innerHTML;
    }

    set html(value) {
        this.element.innerHTML = value;
    }
}

var descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype, "html");
console.log("get" in descriptor);   // true
console.log("set" in descriptor);   // true
console.log(descriptor.enumerable); // false
```

In this example, the `CustomHTMLElement` class is made as a wrapper around an existing DOM element. It has both a getter and setter for `html` that delegates to the `innerHTML` method on the element itself. This accessor property is created as non-enumerable, just like any other method would be, and is created on the `CustomHTMLElement.prototype`. The equivalent non-class representation is:

```js
// direct equivalent to previous example
let CustomHTMLElement = (function() {

    "use strict";

    const CustomHTMLElement = function(element) {

        // make sure the function was called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.element = element;
    }

    Object.defineProperty(CustomHTMLElement.prototype, "html", {
        enumerable: false,
        configurable: true,
        get: function() {
            return this.element.innerHTML;
        },
        set: function(value) {
            this.element.innerHTML = value;
        }
    });

    return CustomHTMLElement;
}());
```

As with previous examples, this one shows just how much code you're saving by using a class instead of the non-class equivalent. The accessor property definition alone is almost the size of the equivalent class declaration.

The similarities between objects literals and classes aren't quite over yet. Class methods and accessor properties can use computed names using the same syntax as object literals.

## Computed Member Names

Class methods and accessor properties can have computed names. Instead of using an identifier, use square brackets around an expression (the same as with object literal computed names). For example:

```js
let methodName = "sayName";

class PersonClass {

    constructor(name) {
        this.name = name;
    }

    [methodName]() {
        console.log(this.name);
    }
};

let me = new Person("Nicholas");
me.sayName();           // "Nicholas"
```

This version of `PersonClass` uses a variable to assign the method name. The `methodName` variable contains the string `"sayName"`, which is then used to declare the method. The `sayName()` method is later accessed directly. Accessor properties can also use computed names in the same way:

```js
let propertyName = "html";

class CustomHTMLElement {

    constructor(element) {
        this.element = element;
    }

    get [propertyName]() {
        return this.element.innerHTML;
    }

    set [propertyName](value) {
        this.element.innerHTML = value;
    }
}
```

Here, the getter and setter for `html` are set using the variable `propertyName`. You can still access the property by using `.html`, it's just the definition that is affected.

You've seen that there are a lot of similarities between classes and object literals, with methods, accessor properties, and computed names. There's just one more similarity to cover: generators.

## Generator Methods

When generators were introduced in Chapter 8,  you learned how to define a generator on an object literal by prepending a star (`*`) to the method name. The same syntax works for classes as well, allowing any method to be a generator. Here's an example:

```js
class MyClass {

    *createIterator() {
        yield 1;
        yield 2;
        yield 3;
    }

}

let instance = new MyClass();
let iterator = instance.createIterator();
```

This code creates a class called `MyClass` that has a generator method called `createIterator()`. The method returns an iterator whose values are hardcoded into the generator. While this is a useful capability, it's much more useful to define a default iterator for your class.

You can define the default iterator for a class by using `Symbol.iterator` to define a generator method, such as:

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

This example uses a computed name for a generator method that delegates to the `values()` iterator of `this.items`. In this way, any instance of `Collection` can be used directly in a `for-of` loop or with the spread operator. It's a good idea to define a default iterator for any class that manages a collection of values.

Adding methods and accessor properties to a class prototype is useful when you want those to show up on object instances. If, on the other hand, you'd like methods or accessor properties on the class itself, then you'll need to use static members.

## Static Members

Another common pattern in JavaScript is adding additional methods directly onto constructors to simulate static members. For example:

```js
function PersonType(name) {
    this.name = name;
}

// static method
PersonType.create = function(name) {
    return new PersonType(name);
};

// instance method
PersonType.prototype.sayName = function() {
    console.log(this.name);
};

var person = PersonType.create("Nicholas");
```

This code creates a factory method called `PersonType.create()`. In other programming languages, this would be considered a static method as it is not dependent on an instance of `PersonType` for its data.

Classes simplify the creation of static members by using the formal `static` annotation before the method or accessor property name. Here's the equivalent of the last example:

```js
class PersonClass {

    // equivalent of the PersonType constructor
    constructor(name) {
        this.name = name;
    }

    // equivalent of PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }

    // equivalent of PersonType.create
    static create(name) {
        return new PersonClass(name);
    }
}

let person = PersonClass.create("Nicholas");
```

The `PersonClass` definition has a single static method called `create()`. The method syntax is the same as for `sayName()` with the exception of the `static` keyword. You can use the `static` keyword on any method or accessor property definition within a class. The only restriction is that you cannot use `static` with the `constructor` method definition.

W> Static members are not accessible from instances. You must always access static members from the class directly.


## Inheritance with Derived Classes

Another problem with custom types in ECMAScript 5 and earlier was the extensive process necessary to implement inheritance. To properly inherit, you would need multiple steps. For instance:

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

function Square(length) {
    Rectangle.call(this, length, length);
}

Square.prototype = Object.create(Rectangle.prototype, {
    constructor: {
        value:Square,
        enumerable: true,
        writable: true,
        configurable: true
    }
});

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true
```

Here, `Square` inherits from `Rectangle`, and to do so, it must be overwrite `Square.prototype` with a new object created from `Rectangle.prototype` as well as call `Rectangle.call()`. These steps often confused newcomers to the language and were a source of errors for experienced developers.

Classes make inheritance easier by using the familiar `extends` keyword to specify the function from which the class should inherit. The prototypes are automatically adjusted and you can access the base class constructor using `super()`. Here's the equivalent of the previous example:

```js
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }
}

class Square extends Rectangle {
    constructor(length) {

        // same as Rectangle.call(this, length, length)
        super(length, length);
    }
}

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true
```

In this example, the `Square` class inherits from `Rectangle` using the `extends` keyword. The `Square` constructor uses `super()` to call the `Rectangle` constructor with the specified arguments. Note that unlike the ECMAScript 5 version of the code, the identifier `Rectangle` is only used within the class declaration (after `extends`).

Using `super()` is a requirement of derived classes if you specify a constructor (if you don't, an error will occur). If you choose not to use a constructor, then `super()` is automatically called for you with all arguments upon creating a new instance of the class. For instance, the following two classes are identical:

```js
class Square extends Rectangle {
    // no constructor
}

// Is equivalent to

class Square extends Rectangle {
    constructor(...args) {
        super(...args);
    }
}
```

The second class in this example shows the equivalent of the default constructor for all derived classes. All of the arguments are passed, in order, to the base class constructor. In this case, the functionality isn't quite correct because the `Square` constructor needs only one argument and so it's best to manually define the constructor.

W> There are a few things to keep in mind when using `super()`:
W>
W> 1. You can only use `super()` in a derived class. If you try to use it in a non-derived class (a class that doesn't use `extends`) or a function, it will throw an error.
W> 1. You must call `super()` before accessing `this` in the constructor. Since `super()` is responsible for initializing `this`, attempting to access `this` before calling `super()` results in an error.
W> 1. The only way to avoid calling `super()` is to return an object from the class constructor.

### Shadowing Class Methods

The methods on derived classes always shadow methods of the same name on the base class. For instance, you can add `getArea()` to `Square` in order to redefine that functionality:

```js
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }

    // override and shadow Rectangle.prototype.getArea()
    getArea() {
        return this.length * this.length;
    }
}
```

In this code, `getArea()` is now defined as part of `Square` and therefore `Rectangle.prototype.getArea()` will no longer be called by any instances of `Square`. Of course, you can always decide to call the base class version of the method by using `super.getArea()`, such as:

```js
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }

    // override, shadow, and call Rectangle.prototype.getArea()
    getArea() {
        return super.getArea();
    }
}
```

Using `super` in this way is the same as discussed in Chapter 4: the `this` value is automatically set correctly so you can make a simple method call.

### Inherited Static Members

If a base class has static members then those static members are also available on the derived class. This maps to how inheritance works in other languages, but is a new concept for JavaScript. Here's an example:

```js
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }

    static create(length, width) {
        return new Rectangle(length, width);
    }
}

class Square extends Rectangle {
    constructor(length) {

        // same as Rectangle.call(this, length, length)
        super(length, length);
    }
}

var rect = Square.create(3, 4);

console.log(rect instanceof Rectangle);     // true
console.log(rect.getArea());                // 12
console.log(rect instanceof Square);        // false
```

In this code, a new static `create()` method is added to `Rectangle`. Through inheritance, that method is available as `Square.create()` and behaves in the same manner as `Rectangle.create()`.

### Derived Classes from Expressions

Perhaps the most powerful aspect of derived classes in ECMAScript 6 is the ability to derive a class from an expression. You can use `extends` with any expression as long as the expression resolves to a function with `[[Construct]]` and a prototype. For example:

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x instanceof Rectangle);    // true
```

This example defines `Rectangle` as an ECMAScript 5-style constructor while `Square` is a class. Since `Rectangle` has `[[Construct]]` and a prototype, the class can still inherit directly from it.

Accepting any type of expression after `extends` allows for some powerful possibilities, such as dynamically determining what to inherit from. For example:

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

function getBase() {
    return Rectangle;
}

class Square extends getBase() {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x instanceof Rectangle);    // true
```

Here, the `getBase()` function is called directly as part of the class declaration. It returns `Rectangle`, which means this example is functionally equivalent to the previous one. And since you can determine the base dynamically, that means it's possible to create different inheritance approaches. For instance, you can effectively create mixins:

```js
let SerializableMixin = {
    serialize() {
        return JSON.stringify(this);
    }
};

let AreaMixin = {
    getArea() {
        return this.length * this.width;
    }
};

function mixin(...mixins) {
    var base = function() {};
    Object.assign(base.prototype, ...mixins);
    return base;
}

class Square extends mixin(AreaMixin, SerializableMixin) {
    constructor(length) {
        super();
        this.length = length;
        this.width = length;
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x.serialize());             // "{"length":3,"width":3}"
```

In this example, mixins are used instead of classical inheritance. The `mixin()` function takes any number of arguments that represent mixin objects. It creates a function called `base` and assigns the properties of each mixin object to the prototype. The function is then returned so `Square` can use `extends`. Keep in mind that since `extends` is still used, you are required to call `super()` in the constructor.

The instance of `Square` has both `getArea()` from `AreaMixin` and `serialize` from `SerializableMixin`. This is accomplished through prototypal inheritance, as the `mixin()` function dynamically populates the prototype of a new function with all of the own properties of each mixin.

W> Even though any expression can be used after `extends`, not all expressions result in a valid class. Specifically, the following expression types causes errors:
W>
W> * `null`
W> * generator functions (chapter 8)
W>
W> In these cases, attempting to create a new instance of the class will throw an error because there is no `[[Construct]]` to call.

### Inheriting from Built-ins

For almost as long as there have been JavaScript arrays, developers have wanted to inherit from arrays to create their own special array types. However, in ECMAScript 5 and earlier, this wasn't possible. Attempting to use classical inheritance didn't result in functioning code. For example:

```js
// built-in array behavior
var colors = [];
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined

// trying to inherit from array in ES5

function MyArray() {
    Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
    constructor: {
        value: MyArray,
        writable: true,
        configurable: true,
        enumerable: true
    }
});

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 0

colors.length = 0;
console.log(colors[0]);             // "red"
```

As you can see, using the classical form of JavaScript inheritance results in unexpected behavior. The `length` and numeric properties don't behave the same as the built-in array because this functionality isn't covered either by `Array.apply()` or by assigning the prototype.

One of the goals of ECMAScript 6 classes is to allow inheritance from all built-ins. In order to accomplish this, the inheritance model of classes is slightly different than the classical inheritance model found in ECMAScript 5 and earlier:

* In ECMAScript 5 classical inheritance, the value of `this` is first created by the derived type (for example, `MyArray`) and then the base type constructor is called (`Array.apply()`). That means `this` starts out as an instance of `MyArray` and then is decorated with additional properties from `Array`.
* In ECMAScript 6 class-based inheritance, the value of `this` is first created by the base (`Array`) and then modified by the derived class constructor (`MyArray`). The result is that `this` starts out with all of the built-in functionality of the base and correctly receives all functionality related to it.

The following class-based special array works as you would expect:

```js
class MyArray extends Array {
    // ...
}

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined
```

In this example, `MyArray` inherits directly from `Array` and therefore works in the exact same way. Interacting with numeric properties updates the `length` property, and manipulating the `length` property updates the numeric properties. That means not only can you properly inherit from `Array` to create your own derived array classes, you can also inherit from other builtins as well. ECMAScript 6 and derived classes have effectively removed the last special case of inheriting from builtins.

## new.target

In Chapter 3, you learned about `new.target` and how its value changes depending on how a function is called. You can also use `new.target` in class constructors to determine how the class is being invoked. In the simple case, `new.target` is equal to the constructor function for the class, as in this example:

```js
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

// new.target is Rectangle
var obj = new Rectangle(3, 4);      // outputs true
```

In this code, you can see that `new.target` is equivalent to `Rectangle` when `new Rectangle(3, 4)` is called. Since class constructors cannot be called without `new`, `new.target` is always defined inside of class constructors. However, the value may not always be the same.

```js
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

class Square extends Rectangle {
    constructor(length) {
        super(length, length)
    }
}

// new.target is Square
var obj = new Square(3);      // outputs false
```

Here, `Square` is calling the `Rectangle` constructor, so `new.target` is equal to `Square` when the `Rectangle` constructor is called. This is important because it gives each constructor the ability to alter its behavior based on how it's being called. For instance, you can create an abstract base class (one that cannot be instantiated directly) by using `new.target`:

```js
// abstract base class
class Shape {
    constructor() {
        if (new.target === Shape) {
            throw new Error("This class cannot be instantiated directly.")
        }
    }
}

class Rectangle extends Shape {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

var x = new Shape();                // throws error

var y = new Rectangle(3, 4);        // no error
console.log(y instanceof Shape);    // true
```

In this example, the `Shape` class constructor throws an error whenever `new.target` is `Shape`, meaning that `new Shape()` always throws an error. However, you can still use `Shape` as a base class, which is what `Rectangle` does. The `super()` call executes the `Shape` constructor and `new.target` is equal to `Rectangle` so the constructor continues without error.

I> Since classes cannot be called without `new`, `new.target` is never `undefined` inside of a class constructor.

## Summary

For those who have struggled to understand JavaScript in the absence of classes, ECMAScript 6 classes provide an easier way to become acclimated with the language without needing to completely throw away their understanding of inheritance. ECMAScript 6 classes start out as syntactic sugar for the classical inheritance model of ECMAScript 5, but add a lot of features to reduce mistakes.

ECMAScript 6 classes work with prototypal inheritance by defining non-static methods on the class prototype while static methods end up on the constructor itself. All methods are non-enumerable, which better matches the behavior of built-in objects for which methods are typically nonenumerable by default. Additionally, class constructors cannot be called without `new`, ensuring that you can't accidentally call a class as a function.

Class-based inheritance allows you to derive a class from another class, function, or expression. This ability means you can call a function to determine the correct base to inherit from, allowing you to use mixins and other different composition patterns to create a new class. Inheritance works in such a way that inheriting from built-in objects, such as `Array`, is now possible and works as expected.

You can use `new.target` in class constructors to behave differently depending on how the class is called. The most common use is to create an abstract base class that throws an error when instantiated directly but still allows inheritance via other classes.

Overall, classes are an important addition to the language that provide a more concise syntax and better functionality for defining custom object types in a safe, consistent manner.
