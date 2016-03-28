# Proxies and Reflection

W> This chapter is a work in progress. The content should be correct but may be incomplete.

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
|`get`                     | Reading a property value  | `Reflect.get()` |
|`set`                     | Writing to a property     | `Reflect.set()` |
|`has`                     | The `in` operator         | `Reflect.has()` |
|`deleteProperty`          | The `delete` operator     | `Reflect.deleteProperty()` |
|`getPrototypeOf`          | `Object.getPrototypeOf()` | `Reflect.getPrototypeOf()` |
|`setPrototypeOf`          | `Object.setPrototypeOf()` | `Reflect.setPrototypeOf()` |
|`isExtensible`            | `Object.isExtensible()`   | `Reflect.isExtensible()` |
|`preventExtensions`       | `Object.preventExtensions()` | `Reflect.preventExtensions()` |
|`getOwnPropertyDescriptor`| `Object.getOwnPropertyDescriptor()` | `Reflect.getOwnPropertyDescriptor()` |
|`defineProperty`          | `Object.defineProperty()` | `Reflect.defineProperty` |
|`enumerate`               | `for-in` and `Object.keys()` | `Reflect.enumerate()` |
|`ownKeys`                 | `Object.getOwnPropertyNames()` and `Object.getOwnPropertySymbols()` | `Reflect.ownKeys()` |
|`apply`                   | Calling a function | `Reflect.apply()` |
|`construct`               | Calling a function with `new` | `Reflect.construct()` |


Each of the traps overrides some built-in behavior of JavaScript objects, allowing you to intercept and modify the behavior. If you need to still use the built-in behavior, then you can use the corresponding reflection API method. The relationship between proxies and the reflection API becomes clear when you start creating proxies, so it's best to look at some examples.

## Creating a Proxy

Proxies are created using the `Proxy` constructor and passing in two arguments, the target and a handler. A *handler* is an object that defines one or more traps. The proxy uses the default behavior for all operations except when traps are defined for that operation. To create a simple forwarding proxy, you can use a handler without any traps:

```js
let target = {};

let proxy = new Proxy(target, {});

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

1. `trapTarget` - the object that will receive the property (the proxy's target)
1. `key` - the property key (string or symbol) to write to
1. `value` - the value being written to the property
1. `receiver` - the object on which the operation took place (usually the proxy)

The corresponding reflection method is `Reflect.set()`, which is the default behavior for this operation. The `Reflect.set()` method accepts the same four arguments as the `set` proxy trap, making the method easy to use inside of the trap. The trap should return `true` if the property was set or `false` if not (`Reflect.set()` returns the correct value based on if the operation succeeded).

To validate the value of properties, you use the `set` trap and inspect the `value` that is passed in. Here's an example:

```js
let target = {
    name: "target"
};

