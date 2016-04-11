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
|`ownKeys`                 | `Object.getOwnPropertyNames()` and `Object.getOwnPropertySymbols()` | `Reflect.ownKeys()` |
|`apply`                   | Calling a function | `Reflect.apply()` |
|`construct`               | Calling a function with `new` | `Reflect.construct()` |


Each of the traps overrides some built-in behavior of JavaScript objects, allowing you to intercept and modify the behavior. If you need to still use the built-in behavior, then you can use the corresponding reflection API method. The relationship between proxies and the reflection API becomes clear when you start creating proxies, so it's best to look at some examples.

I> The original ECMAScript 6 specification had an additional trap called `enumerate` that was designed to alter how `for-in` and `Object.keys()` enumerated properties on an object. However, the `enumerate` trap was removed in ECMAScript 7 (also called ECMAScript 2016) as difficulties were discovered during implementation. The `enumerate` trap was never released in any JavaScript environment and is therefore not covered in this chapter.

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

### Property Descriptor Traps

One of the most important features of ECMAScript 5 was the ability to define property attributes using `Object.defineProperty()`. In previous versions, there was no way to define an accessor property, make a property read-only, or make a property nonenumerable. All of these are possible using `Object.defineProperty()`, and those attributes are retrievable using `Object.getOwnPropertyDescriptor()`. Proxies let you intercepts calls to `Object.defineProperty()` and `Object.getOwnPropertyDescriptor()` using the `defineProperty` and `getOwnPropertyDescriptor` traps, respectively. The `defineProperty` trap receives the following arguments:

1. `trapTarget` - the object on which the property should be defined (the proxy's target)
1. `key` - the string or symbol for the property
1. `descriptor` - the descriptor object for the property

The `defineProperty` trap requires you to return `true` if the operation is successful and `false` if not.The `getOwnPropertyDescriptor` traps receives only `trapTarget` and `key`, and you are expected to return the descriptor. The corresponding `Reflect.defineProperty()` and `Reflect.getOwnPropertyDescriptor()` methods accept the same arguments as their proxy trap counterparts. Here's a simple example that just implements the default behavior for each trap:

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        return Reflect.defineProperty(trapTarget, key, descriptor);
    },
    getOwnPropertyDescriptor(trapTarget, key) {
        return Reflect.getOwnPropertyDescriptor(trapTarget, key);
    }
});


Object.defineProperty(proxy, "name", {
    value: "proxy"
});

console.log(proxy.name);            // "proxy"

let descriptor = Object.getOwnPropertyDescriptor(proxy, "name");

console.log(descriptor.value);      // "proxy"
```

This example defines a property `"name"` on the proxy using `Object.defineProperty()`. The property descriptor for that property is then retrieved using `Object.getOwnPropertyDescriptor()`.

#### Blocking Object.defineProperty()

The `defineProperty` trap requires you to return a boolean value to indicate if the operation was successful. When `true` is returned, `Object.defineProperty()` succeeds as usual; when `false` is returned, `Object.defineProperty()` throws an error. You can use this functionality to restrict the kind of properties that can be defined using `Object.defineProperty()`. For instance, if you want to prevent symbol properties from being defined, you could check that the key is a string and return `false` if not. For example:

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {

        if (typeof key !== "string") {
            return false;
        }

        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});


Object.defineProperty(proxy, "name", {
    value: "proxy"
});

console.log(proxy.name);                    // "proxy"

let nameSymbol = Symbol("name");

// throws error
Object.defineProperty(proxy, nameSymbol , {
    value: "proxy"
});
```

In this code, the `defineProperty` proxy trap returns `false` when `key` isn't a string and otherwise proceeds with the default behavior. When `Object.defineProperty()` is called with a key of `"name"`, it succeeds because the key is a string. When `Object.defineProperty()` is called with `nameSymbol`, then it throws an error because the `definePropert` trap returns `false`.

I> You can also have `Object.defineProperty()` silently failed by returning `true` and not calling `Reflect.defineProperty()`. That will suppress the error while not actually defining the property.

#### Descriptor Object Restrictions

The descriptor object passed to the `defineProperty` trap and returned from `getOwnPropertyDescriptor` trap are normalized and validated, respectively, to ensure consistent behavior when using `Object.defineProperty()` and `Object.getOwnPropertyDescriptor()`. To start, no matter what object is passed as the third argument to `Object.defineProperty()` only the properties `enumerable`, `configurable`, `value`, `writable`, `get`, and `set` will be on the descriptor object passed to the `defineProperty` trap. For example:

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        console.log(descriptor.value);              // "proxy"
        console.log(descriptor.name);               // undefined

        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});


