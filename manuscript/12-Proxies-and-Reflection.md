# Proxies and Reflection

W> Work in progress! Not ready for review.

One of the goals shared by ECMAScript 5 and ECMAScript 6 is the continued demystifying of JavaScript functionality. Prior to ECMAScript 5, for example, there were nonenumerable and nonwritable object properties present in JavaScript environments, but there was no way for developers to define nonenumerable or nonwritable properties of their own. This led to inclusion of `Object.defineProperty()` in ECMAScript 5 to give developers the ability to do what JavaScript engines were already capable of doing.

ECMAScript 6 continues the trend of giving developers the ability to do what JavaScript engines already do with built-in objects. To do that, the language had to expose more about how objects work, and the result was the creation of proxies. But before you can understand what proxies and are and what they can do, it's helpful to understand the type of problem that proxies are meant to address.

## The Array Problem

The JavaScript array object has a long history of having behaviors that developers cannot mimic in their own objects. The `length` property is affected by assigning values to specific array items, and you can modify array items by modifying the `length` property. For example:

```js
let colors = ["red", "green", "blue"];

console.log(colors.length);         // 3

colors[3] = "black";

console.log(colors.length);         // 4
console.log(colors[3]);             // "black"

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
```

Here, the array `colors` starts out with three items. Assigning to `colors[3]` automatically increments the `length` property to four. Setting the `length` property back to two removes the last two items in the array, leaving only the first two items. There is nothing in ECMAScript 5 that allows developers to achieve this same behavior. And this is why proxies were created.

## What are Proxies and Reflection?

A proxy, created using `new Proxy()`, is an object that you use in place of another object (called the *target*). The proxy virtualizes the target, which means that the proxy and the target appear to be the same object to functionality using the proxy. Proxies, however, allow you to intercept low-level object operations on the target that are otherwise internal to the JavaScript engine. These low-level operations are intercepted using a *trap*, which is a function that responds to a specific operation.

The reflection API, represented by the `Reflect` object, is a collection of methods that provide the default behavior for the same low-level operations that proxies can override. There is a `Reflect` method for every proxy trap, and those methods have the same name and are passed the same arguments as their respective proxy trap. The following table contains all of the proxy traps.

| Proxy Trap               | Overrides the Behavior Of | Default Behavior |
|--------------------------|---------------------------|------------------|
|`getPrototypeOf`          | `Object.getPrototypeOf()` | `Reflect.getPrototypeOf()` |
|`setPrototypeOf`          | `Object.setPrototypeOf()` | `Reflect.setPrototypeOf()` |
|`isExtensible`            | `Object.isExtensible()`   | `Reflect.isExtensible()` |
|`preventExtensions`       | `Object.preventExtensions()` | `Reflect.preventExtensions()` |
|`getOwnPropertyDescriptor`| `Object.getOwnPropertyDescriptor()` | `Reflect.getOwnPropertyDescriptor()` |
|`has`                     | The `in` operator         | `Reflect.has()` |
|`get`                     | Reading a property value  | `Reflect.get()` |
|`set`                     | Writing to a property     | `Reflect.set()` |
|`deleteProperty`          | The `delete` operator     | `Reflect.deleteProperty()` |
|`defineProperty`          | `Object.defineProperty()` | `Reflect.defineProperty` |
|`enumerate`               | `for-in` and `Object.keys()` | `Reflect.enumerate()` |
|`ownKeys`                 | `Object.getOwnPropertyNames()` and `Object.getOwnPropertySymbols()` | `Reflect.ownKeys()` |
|`apply`                   | Calling a function | `Reflect.apply()` |
|`construct`               | Calling a function with `new` | `Reflect.construct()` |


Each of the traps overrides some built-in behavior of JavaScript objects, allowing you to intercept and modify the behavior. If you need to still use the built-in behavior, then you can use the corresponding reflection API method. The relationship between proxies and the reflection API becomes clear when you start creating proxies, so it's best to look at some examples.

## Creating a Proxy

Proxies are created using the `Proxy` constructor and passing in two arguments, the target and a handler. A *handler* is an object that defines one or more traps. The proxy uses the default behavior for all operations except when traps are defined for that operation. To create a simple forwarding proxy, you can use a handler without any traps:

```js
let target = {},
    proxy = new Proxy(target, {});

proxy.name = "proxy";
console.log(proxy.name);        // "proxy"
console.log(target.name);       // "proxy"

target.name = "target";
console.log(proxy.name);        // "target"
console.log(target.name);       // "target"
```

In this example, `proxy` forwards all operations directly to `target`. The property `name` is created on `target` when `proxy.name` is assigned `"proxy"`. The proxy itself is not storing this property, it's simply forwarding the operation to `target`. Similarly, the values of `proxy.name` and `target.name` are the same because they are both references to `target.name`. That also means setting `target.name` to a new value results in `proxy.name` reflecting the same change. Of course, proxies without traps aren't very interesting, so what happens when you define a trap?