let proxy = new Proxy(target, {
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

1. `trapTarget` - the object from which the property is read (the proxy's target)
1. `key` - the property key (string or symbol) to read
1. `receiver` - the object on which the operation took place (usually the proxy)

These arguments mirror those for the `set` trap, with the noticeable difference being there is no `value` argument. The `Reflect.get()` method accepts these same three arguments and returns the property's default value. You can use these to throw an error when a property doesn't exist on the target:

```js
let proxy = new Proxy({}, {
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

### Hiding Property Existence Using the `has` Trap

The `in` operator determines if a property exists on a given object and returns `true` if there is either an own property or a prototype property matching the name or symbol. For example:

```js
let target = {
    value: 42;
}

console.log("value" in target);     // true
console.log("toString" in target);  // true
```

Both `value` and `toString` exist on `object`, so in both cases the `in` operator returns `true`. The `value` property is an own property while `toString` is a prototype property (inherited from `Object`). Proxies allow you to intercept this operation and return a different value for `in` by using the `has` trap.

The `has` trap is called whenever the `in` operator is used. When called, there are two arguments passed to the `has` trap:

1. `trapTarget` - the object from which the property is read (the proxy's target)
1. `key` - the property key (string or symbol) to check

The `Reflect.has()` method accepts these same arguments and returns the default response for the `in` operator. Using the `has` trap and `Reflect.has()` allows you to alter the behavior of `in` for some properties while falling back to default behavior for others. For instance, suppose you just want to hide the `value` property, you can do so like this:

```js
let target = {
    name: "target",
    value: 42
};

let proxy = new Proxy(target, {
    has(trapTarget, key) {

        if (key === "value") {
            return false;
        } else {
            return Reflect.has(trapTarget, key);
        }
    }
});


console.log("value" in proxy);      // false
console.log("name" in proxy);       // true
console.log("toString" in proxy);   // true
```

The `has` trap for `proxy` checks to see if `key` is `"value"`, and if so, returns `false`. Otherwise, the default behavior is used via `Reflect.has()`. The result is that the `in` operator returns `false` for the `value` property even though it actually does exist on the target. The other properties, `name` and `toString`, correctly return `true` when used with the `in` operator.

### Preventing Property Deletion with the `deleteProperty` Trap

The `delete` operator removes a property from an object and returns `true` when successfull and `false` when unsuccessful. In strict mode, `delete` throws an error when you attempt to delete a nonconfigurable property (`delete` simply returns `false` is nonstrict mode). Here's an example:

```js
let target = {
    name: "target",
    value: 42
};

Object.defineProperty(target, "name", { configurable: false });

console.log("value" in target);     // true

let result1 = delete target.value;
console.log(result1);               // true

console.log("value" in target);     // false

// Note: The following line throws an error in strict mode
let result2 = delete target.name;
console.log(result2);               // false

console.log("name" in target);      // true
```

The `value` property is deleted using the `delete` operator and, as a result, the `in` operator returns `false`. The nonconfigurable `name` property can't be deleted so the `delete` operator simply returns `false` (if this code is run in strict mode, an error is thrown instead). You can alter this behavior by using the `deleteProperty` trap in a proxy.

The `deleteProperty` trap is called whenever the `delete` operator is used on an object property. The trap is passed two arguments:

1. `trapTarget` - the object from which the property should be deleted (the proxy's target)
1. `key` - the property key (string or symbol) to delete

The `Reflect.deleteProperty()` method provides the default implementation of the `deleteProperty` trap and accepts these same two arguments. Using a combination of `Reflect.deleteProperty()` and the `deleteProperty` trap, you can alter the behavior of the `delete` operator, for instance, by ensuring that the `value` property cannot be deleted:

```js
let target = {
    name: "target",
    value: 42
};

let proxy = new Proxy(target, {
    deleteProperty(trapTarget, key) {

        if (key === "value") {
            return false;
        } else {
            return Reflect.deleteProperty(trapTarget, key);
        }
    }
});

// Attempt to delete proxy.value

console.log("value" in proxy);      // true

let result1 = delete proxy.value;
console.log(result1);               // false

console.log("value" in proxy);      // true

// Attempt to delete proxy.name

console.log("name" in proxy);       // true

let result2 = delete proxy.name;
console.log(result2);               // true

console.log("name" in proxy);       // false
```

This code is very similar to the `has` trap example in that the `deleteProperty` trap checks to see if the `key` is `"value"`, and if so, returns `false`. Otherwise, the default behavior is used by calling `Reflect.deleteProperty()`. The `value` property cannot be deleted through `proxy` because the operation is trapped whereas the `name` property is deleted as expected. This approach is especially useful when you want to protect properties from deletion without causing an error to be thrown in strict mode.

### Prototype Proxy Traps

In Chapter 4, you learned about the ECMAScript 6 `Object.setPrototypeOf()` method that was added to complement the ECMAScript 5 `Object.getPrototypeOf()` method. Proxies allow you to intercept the execution of both methods through the `setPrototypeOf` and `getPrototypeOf` traps. In both cases, the method on `Object` calls the trap of the corresponding name on the proxy, allowing you to alter their behavior. The `setPrototypeOf` trap receives these arguments:

1. `trapTarget` - the object for which the prototype should be set (the proxy's target)
1. `proto` - the object to use for as the prototype

These are the same arguments passed to `Object.setPrototypeOf()` and `Reflect.setPrototypeOf()`. The `getPrototypeOf` trap only receives the `trapTarget` argument, which is the argument passed to `Object.getPrototypeOf()` and `Reflect.setPrototypeOf()`.

I> Yes, there are two sets of methods: `Object.getPrototypeOf()` and `Object.setPrototypeOf()`, along with `Reflect.getPrototypeOf()` and `Reflect.setPrototypeOf()`. The differences between these methods are subtle and are discussed later in this chapter.

There are some restrictions on these traps. First, the `getPrototypeOf` trap must return an object or `null`, and any other return value results in a runtime error. The return value check ensures that `Object.getPrototypeOf()` will always return an expected value. Similarly, the return value of the `setPrototypeOf` trap must be `false` if the operation does not succeed. When `setPrototypeOf` returns `false`, `Object.setPrototypeOf()` throws an error. If `setPrototypeOf` returns any value other than `false`, then `Object.setPrototypeOf()` assumes the operation succeeded.

The following example hides the prototype of the proxy by always returning `null` and also doesn't allow the prototype to be changed:

```js
let target = {};
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return null;
    },
    setPrototypeOf(trapTarget, proto) {
        return false;
    }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype);      // true
console.log(proxyProto === Object.prototype);       // false
console.log(proxyProto);                            // null

// succeeds
Object.setPrototypeOf(target, {});

// throws error
Object.setPrototypeOf(proxy, {});
```

You can see the difference between the behavior of `target` and `proxy` in this example. While `Object.getPrototypeOf()` returns a value for `target`, it returns `null` for `proxy` because the `getPrototypeOf` trap is called. Similarly, `Object.setPrototypeOf()` succeeds when used on `target` but throws an error when used on `proxy` due to the `setPrototypeOf` trap.

If you want to use the default behavior for these two traps, you can use the corresponding methods on `Reflect`. For instance, this example implements the default behavior for the `getPrototypeOf` and `setPrototypeOf` traps:

```js
let target = {};
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return Reflect.getPrototypeOf(trapTarget);
    },
    setPrototypeOf(trapTarget, proto) {
        return Reflect.setPrototypeOf(trapTarget, proto);
    }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype);      // true