Object.defineProperty(proxy, "name", {
    value: "proxy",
    name: "custom"
});
```

Here, `Object.defineProperty()` is called with a nonstandard `name` property on the third argument. When the `defineProperty` trap is called, the `descriptor` object does not have a `name` property but does have a `value` property. That's because `descriptor` is not a reference to the actual third argument to `Object.defineProperty()`, but rather a new object that contains only the allowable properties. The `Reflect.defineProperty()` method also ignores any nonstandard properties on the descriptor.

The `getOwnPropertyDescriptor` trap has a slightly different restriction that requires the return value to be `null`, `undefined`, or an object. If an object is returned, only `enumerable`, `configurable`, `value`, `writable`, `get`, and `set` are allowed as own properties of the object. An error is thrown if you return an object with an own property that isn't allowed, such as:

```js
let proxy = new Proxy({}, {
    getOwnPropertyDescriptor(trapTarget, key) {
        return {
            name: "proxy";
        };
    }
});

// throws error
let descriptor = Object.getOwnPropertyDescriptor(proxy, "name");
```

The property `name` is not allowable on property descriptors, so when `Object.getOwnPropertyDescriptor()` is called, the `getOwnPropertyDescriptor` return value triggers an error. This restriction ensures that the value returned by `Object.getOwnPropertyDescriptor()` always has a reliable structure regardless of use on proxies.

#### Duplicate Descriptor Methods

Once again, ECMAScript 6 has some confusingly similar methods, as `Object.defineProperty() and `Object.getOwnPropertyDescriptor()` appear to do the same thing as `Reflect.defineProperty()` and `Reflect.getOwnPropertyDescriptor()`, respectively. As with other method pairs discussed earlier in this chapter, there are some subtle but important differences.

The `Object.defineProperty()` and `Reflect.defineProperty()` method are exactly the same except for their return values. The `Object.defineProperty()` method returns the first argument whereas `Reflect.defineProperty()` returns a boolean value, `true` if the operation succeeded and `false` if not. For example:


```js
let target = {};

let result1 = Object.defineProperty(target, "name", { value: "target "});

console.log(target === result1);        // true

let result2 = Reflect.defineProperty(target, "name", { value: "reflect" });

console.log(result2);                   // true
```

When `Object.defineProperty()` is called on `target`, the return value is `target`. When `Reflect.defineProperty()` is called on `target`, the return value is `true`, indicating that the operation succeeded. Since the `defineProperty` proxy trap requires a boolean value to be returned, it's better to use `Reflect.defineProperty()` to implement the default behavior when necessary.

The `Object.getOwnPropertyDescriptor()` method coerces its first argument into an object when a primitive value is passed and then continues the operation whereas `Reflect.getOwnPropertyDescriptor()` throws an error if the first argument is a primitive value. Here's an example:

```js
let descriptor1 = Object.getOwnPropertyDescriptor(2, "name");
console.log(descriptor1);       // undefined

// throws an error
let descriptor2 = Reflect.getOwnPropertyDescriptor(2, "name");
```

The `Object.getOwnPropertyDescriptor()` method returns `undefined` because it coerces `2` into an object and that object doesn't have a `name` property. This is the standard behavior of the method when a property with the given name isn't found on an object. However, when `Reflect.getOwnPropertyDescriptor()` is called, an error is thrown immediately because it does not accept primitive values for the first argument.

### The `ownKeys` Trap

The `ownKeys` proxy trap intercepts the internal method `[[OwnPropertyKeys]]` and allows you to override that behavior by returning an array of values. This array is used in three places: `Object.getOwnPropertyNames()`, `Object.getOwnPropertySymbols()`, and `Object.assign()` (to determine which properties to copy). The default behavior, as implemented by `Reflect.ownKeys()`, is to return an array of all own property keys (both strings and symbols). The `Object.getOwnProperyNames()` method filters symbols out of the array and returns the result while `Object.getOwnPropertySymbols()` filters the strings out of the array and returns the result. The `Object.assign()` method uses the array with both strings and symbols.

The `ownKeys` trap receives a single argument, the target, and must always return an array or array-like object (otherwise, an error is thrown). Using the `ownKeys` trap you can, for example, filter out certain property keys that you don't want used when `Object.getOwnPropertyNames()`, `Object.getOwnPropertySymbols()`, or `Object.assign()` are used. Suppose you don't want to include any property name that begins with an underscore character, a common notation in JavaScript indicating that a field is private. You can use the `ownKeys` trap to filter out those keys:

