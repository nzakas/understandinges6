# Reflection

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

Many programming languages have what is referred to as a reflection API. *Reflection* is the term used for utilities that can inspect code and otherwise interact with code during runtime in a manner similar to the compiler or execution engine. For instance, a reflection API might allow you to determine the number of arguments for an arbitrary function or return a list of properties on a given object.

If you've written JavaScript before you may be thinking that you've accomplished exactly these tasks. JavaScript has, for a long time, had reflection capabilities built into the language. The problem was that these capabilities were spread out and sometimes hidden. For instance, you can retrieve the number of arguments in a function by using the `length` property of functions and you can tell if an object property is enumerable by using the `propertyIsEnumerable()` method that each object inherits. ECMAScript 6 sought to begin cleaning up JavaScript reflection so that as much as functionality as possible is centralized.

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
8. ownKeys()
9. set()
10. setPrototypeOf()