console.log(proxyProto === Object.prototype);       // true

// succeeds
Object.setPrototypeOf(target, {});

// also succeeds
Object.setPrototypeOf(proxy, {});
```

In this example, you can use `target` and `proxy` interchangeably and get the same results because the `getPrototypeOf` and `setPrototypeOf` traps are just passing through to use the default implementation. It's important that this example use the `Reflect.getPrototypeOf()` and `Reflect.setPrototypeOf()` methods rather than the methods of the same name on `Object` due to some important differences.

#### Why Two Sets of Methods?

The confusing aspect of `Reflect.getPrototypeOf()` and `Reflect.setPrototypeOf()` is that they look suspiciously similar to `Object.getPrototypeOf()` and `Object.setPrototypeOf()`. While both sets of methods perform similar operations, there are some distinct differences between the two.

To begin, `Object.getPrototypeOf()` and `Object.setPrototypeOf()` are higher-level operations, meaning that they were created for developer use from the start. The `Reflect.getPrototypeOf()` and `Reflect.setPrototypeOf()` methods are lower-level operations, meaning that they are exposing some behavior that was previously not intended for developers. The methods on `Reflect` are meant to give access to the previously internal-only `[[GetPrototypeOf]]` and `[[SetPrototypeOf]]` operations. You can think of the relationship between these two sets of methods as `Object.getPrototypeOf()` calls `Reflect.getPrototypeOf()` and `Object.setPrototypeOf()` calls `Reflect.setPrototypeOf()`. While this isn't strictly true according to the specification, this is still a good way to describe their relationship.

The `Reflect.getPrototypeOf()` methods throws an error if its argument is not an object, whereas `Object.getPrototypeOf()` first coerces the value into an object before performing the operation. So if you were to pass a number into each method, you'd get a different result:

```js
let result1 = Object.getPrototypeOf(1);
console.log(result1 === Number.prototype);  // true