### Validating Properties Using the `set` Trap

Suppose you want to create an object whose property values must be numbers. That means every new property added to the object must be validated and an error thrown if the value is not a number. To accomplish this, you must define a `set` trap that overrides the default behavior of setting a value. The `set` trap receives four arguments:

1. `trapTarget` - the object that will receive the property (usually the proxy's target)
1. `key` - the property key (string or symbol) to write to
1. `value` - the value being written to the property
1. `receiver` - the object on which the operation took place (usually the proxy)

The corresponding reflection method is `Reflect.set()`, which is the default behavior for this operation. The `Reflect.set()` method accepts the same four arguments as the `set` proxy trap, making the method easy to use inside of the trap. The trap should return `true` if the property was set or `false` if not (`Reflect.set()` returns the correct value based on if the operation succeeded).

To validate the value of properties, you use the `set` trap and inspect the `value` that is passed in. Here's an example:

```js
let target = { name: "target" },
    proxy = new Proxy(target, {
        set(trapTarget, key, value, receiver) {

            // ignore existing properties so as not to affect them
            if (!trapTarget.hasOwnProperty(key)) {
                if (Object.is(Number(value), NaN)) {
                    throw new TypeError("Property must be a number.");
                }
            }

            // add the property
            return Reflect.set(trapTarget, key, value, receiver);
        }
    });

// adding a new property
proxy.count = 1;
console.log(proxy.count);       // 1
console.log(target.count);      // 1

// you can assign to name because it exists on target already
proxy.name = "proxy";
console.log(proxy.name);        // "proxy"
console.log(target.name);       // "proxy"

// throws an error
proxy.anotherName = "proxy";
```

This example defines a proxy trap that validates the value of any new property added to `target`. When `proxy.count = 1` is executed, the `set` trap is called. The `trapTarget` is equal to `target`, `key` is `"count"`, `value` is `1`, and `receiver` (not used in this example) is `proxy`. Because there is no existing property named `count` in `target`, the value is validated by passing it to `Number` and comparing to `NaN`. If the result is `NaN`, then the property value is not numeric and an error is thrown. However, since `count` was set to `1`, the new property is added by calling `Reflect.set()` with the same four arguments that were passed to the trap.

When `proxy.name` is assigned a string, the operation completes successfully. Since `target` already has a `name` property, that property is omitted from the validation check by using `trapTarget.hasOwnProperty()`. This ensure that previously-existing non-numeric property values are still supported.

When `proxy.anotherName` is assigned a string, however, an error is thrown. The `anotherName` property doesn't exist on the target, so its value is validated. Because `"proxy"` is not a numeric value, the error is thrown.

Whereas the `set` proxy trap lets you intercept when properties are being written to, the `get` proxy trap lets you intercept when properties are being read.

### Object Shape Validation Using the `get` Trap

One of the interesting, and sometimes confusing, aspects of JavaScript is that reading nonexistent properties does not throw an error. Instead, the value `undefined` is used for the property value. For example:

```js
let target = {};

console.log(target.name);       // undefined
```

In most other languages, attempting to read `target.name` throws an error because the property doesn't exist. JavaScript, however, uses the value `undefined` for `target.name`. If you've ever worked on a large code base, this behavior can cause significant problems, especially when you have a typo in the property name. With proxies, you can save yourself from this problem by having object shape validation.

An *object shape* is the collection of properties and methods available on the object. JavaScript engines use the object shape to optimize the code, often creating classes to represent the objects. If you can safely assume that an object will only have the same properties and methods it began with (using `Object.preventExtensions()`, `Object.seal()`, or `Object.freeze()`) then it can be helpful to throw an error when you try to access a nonexistent property. Such object shape validation is easy to do using proxies.

Since property validation only has to be done when a property is read, you need to use the `get` trap. The `get` trap is called whenever a property is read, even if that property doesn't exist on the object. There are three arguments passed to the `get` trap:

1. `trapTarget` - the object that from which the property is read (usually the proxy's target)
1. `key` - the property key (string or symbol) to read
1. `receiver` - the object on which the operation took place (usually the proxy)

These arguments mirror those for the `set` trap, with the noticeable difference being there is no `value` argument. The `Reflect.get()` method accepts these same three arguments and returns the property's default value. You can use these to throw an error when a property doesn't exist on the target:

```js
let target = {},
    proxy = new Proxy(target, {
        get(trapTarget, key, receiver) {
            if (!(key in receiver)) {
                throw new TypeError("Property " + key + " doesn't exist.");
            }

            return Reflect.get(trapTarget, key, receiver);
        }
    });

// adding a property still works
proxy.name = "proxy";
console.log(proxy.name);            // "proxy"

// nonexistent properties throw an error
console.log(proxy.nme);             // throws error
```

In this example, the `get` trap is used to intercept property read operations. The `in` operator is used to determine if the property already exists on the `receiver`. The `receiver` is used with `in` instead of `trapTarget` in case `receiver` is a proxy with a `has` trap. Using `trapTarget` would sidestep the `has` trap and potentially give you the wrong result. An error is thrown if the property doesn't exist, and otherwise, the default behavior is used.

This code allows new properties to be added, such as adding `proxy.name`, which is written to and read from without any problem. The last line contains a typo, `proxy.nme`, which should probably be `proxy.name`. This throws an error because `nme` does not exist as a property.




TODO: Continue and clean up below



```js
let target = { length: 0 },
    proxy = new Proxy(target, {
        set(trapTarget, key, value) {

            let numericKey = Number(key),
                keyIsInteger = Number.isInteger(numericKey);

            if (key === "length" && value < trapTarget.length) {
                for (let index = trapTarget.length; index >= value; index--) {
                    Reflect.deleteProperty(trapTarget, index);
                }
            } else if (keyIsInteger && numericKey >= trapTarget.length) {
                Reflect.set(trapTarget, "length", numericKey + 1);
            }

            Reflect.set(trapTarget, key, value);
        }
    });

console.log(proxy.length);      // 0

proxy[0] = "proxy1";

console.log(proxy.length);      // 1

proxy[1] = "proxy2";

console.log(proxy.length);      // 2

proxy.length = 0;

console.log(proxy.length);      // 0
console.log(proxy[0]);          // undefined
console.log(0 in proxy);        // false






Many programming languages have what is referred to as a reflection API. *Reflection* is the term used for utilities that can inspect code and otherwise interact with code during runtime in a manner similar to the compiler or execution engine. For instance, a reflection API might allow you to determine the number of arguments for an arbitrary function or return a list of properties on a given object.

If you've written JavaScript before you may be thinking that you've accomplished exactly these tasks. JavaScript has, for a long time, had reflection capabilities built into the language. The problem was that these capabilities were spread out and sometimes hidden. For instance, you can retrieve the number of arguments in a function by using the `length` property of functions and you can tell if an object property is enumerable by using the `propertyIsEnumerable()` method that each object inherits. ECMAScript 6 sought to begin cleaning up JavaScript reflection so that as much functionality as possible is centralized.

## The Reflect Object

The new reflection API for JavaScript is represented by the `Reflect` global object. This is an object and not a constructor despite beginning with an uppercase letter (as opposed to, for example, `Array` or `Object`). The `Reflect` object has no other purpose than to a be a container for reflection methods.

Reflection methods are operations that previously were not exposed in JavaScript, and instead existed only as low-level primitive operations in the ECMAScript specification. With the introduction of proxies and other more complex ways of intercepting low-level operations, it because necessary to introduce references to the original low-level operations so that developers can have known safe implementations.

While some of the reflection methods look similar to those already available, they do have subtle differences related to their lower-level relative to the existing methods. Pay close attention to how each method differs from an already existing method.

## Property-Related Methods

In ECMAScript 5, several new static methods were added to `Object` in order to work with object properties, such as `Object.defineProperty()`. During the course of ECMAScript 6 development, it was decided that several of these methods should be part of the reflection API and therefore should be present on `Reflect`. These methods are present both on `Object` and `Reflect`:

* `Reflect.defineProperty()`
* `Reflect.getOwnPropertyDescriptor()`
* `Reflect.preventExtensions()`
* `Reflect.isExtensible()`





TODO

1. apply()
2. construct
3. deleteProperty
4. enumerate()
5. get()
6. getPrototypeOf()
7. has()
8. ownKeys() - includes all keys, including symbols, whether enumerable or not
9. set()
10. setPrototypeOf()



# Proxies

Proxies have a long and complicated history in ECMAScript 6. An early proposal was implemented by both Firefox and Chrome before TC-39 decided to change proxies in a very dramatic way. The changes were, in my opinion, for the better, as they smoothed out a lot of the rough edges from the original proxies proposal.

Through experimentation, it was found that the vast majority of proxy uses revolved around a target object, in effect making proxies more of a filter in between a developer and an object rather than a standalone concept. Proxies were then reborn in a new form where the target object became part of the proxy definition. This was the proxy design that ultimately was standardized in ECMAScript 6.

## Proxy Theory

TODO

## Creating Proxies

TODO

```js
var proxy = new Proxy({}, {
    get: function(target, property) {
        return 35;
    }
});

console.log(proxy.time);        // 35
console.log(proxy.name);        // 35
console.log(proxy.title);       // 35
```

## Extra

`Array.isArray(new Proxy([], {}))` is true.


## Uses


### Defensive Objects

Understanding how to intercept the `[[Get]]` operation is all that is necessary for creating "defensive" objects. I call them defensive because they behave like a defensive teenager trying to assert their independence of their parents' view of them ("I am *not* a child, why do you keep treating me like one?"). The goal is to throw an error whenever a nonexistent property is accessed ("I am `not` a duck, why do you keep treating me like one?"). This can be accomplished using the `get` trap and just a bit of code:

```js
function createDefensiveObject(target) {

    return new Proxy(target, {
        get: function(target, property) {
            if (property in target) {
                return target[property];
            } else {
                throw new ReferenceError("Property \"" + property + "\" does not exist.");
            }
        }
    });
}
```

The `createDefensiveObject()` function accepts a target object and creates a defensive object for it. The proxy has a `get` trap that checks the property when it's read. If the property exists on the target object, then the value of the property is returned. If, on the other hand, the property does not exist on the object, then an error is thrown. Here's an example:

```js
let person = {
    name: "Nicholas"
};

let defensivePerson = createDefensiveObject(person);

console.log(defensivePerson.name);      // "Nicholas"
console.log(defensivePerson.age);       // ReferenceError!
```

Here, the `name` property works as usual while `age` throws an error.
Defensive objects allow existing properties to be read, but non-existent properties throw an error when read. However, you can still add new properties without error:


```js
var person = {
    name: "Nicholas"
};

var defensivePerson = createDefensiveObject(person);

console.log(defensivePerson.name);        // "Nicholas"

defensivePerson.age = 13;
console.log(defensivePerson.age);         // 13
```

So objects retain their ability to mutate unless you do something to change that. Properties can always be added but non-existent properties will throw an error when read rather than just returning `undefined`.

You can then truly defend the interface of an object, disallowing additions and throwing an error when accessing a non-existent property, by using a couple of steps:

```js
function Person(name) {
    this.name = name;

    return createDefensiveObject(name);
}
```

TODO

### Type Safety

The idea behind type safety is that each variable or property can only contain a particular type of value. In type-safe languages, the type is defined along with the declaration. In JavaScript, of course, there is no way to make such a declaration natively. However, many times properties are initialized with a value that indicates the type of data it should contain. For example:

```js
var person = {
    name: "Nicholas",
    age: 16
};
```

In this code, it's easy to see that `name` should hold a string and `age` should hold a number. You wouldn't expect these properties to hold other types of data for as long as the object is used. Using proxies, it's possible to use this information to ensure that new values assigned to these properties are of the same type.

Since assignment is the operation to worry about (that is, assigning a new value to a property), you need to use the proxy `set` trap. The `set` trap gets called whenever a property value is set and receives four arguments: the target of the operation, the property name, the new value, and the receiver object. The target and the receiver are always the same (as best I can tell). In order to protect properties from having incorrect values, simply evaluate the current value against the new value and throw an error if they don't match:

```js
function createTypeSafeObject(object) {

    return new Proxy(object, {
          set: function(target, property, value) {
              var currentType = typeof target[property],
                  newType = typeof value;

              if (property in target && currentType !== newType) {
                  throw new Error("Property " + property + " must be a " + currentType + ".");
              } else {
                  target[property] = value;
              }
          }
    });
}
```

The `createTypeSafeObject()` method accepts an object and creates a proxy for it with a `set` trap. The trap uses `typeof` to get the type of the existing property and the value that was passed in. If the property already exists on the object and the types don't match, then an error is thrown. If the property either doesn't exist already or the types match, then the assignment happens as usual. This has the effect of allowing objects to receive new properties without error. For example:

```js
var person = {
    name: "Nicholas"
};

var typeSafePerson = createTypeSafeObject(person);

typeSafePerson.name = "Mike";        // succeeds, same type
typeSafePerson.age = 13;             // succeeds, new property
typeSafePerson.age = "red";          // throws an error, different types
```

In this code, the `name` property is changed without error because it's changed to another string. The `age` property is added as a number, and when the value is set to a string, an error is thrown. As long as the property is initialized to the proper type the first time, all subsequent changes will be correct. That means you need to initialize invalid values correctly. The quirk of `typeof null` returning "object" actually works well in this case, as a `null` property allows assignment of an object value later.

As with defensive objects, you can also apply this method to constructors:

```js
function Person(name) {
    this.name = name;
    return createTypeSafeObject(this);
}

var person = new Person("Nicholas");

console.log(person instanceof Person);    // true
console.log(person.name);                 // "Nicholas"
```

Since proxies are transparent, the returned object has all of the same observable characteristics as a regular instance of `Person`, allowing you to create as many instances of a type-safe object while making the call to `createTypeSafeObject()` only once.