```js
let proxy = new Proxy({}, {
    ownKeys(trapTarget) {
        return Reflect.ownKeys(trapTarget).filter(key => {
            return typeof key !== "string" || key[0] !== "_";
        });
    }
});

let nameSymbol = Symbol("name");

proxy.name = "proxy";
proxy._name = "private";
proxy[nameSymbol] = "symbol";

let names = Object.getOwnPropertyNames(proxy),
    symbols = Object.getOwnPropertySymbols(proxy);

console.log(names.length);      // 1
console.log(names[0]);          // "proxy"

console.log(symbols.length);    // 1
console.log(symbols[0]);        // "Symbol(name)"
```

This example uses an `ownKeys` trap that first calls `Reflect.ownKeys()` to get the default list of keys for the target. Then, the `filter()` method is used to filter out keys that are strings and begin with an underscore character. The proxy object has three properties added, `name`, `_name`, and `nameSymbol`. When `Object.getOwnPropertyNames()` is called on `proxy`, only the `name` property is returned. Similarly, only `nameSymbol` is returned when `Object.getOwnPropertySymbols()` is called on `proxy`. The `_name` property doesn't appear in either result because it has been filtered out.

While the `ownKeys` proxy trap allows you to alter the keys returned from a small set of operations, it does not affect more commonly used operations such as the `for-of` loop and `Object.keys()`. Those cannot be altered using proxies.

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

#### Calling Constructors Without new

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

#### Overriding Abstract Base Class Constructors

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

## Solving the Array Problem

At the beginning of this chapter, I explained how the behavior of an array could not be accurately mimicked in JavaScript prior to ECMAScript 6. Proxies and the reflection API allow you to create an object that behaves in the same manner as the built-in `Array` type when properties are added and removed. To refresh your memory, here's the behavior that proxies help to mimick:

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

There are a couple things going on in this example, so here's a break down:

1. The `length` property is increased to 4 when `colors[3]` is assigned a value.
1. The last two items in the array are deleted when the `length` property is set to 2.

These two behaviors, which I'll call behavior #1 and behavior #2, are the only ones that need to be mimicked to accurately recreate how built-in arrays work. The next few sections describe how to make an object that correctly mimics these two behaviors.

### Detecting Array Indices

Keep in mind that assigning to an integer property key is a special case for arrays, as those are treated differently from non-integer keys. The specification gives some instructions on how to determine if a property key is an array index:

> A String property name `P` is an array index if and only if `ToString(ToUint32(P))` is equal to `P` and `ToUint32(P)` is not equal to 2^32^−1.

This operation can be implemented in JavaScript as the following:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}
```

The `toUint32()` function converts a given value into an unsigned 32-bit integer using an algorithm described in the specification. The `isArrayIndex()` function first converts the key into a uint32 and then performs the comparisons to determine if the key is an array index or not. With these utility functions available, you can start to implement an object that will mimic a built-in array.

### Implementing Behavior #1

You might have noticed that both behavior #1 and behavior #2 rely on the assignment of a property. That means you really only need to use the `set` proxy trap to accomplish both behaviors. To get started, here's an example that implements the first of the two array behaviors by incrementing the `length` property when an array index larger than `length - 1` is used:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length=0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {

            let currentLength = Reflect.get(trapTarget, "length");

            // the special case
            if (isArrayIndex(key)) {
                let numericKey = Number(key);

                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, "length", numericKey + 1);
                }
            }

            // always do this regardless of key type
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";

console.log(colors.length);         // 3

colors[3] = "black";

console.log(colors.length);         // 4
console.log(colors[3]);             // "black"
```

This example uses the `set` proxy trap to intercept the setting of an array index. If the key is an array index, then it is converted into a number (since keys are always passed as strings). Next, if that numeric value is greater than or equal to the current `length` property, then the `length` property is updated to be one more than the numeric key (setting an item in position 3 means the `length` must be 4). After that, the default behavior for setting a property is used via `Reflect.set()`, since you do want the property to receive the value as specified.

The initial custom array is created by calling `createMyArray()` with a `length` of 3 and the values for those three items are added immediately afterward. The `length` property correctly remains 3 until the value `"black"` is assigned to position 3. At that point, `length` is set to 4.