// throws an error
Reflect.getPrototypeOf(1);
```

The `Object.getPrototypeOf()` method allows you retrieve a prototype for the number `1` because it first coerces the value into a `Number` object and then returns `Number.prototype`. The `Reflect.getPrototypeOf()` method does not coerce the value, and since `1` is not an object, it throws an error.

The `Reflect.setPrototypeOf()` method also has some distinct differences from the `Object.setPrototypeOf()` method. First, it returns a boolean value indicating if the operation was successful (`true` for success, `false` for failure); if `Object.setPrototypeOf()` fails, it throws an error. As you saw earlier, when the `setPrototypeOf` proxy trap returns `false`, it causes `Object.setPrototypeOf()` to throw an error. The `Object.setPrototypeOf()` method returns the first argument as its value and therefore isn't suitable for implementing the default behavior of the `setPrototypeOf` proxy trap. Here's an example of these differences:

```js
let target1 = {};
let result1 = Object.setPrototypeOf(target1, {});
console.log(result1 === target1);                   // true

let target2 = {};
let result2 = Reflect.setPrototypeOf(target2, {});
console.log(result2 === target2);                   // false
console.log(result2);                               // true
```

In this example, `Object.setPrototypeOf()` returns `target1` as its value whereas `Reflect.setPrototypeOf()` returns `true`. This subtle difference is very important when used in proxy traps, so be sure to use the method on `Reflect` inside of them.

I> Both sets of methods will call the `getPrototypeOf` and `setPrototypeOf` proxy traps when used on a proxy.

### Object Extensibility Traps

ECMAScript 5 added object extensibility modification through the `Object.preventExtensions()` and `Object.isExtensible()` methods, and ECMAScript 6 allows proxies to intercept those method calls to the underlying objects through the `preventExtensions` and `isExtensible` traps. Both traps receive a single argument, `trapTarget`, that is the object on which the method was called. The `isExtensible` trap must return a boolean value indicating if the object is extensible while the `preventExtensions` trap must return a boolean value indicating if the operation succeeded.

There are also `Reflect.preventExtensions()` and `Reflect.isExtensible()` methods to implement the default behavior (both return boolean values, so they can be used directly in their corresponding traps). Here's a simple example that implements the default behavior for the `isExtensible` and `preventExtensions` traps:

```js
let target = {};
let proxy = new Proxy(target, {
    isExtensible(trapTarget) {
        return Reflect.isExtensible(trapTarget);
    },
    preventExtensions(trapTarget) {
        return Reflect.preventExtensions(trapTarget);
    }
});


console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true

Object.preventExtensions(proxy);

console.log(Object.isExtensible(target));       // false
console.log(Object.isExtensible(proxy));        // false
```

This example shows that both `Object.preventExtensions()` and `Object.isExtensible()` correctly pass through from `proxy` to `target`. You can, of course, also change the behavior. For example, if you didn't want to allow `Object.preventExtensions()` to succeed on your proxy, you can return `false` from the `preventExtensions` trap:

```js
let target = {};
let proxy = new Proxy(target, {
    isExtensible(trapTarget) {
        return Reflect.isExtensible(trapTarget);
    },
    preventExtensions(trapTarget) {
        return false
    }
});


console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true

Object.preventExtensions(proxy);

console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true
```

Here, the call to `Object.preventExtensions(proxy)` is effectively ignored because the `preventExtensions` trap returns `false`. The operation is not forwarded to the underlying `target`, so `Object.isExtensible()` returns `true`.

#### Duplicate Extensibility Methods

You may have noticed that, once again, there are seemingly duplicate methods on `Object` and `Reflect`, and in this case, they are more similar than not. The methods `Object.isExtensible()` and `Reflect.isExtensible()` are similar except when passed a non-object value. In that case, `Object.isExtensible()` always returns `false` while `Reflect.isExtensible()` throws an error. Here's an example of that behavior:

```js
let result1 = Object.isExtensible(2);
console.log(result1);                       // false

// throws error
let result2 = Reflect.isExtensible(2);
```

This restriction is similar to the difference between `Object.getPrototypeOf()` and `Reflect.getPrototypeOf()`, as lower-level functionality has stricter error checks than their higher-level counterparts.

The `Object.preventExtensions()` and `Reflect.preventExtensions()` methods are also very similar. The `Object.preventExtensions()` method always returns the value that was passed to it as an argument even if the value is not an object. The `Reflect.preventExtensions()` method, on the other hand, throws an error if the argument is not an object; if the argument is an object, then `Reflect.preventExtensions()` returns `true` when the operation succeeds or `false` if not. For example:

```js
let result1 = Object.preventExtensions(2);
console.log(result1);                               // 2

let target = {};
let result2 = Reflect.preventExtensions(target);
console.log(result2);                               // true

// throws error
let result3 = Reflect.preventExtensions(2);
```

Here, `Object.preventExtensions()` passed through the value `2` as its return value even though it's not an object. The `Reflect.preventExtensions()` method returns `true` when an object is passed to it and throws an error when `2` is passed to it.

As mentioned earlier, whenever there are seemingly duplicate methods on `Object` and `Reflect`, you should always use the methods on `Reflect` inside of proxy traps.

### Function Proxies Using the `apply` and `construct` traps

Of all the proxy traps, only `apply` and `construct` require the proxy target to be a function. You learned in Chapter 3 that functions have two internal methods, `[[Call]]` and `[[Construct]]`, that are executed when a function is called without and with the `new` operator, respectively. The `apply` and `construct` traps correspond to those internal methods and let you override them. The `apply` trap receives, and `Reflect.apply()` expects, the following the arguments when a function is called without `new`:

1. `trapTarget` - the function being executed (the proxy's target)
1. `thisArg` - the value of `this` inside of the function during the call
1. `argumentsList` - an array of arguments passed to the function

The `construct` trap, which is called when the function is executed using `new`, receives the following arguments:

1. `trapTarget` - the function being executed (the proxy's target)
1. `argumentsList` - an array of arguments passed to the function

The `Reflect.construct()` method also accepts these two arguments and has an optional third argument, `newTarget`. The `newTarget` argument, when given, specifies the value of `new.target` inside of the function.

Together, the `apply` and `construct` traps completely control the behavior of any proxy target function. To mimic the default behavior of a function, you can do this:


```js
let target = function() { return 42 },
    proxy = new Proxy(target, {
        apply: function(trapTarget, thisArg, argumentList) {
            return Reflect.apply(trapTarget, thisArg, argumentList);
        },
        construct: function(trapTarget, argumentList) {
            return Reflect.construct(trapTarget, argumentList);
        }
    });

// a proxy with a function as its target looks like a function
console.log(typeof proxy);                  // "function"

console.log(proxy());                       // 42

var instance = new proxy();
console.log(instance instanceof proxy);     // true
console.log(instance instanceof target);    // true
```

This example has a function that returns the number 42. The proxy for that function uses the `apply` and `construct` traps to delegate those behaviors to `Reflect.apply()` and `Reflect.construct()`, respectively. The end result is that the proxy function works exactly like the target function, including identifying itself as a function when `typeof` is used. The proxy is called without `new` to return 42 and then is called with `new` to create an object called `instance`. The `instance` object is considered an instance of both `proxy` and `target` because `instanceof` uses the prototype chain to determine this information. Since prototype chain lookup is not affected by this proxy, `proxy` and `target` appear to have the same prototype to the JavaScript engine.

#### Validating Function Parameters

The `apply` and `construct` traps open up a lot of possibilities for altering the way a function is executed. For instance, suppose you want to validate that all arguments are of a specific type? You can check the arguments in the `apply` trap:

```js
// adds together all arguments
function sum(...values) {
    return values.reduce((previous, current) => previous + current, 0);
}

let sumProxy = new Proxy(sum, {
        apply: function(trapTarget, thisArg, argumentList) {

            argumentList.forEach((arg) => {
                if (typeof arg !== "number") {
                    throw new TypeError("All arguments must be numbers.");
                }
            });

            return Reflect.apply(trapTarget, thisArg, argumentList);
        },
        construct: function(trapTarget, argumentList) {
            throw new TypeError("This function can't be called with new.");
        }
    });