With behavior #1 working, it's now time to move on to behavior #2.

### Implementing Behavior #2

Whereas behavior #1 is used only when an array index is greater than or equal to the `length` property, behavior #2 does the opposite and removes array items when the `length` property is set to a smaller value than it previously contained. That means not only changing the `length` property, but also deleting all of the items that might otherwise exist. For instance, if an array with a `length` of 4 then has `length` set to 2, the items in position 2 and 3 are deleted. This can be accomplished inside of the `set` proxy trap alongside behavior #1:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length=0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {

            let currentLength = Reflect.get(trapTarget, "length");

            // the special case
            if (isArrayIndex(key)) {
                let numericKey = Number(key);

                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, "length", numericKey + 1);
                }
            } else if (key === "length") {

                if (value < currentLength) {
                    for (let index = currentLength; index >= value; index--) {
                        Reflect.deleteProperty(trapTarget, index);
                    }
                }

            }

            // always do this regardless of key type
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";
colors[3] = "black";

console.log(colors.length);         // 4

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
console.log(colors[0]);             // "red"
```

The `set` proxy trap in this code checks to see if `key` is `"length"` in order to adjust the rest of the object correctly. When that happens, the current length is first retrieved using `Reflect.get()` and compared against the new value. If the new value is less than the current length, then a `for` loop deletes all of the properties on the target that should no longer be available. The `for` loop goes backwards from the current array length (`currentLength`) of the array and deletes each property until it reaches the new array length (`value`).

This example adds four colors to start and then sets the `length` property to 2. Doing so effectively removes the items in positions 2 and 3, so they now return `undefined` when you attempt to access them. The `length` property is correctly set to 2 and the items in positions 0 and 1 are still accessible.

With both behaviors now implemented, you can easily create an object that mimics the behavior of built-in arrays. However, doing so with a function isn't as desirable as creating a class to encapsulate this behavior, so the next step is to implement this functionality as a class.

### Implementing the MyArray Class

The simplest way to create a class that uses a proxy is to define the class as usual and then return a proxy from the constructor. So the object returned with a class is instantiated is the proxy instead of the instance (the value of `this` inside the constructor). The instance becomes the target of the proxy and the proxy is returned as if it were the instance. That means the instance is completely private and cannot be accessed directly (it can be accessed indirectly through the proxy). Here's a simple example of returning a proxy from a class constructor:

```js
class Thing {
    constructor() {
        return new Proxy(this, {});
    }
}

let myThing = new Thing();
console.log(myThing instanceof Thing);      // true
```

In this example, the class `Thing` returns a proxy from its constructor. The proxy target is `this` and the proxy is returned from the constructor. That means `myThing` is actually a proxy even though it was created by calling the `Thing` constructor. Because proxies pass through their behavior to the target, `myThing` is still considered an instance of `Thing`, making the proxy completely transparent to anyone using the `Thing` class.

With that in mind, it's fairly straightforward to create a custom array class using a proxy. The code is mostly the same as the code in the "Implementing Behavior #2" section. The same proxy code is used, but this time, it's used inside of a class constructor. Here's the complete example:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

class MyArray {
    constructor(length=0) {
        this.length = length;

        return new Proxy(this, {
            set(trapTarget, key, value) {

                let currentLength = Reflect.get(trapTarget, "length");

                // the special case
                if (isArrayIndex(key)) {
                    let numericKey = Number(key);

                    if (numericKey >= currentLength) {
                        Reflect.set(trapTarget, "length", numericKey + 1);
                    }
                } else if (key === "length") {

                    if (value < currentLength) {
                        for (let index = currentLength; index >= value; index--) {
                            Reflect.deleteProperty(trapTarget, index);
                        }
                    }

                }

                // always do this regardless of key type
                return Reflect.set(trapTarget, key, value);
            }
        });

    }
}


let colors = new MyArray(3);
console.log(colors instanceof MyArray);     // true

console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";
colors[3] = "black";

console.log(colors.length);         // 4

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
console.log(colors[0]);             // "red"
```

This code creates a `MyArray` class that returns a proxy from its constructor. The `length` property is added in the constructor (initialized to the value that is passed in or the default value of 0) and then a proxy is created and returned. This gives the `colors` variable the appearance of being just an instance of `MyArray` and has both behavior #1 and behavior #2.

Although returning a proxy from a class constructor is easy, it does mean that a new proxy is created for every instance. There is a way to have all instances share one proxy -- it's a big more complicated and involves using the proxy as a prototype.