console.log(sumProxy(1, 2, 3, 4));          // 10

// throws error
console.log(sumProxy(1, "2", 3, 4));

// also throws error
let result = new sumProxy();
```

This example uses the `apply` trap to ensure that all arguments are numbers. The `sum()` function adds up all of the arguments that are passed. If a non-number value is passed, it will still attempt the operation, which can result in unexpected results. By wrapping `sum()` into a proxy called `sumProxy()`, this code intercepts function calls and ensures that each argument is a number before allowing the call to proceed. To be safe, the code also uses the `construct` trap to ensure that the function can't be called with `new`.

You can also do the opposite, ensuring that a function must be called with `new`, and validating its arguments to be numbers:

```js
function Numbers(...values) {
    this.values = values;
}

let NumbersProxy = new Proxy(Numbers, {

        apply: function(trapTarget, thisArg, argumentList) {
            throw new TypeError("This function must be called with new.");
        },

        construct: function(trapTarget, argumentList) {
            argumentList.forEach((arg) => {
                if (typeof arg !== "number") {
                    throw new TypeError("All arguments must be numbers.");
                }
            });

            return Reflect.construct(trapTarget, argumentList);
        }
    });

let instance = new NumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]

// throws error
NumbersProxy(1, 2, 3, 4);
```

Here, the `apply` trap is throwing an error while the `construct` trap does input validation and returns a new instance using `Reflect.construct()`. Of course, you can accomplish the same thing without proxies if you use `new.target`.

### Calling Constructors Without new

Chapter 3 introduced the `new.target` metaproperty. To review, `new.target` is a reference to the function on which `new` is called, meaning that you can tell if a function was called using `new` or not by checking the value of `new.target`, such as:

```js
function Numbers(...values) {

    if (typeof new.target === "undefined") {
        throw new TypeError("This function must be called with new.");
    }

    this.values = values;
}

let instance = new Numbers(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]

// throws error
Numbers(1, 2, 3, 4);
```

This example throws an error when `Numbers()` is called without using `new` (similar to the example in the previous section, but without using a proxy). Writing code like this is much simpler than using a proxy and is preferable if your only goal is to prevent calling the function without `new`. However, sometimes you aren't in control of the function whose behavior needs to be modified. In that case, using a proxy makes sense.

Suppose that the `Numbers` function was defined elsewhere, in code you couldn't modify. You know that the code relies on `new.target` and so you want to avoid that check and still call the function. The behavior when using `new` is already set, so you can just use the `apply` trap:

```js
function Numbers(...values) {

    if (typeof new.target === "undefined") {
        throw new TypeError("This function must be called with new.");
    }

    this.values = values;
}


let NumbersProxy = new Proxy(Numbers, {
        apply: function(trapTarget, thisArg, argumentsList) {
            return Reflect.construct(trapTarget, argumentsList);
        }
    });


let instance = NumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]
```

The `NumbersProxy` function allows you to call `Numbers` without using `new` and have it behave as if `new` were used. To do so, the `apply` trap calls `Reflect.construct()` with the arguments passed into `apply`. Doing so means that `new.target` inside of `Numbers` is equal to `Numbers` itself and therefore no error is thrown. While this is a simple example of modifying `new.target`, you can also do so more directly.

### Overriding Abstract Base Class Constructors

You can go one step further and specify the third argument to `Reflect.construct()` as the specific value to assign to `new.target`. This is useful when a function is checking `new.target` against a known value, such as in the creation of an abstract base class constructor (discussed in Chapter 9). In an abstract base class constructor, `new.target` is expected to be something other than the class constructor itself, such as:

```js
class AbstractNumbers {

    constructor(...values) {
        if (new.target === AbstractNumbers) {
            throw new TypeError("This function must be inherited from.");
        }

        this.values = values;
    }
}

class Numbers extends AbstractNumbers {}

let instance = new Numbers(1, 2, 3, 4);
console.log(instance.values);           // [1,2,3,4]

// throws error
new AbstractNumbers(1, 2, 3, 4);
```

When `new AbstractNumbers()` is called, `new.target` is equal to `AbstractNumbers` and an error is thrown. However, `new Numbers()` works because `new.target` is equal to `Numbers`. You can bypass this restriction by manually assigning `new.target` with a proxy:

```js
class AbstractNumbers {

    constructor(...values) {
        if (new.target === AbstractNumbers) {
            throw new TypeError("This function must be inherited from.");
        }

        this.values = values;
    }
}

let AbstractNumbersProxy = new Proxy(AbstractNumbers, {
        construct: function(trapTarget, argumentList) {
            return Reflect.construct(trapTarget, argumentList, function() {});
        }
    });


let instance = new AbstractNumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]
```

The `AbstractNumbersProxy` uses the `construct` trap to intercept the call to `new AbstractNumbersProxy()`. Then, the `Reflect.construct()` method is called with arguments from the trap and adds an empty function as the third argument. That empty function is used as the value of `new.target` inside of the constructor. Because `new.target` is not equal to `AbstractNumbers`, no error is thrown and the constructor executes completely.

#### Callable Class Constructors

In Chapter 9, you learned that class constructors must always be called with `new`. That happens because the internal `[[Call]]` method for class constructors is specified to throw an error. However, since proxies can intercept calls to `[[Call]]`, you can effectively create callable class constructors my using a proxy. For instance, if you want a class constructor to work without using `new`, you can use the `apply` trap to create a new instance. Here's some sample code:

```js
class Person {
    constructor(name) {
        this.name = name;
    }
}

let PersonProxy = new Proxy(Person, {
        apply: function(trapTarget, thisArg, argumentList) {
            return new trapTarget(...argumentList);
        }
    });


let me = PersonProxy("Nicholas");
console.log(me.name);                   // "Nicholas"
console.log(me instanceof Person);      // true
console.log(me instanceof PersonProxy); // true
```

The `PersonProxy` object is a proxy of the `Person` class constructor. Class constructors are just functions, so they behave the same way when used in proxies. The `apply` trap overrides the default behavior and instead returns a new instance of `trapTarget`, which is equal to `Person` (the example uses `trapTarget` to show that you don't need to manually specify the class). The `argumentList` is passed to `trapTarget` using the spread operator to pass each argument separately. Calling `PersonProxy()` without using `new` returns an instance of `Person` (if you attempt to call `Person()` without `new`, it will still throw an error). Creating callable class constructors is something that is only possible using proxies.

## Revocable Proxies

Normally, a proxy cannot be unbound from its target once the proxy has been created. All of the examples to this point in this chapter have used nonrevocable proxies. However, there may be situations when you want to revoke a proxy at a later point in time so that it can no longer be used. This is most frequently the case when you want to provide an object through an API for security purposes, maintaining the ability to cut off access to some functionality at any point in time.

Revocable proxies are created using the `Proxy.revocable()` method, which the same arguments as the `Proxy` constructor, a target object and the proxy handler. The return value is an object with properties:

1. `proxy` - the proxy object itself
1. `revoke` - the function to call to revoke the proxy

When the `revoke()` function is called, no further operations can be performed through the `proxy`. Any attempt to interact with the proxy object in a way that would trigger a proxy trap throws an error. For example:

```js
let target = {
    name: "target"
};

let { proxy, revoke } = Proxy.revocable(target, {});

console.log(proxy.name);        // "target"

revoke();

// throws error
console.log(proxy.name);
```

This example creates a revocable proxy and uses destructuring to assign the `proxy` and `revoke` variables to the properties of the same name on the object returned from `Proxy.revocable()`. After that, the `proxy` object can be used just like a nonrevocable proxy object, so `proxy.name` returns `"target"` because it passes through to `target.name`. Once the `revoke()` function is called, however, `proxy` no longer functions. Attempting to access `proxy.name` throws an error, as will any other operation that relies on proxy traps.



<!--
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
-->
