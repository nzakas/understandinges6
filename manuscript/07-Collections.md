# Collections

For most of JavaScript's history, there has been only one type of collection, represented by the `Array` type. Arrays are used in JavaScript just like arrays in other languages but are often used as queues and stacks as well. Since arrays only use numeric indices, developers used non-array objects whenever a non-numeric index was necessary. That led to custom implementations of sets and maps.

A *set* is a list of values that cannot contain duplicates. You typically don't access items in the set like you would items in an array; instead, it's much more common to check the set to see if a value is present. A *map* is a collection of keys that are mapped to specific values. As such, each item in a map stores two pieces of data and values are retrieved by specifying the key to read from. Maps are frequently used as caches, storing data that is to be quickly retrieved later on.

While ECMAScript 5 didn't formally have sets and maps, developers were able to workaround this limitation using non-array objects.

## Sets and Maps in ECMAScript 5

In ECMAScript 5 and earlier, developers mimicked sets and maps by using object properties, such as:

```js
let set = Object.create(null);

set.foo = true;

// checking for existence
if (set.foo) {

    // do something
}
```

The variable `set` in this example is an object with a `null` prototype, ensuring that there are no inherited properties on the object. This is a common ECMAScript 5 approach that uses object properties as unique values to be checked. Properties are added and set to `true` so they may easily be used in conditional statements (such as the `if` statement in this example) to see if the value is present. The only real difference between an object used as a set and an object used as a map is the value being stored. For instance:

```js
let map = Object.create(null);

map.foo = "bar";

// retrieving a value
let value = map.foo;

console.log(value);         // "bar"
```

This example uses an object as a map, storing a string value `"bar"` under the key `"foo"`. Unlike sets, maps are mostly used to retrieve information (rather than just checking for the key's existence).

### Problems

While the approach of using objects as sets and maps works okay in simple situations, it can get a bit more complicated once you run into the limitations of object properties. Since all object properties must be strings, you must certain that no two keys evaluate to the same string. Consider the following:

```js
let map = Object.create(null);

map[5] = "foo";

console.log(map["5"]);      // "foo"
```

This example sets a numeric property of `5` to the value `"foo"`. Internally, that numeric value is converted to a string, so `map["5"]` and `map[5]` actually reference the same property. That can cause problems when keys can be either numbers or strings. Another problem is when using objects as keys:

```js
let map = Object.create(null),
    key1 = {},
    key2 = {};

map[key1] = "foo";

console.log(map[key2]);     // "foo"
```

Here, `map[key2]` is referencing the same value as `map[key1]`. Each object, `key1` and `key2`, are converted to strings because object properties must be strings. The default string representation for objects is `"[object Object]"`, so both `key1` and `key2` are converted to that string. This is a very difficult problem to identify and debug when it occurs in large applications, and a prime reason that ECMAScript 6 adds both sets and maps to the language.

## Sets in ECMAScript 6

ECMAScript 6 adds a `Set` type that is an ordered list of values without duplicates. Sets are allow fast access to the data contained within, adding a more efficient manner of tracking discrete values. Sets are created using `new Set()` and items are added to the set by using the `add()` method. You can see how many items are in the set by using the `size` property.

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.size);    // 2
```

Sets do not coerce values to determine they are the same. So, a set can contain both the number `5` and the string `"5"` (internally, the comparison is done using `Object.is()`, which was discussed in Chapter 4). That means you can also add multiple objects to the set and they remain distinct:

```js
let set = new Set(),
    key1 = {},
    key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size);    // 2
```

Because `key1` and `key2` are not converted to strings, they count as two unique items in the set.

If the `add()` method is called more than once with the same value, all calls after the first one are effectively ignored:

```js
let set = new Set();
set.add(5);
set.add("5");
set.add(5);     // duplicate - this is ignored

console.log(set.size);    // 2
```

You can initialize the set using an array, and the `Set` constructor will ensure that only unique values are used:

```js
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
console.log(set.size);    // 5
```

In this example, an array with feed set is used to initialize the set. The number `5` only appears once in the set even though it appears four times in the array. This functionality makes it easy to convert existing code or JSON structures to use sets.

I> The `Set` constructor actually accepts any iterable object as an argument. Arrays work because they are iterable by default, as are sets and maps. The `Set` constructor uses an iterator to extract values from the argument. Iterables and iterators are discussed in Chapter 8.

You can test to see which set are in the set using the `has()` method:

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true
console.log(set.has(6));    // false
```

Last, you can remove an item from the set by using the `delete()` method:

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true

set.delete(5);

console.log(set.has(5));    // false
```

All of this amounts to a very easy mechanism for tracking unique unordered values.

Keep in mind that while sets are great for tracking values, you can't randomly access a value contained in a set like you can with an array. If you need to do so, then the best option is to convert the set into an array.

### Converting to an Array

It's easy to convert an array into a set because you can pass the array to the `Set` constructor. It's also easy to convert a set back into an array using the spread operator. The spread operator (`...`) was introduced in Chapter 3 as a way to split items in an array into separate function parameters. You can also use the spread operator to work on iterable objects such as sets to convert them into an array. For example:

```js
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

Here, a set is initially loaded with an array that contains duplicates. The set removes the duplicates and then the items are placed into a new array using the spread operator. The set itself still contains the same items, it's just that they've been copied to a new array. This approach is useful when you already have an array and want to create an array without duplicates. For example:

```js
function eliminateDuplicates(items) {
    return [...new Set(items)];
}

let numbers = [1, 2, 3, 3, 3, 4, 5],
    noDuplicates = eliminateDuplicates(numbers);

console.log(noDuplicates);      // [1,2,3,4,5]
```

In the `eliminateDuplicates()` function, the set is just a temporary intermediary used to filter out duplicate values before creating a new array that has no duplicates.

### Weak Sets

The `Set` might alternately be called a strong set because of the way it stores object references. An object stored in an instance of `Set` is effectively the same as storing that object inside of variable, meaning that as long as that reference exists, the variable cannot be garbage collected to free memory. For example:

```js
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size);      // 1

// eliminate original reference
key = null;

console.log(set.size);      // 1

// get the original reference back
key = ...set[0];
```

In this example, setting `key` to `null` clears one reference of the object but another remains inside the set. So, you can retrieve that value by converting the set to an array using the spread operator and accessing the first item. That works fine for most uses, but sometimes it's better for references in a set to disappear when all other references disappear. For instance, if your JavaScript is running in a web page and wants to keep track of DOM elements (that might be removed by another script), you don't want your code holding onto the last reference to a DOM element - this is called a memory leak.

A weak set is a type of set that only stores weak object references (they cannot store primitive values). A *weak reference* to an object is one that does not prevent garbage collection if it is the only remaining reference. Weak sets are created using the `WeakSet` constructor and have `add()`, `has()`, and `delete()` methods. Here's an example:

```js
let set = new WeakSet(),
    key = {};

// add the object to the set
set.add(key);

console.log(set.has(key));      // true

set.delete(key);

console.log(set.has(key));      // false
```

Using a weak set is a lot like using a regular set. You can add, remove, and check for references in the weak set. You can also seed a weak set with values by passing an iterable to the constructor:

```js
let key1 = {},
    key2 = {},
    set = new WeakSet([key1, key2]);

console.log(set.has(key1));     // true
console.log(set.has(key2));     // true
```

In this example, an array is passed to the `WeakSet` constructor. Since this array contains two objects, those objects are added into the weak set. Keep in mind that an error is thrown if the array contains any non-object values.

#### Key Differences

The biggest difference from regular sets is the weak reference held to the object value. Here's an example of what that means:

```js
let set = new WeakSet(),
    key = {};

// add the object to the set
set.add(key);

console.log(set.has(key));      // true

// remove the last strong reference to key, also removes from weak set
key = null;
```

After this code executes, the reference to `key` in the weak set is no longer accessible. It is not possible to verify its removal because you would need one reference to that object to pass to `has()`. This can make testing weak sets a little confusing, however, you can trust that the reference has been properly removed by the JavaScript engine.

From these examples, you can see that weak sets are virtually identical to sets. The key differences are:

1. The `add()`, `has()`, and `delete()` methods throw an error when passed a non-object.
1. Weak sets are not iterables and therefore cannot be used in a `for-of` loop.
1. Weak sets do not expose any iterators (such as `keys()` and `values()`), so there is no way to programmatically determine the contents of a weak set.

## Maps in ECMAScript 6

The ECMAScript 6 `Map` type is an ordered list of key-value pairs where both the key and the value can be of any type. A key of `5` is different than a key of `"5"`, and keys are determined to be the same using `Object.is()`. You can store and retrieve data from a map using the `set()` and `get()` methods, respectively:

```js
let map = new Map();
map.set("name", "Nicholas");
map.set(document.getElementById("my-div"), { flagged: false });

// later
let name = map.get("name"),
    meta = map.get(document.getElementById("my-div"));
```

In this example, two key-value pairs are stored. The key `"name"` stores a string while the key `document.getElementById("my-div")` is used to associate meta data with a DOM element. If the key doesn't exist in the map, then the special value `undefined` is returned when calling `get()`.

Maps shared a couple of methods with sets, such as `has()` for determining if a key exists in the map and `delete()` for removing a key-value pair from the map. You can also use `size` to determine how many items are in the map:

```js
let map = new Map();
map.set("name", "Nicholas");

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.size);          // 1

map.delete("name");
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.size);          // 0
```

In order to make it easier to add large amounts of data into a map, you can pass an array of arrays to the `Map` constructor. Internally, each key-value pair is stored as an array with two items, the first being the key and the second being the value. The entire map, therefore, is an array of these two-item arrays and so maps can be initialized using that format:

```js
let map = new Map([ ["name", "Nicholas"], ["title", "Author"]]);

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.has("title"));  // true
console.log(map.get("title"));  // "Author"
console.log(map.size);          // 2
```

When you want to work with all of the data in the map, you have several options. There are actually three generator methods to choose from: `keys`, which iterates over the keys in the map, `values`, which iterates over the values in the map, and `entries`, which iterates over key-value pairs by returning an array containing the key and the value (`entries` is the default iterator for maps). The easiest way to make use of these is to use a `for-of` loop:

```js
for (let key of map.keys()) {
    console.log("Key: %s", key);
}

for (let value of map.values()) {
    console.log("Value: %s", value);
}

for (let item of map.entries()) {
    console.log("Key: %s, Value: %s", item[0], item[1]);
}

// same as using map.entries()
for (let item of map) {
    console.log("Key: %s, Value: %s", item[0], item[1]);
}
```

When iterating over keys or values, you receive a single value each time through the loop. When iterating over entries, you receive an array whose first item is the key and the second item is the value.

Another way to iterate over items is to use the `forEach()` method. This method works in a similar manner to `forEach()` on arrays. You pass in a function that gets called with three arguments: the value, the key, and the map itself. For example:

```js
map.forEach(function(value, key, map)) {
    console.log("Key: %s, Value: %s", key, value);
});
```

Also similar to the array's version of `forEach()`, you can pass in an optional second argument to specify the `this` value to use inside the callback:

```js
let reporter = {
      report: function(key, value) {
        console.log("Key: %s, Value: %s", key, value);
      }
};

map.forEach(function(value, key, map) {
    this.report(key, value);
}, reporter);
```

Here, the `this` value inside of the callback function is equal to `reporter`. That allows `this.report()` to work correctly.

Compare this to the clunky way of iterating over values of a regular object:

```js
for (let key in object) {

    // make sure it's not from the prototype!
    if (object.hasOwnProperty(key)) {
      console.log("Key: %s, Value: %s", key, object[key]);
    }

}
```

When using objects as maps, it was always a concern that properties from the prototype might leak through in a `for-in` loop. You always need to use `hasOwnProperty()` to be certain that you are getting only the properties that you wanted. Of course, if there were methods on the object, you would also have to filter those:

```js
for (let key in object) {

    // make sure it's not from the prototype or a function!
    if (object.hasOwnProperty(key) && typeof object[key] !== "function") {
      console.log("Key: %s, Value: %s", key, object[key]);
    }

}
```

The iteration features of maps allow you to focus on just the data without worrying about extra pieces of information slipping into your code. This is another big benefit of maps over regular objects for storing key-value pairs.


## Weakmaps

Weakmaps are similar to regular maps in that they map a value to a unique key. That key can later be used to retrieve the value it identifies. Weakmaps are different because the key must be an object and cannot be a primitive value. This may seem like a strange constraint, but it's actually the core of what makes weakmaps different and useful.

A weakmap holds only a weak reference to a key in the same way weaksets hold a weak reference to a value. When the object is destroyed by the garbage collector, the weakmap automatically removes the key-value pair identified by that object. The canonical example for using weakmaps is to create an object related to a particular DOM element. For example, jQuery maintains a cache of objects internally, one for each DOM element that has been referenced. Using a weakmap would allow jQuery to automatically free up memory associated with a DOM element when it is removed from the document.

The ECMAScript 6 `WeakMap` type is an unordered list of key-value pairs where the key must be a non-null object and the value can be of any type. The interface for `WeakMap` is very similar to that of `Map` in that `set()` and `get()` are used to add and retrieve data respectively:

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

// later
let value = map.get(element);
console.log(value);             // "Original"

// later still - remove reference
element.parentNode.removeChild(element);
element = null;

value = map.get(element);
console.log(value);             // undefined
```

In this example, one key-value pair is stored. The key is a DOM element used to store a corresponding string value. That value was later retrieved by passing in the DOM element to `get()`. If the DOM element is then removed from the document and the variable referencing it is set to `null`, then the data is also removed from the weakmap and the next attempt to retrieve data associated with the DOM element fails.

This example is a little bit misleading because the second call to `map.get(element)` is using the value of `null` (which `element` was set to), rather than a reference to the DOM element. You can't use `null` as a key in weakmaps, so this code isn't really doing a valid lookup. Unfortunately, there is no part of the interface that allows you to query whether or not a reference has been cleared (because the reference no longer exists).

A> The weakmap `set()` method will throw an error if you try to use a primitive value as a key. If you want to use a primitive value as a key, then it's best to use `Map` instead.

Weakmaps also have `has()` for determining if a key exists in the map and `delete()` for removing a key-value pair.

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

console.log(map.has(element));   // true
console.log(map.get(element));   // "Original"

map.delete(element);
console.log(map.has(element));   // false
console.log(map.get(element));   // undefined
```

Here, a DOM element is once again used as the key in a weakmap. The `has()` method is useful for checking to see if a reference is currently being used as a key in the weakmap. Keep in mind that this only works when you have a non-null reference to a key. The key is forcibly removed from the weakmap by using `delete()`, at which point `has()` returns `false` and `get()` returns `undefined`.

### Uses and Limitations

Weakmaps have a very specific use case in mind, and that is mapping values to objects that might disappear in the future. The ability to free up memory related to these objects is useful for JavaScript libraries that wrap DOM elements with custom objects such as jQuery and YUI. There'll likely be more use cases discovered once implementations are complete and widespread, but in the short term, don't feel bad if you can't figure out a good spot for using weakmaps.

In many cases, a regular map is probably what you want to use. Weakmaps are limited in that they aren't enumerable and you can't keep track of how many items are contained within. There also isn't a way to retrieve a list of all keys. If you need this type of functionality, then you'll need to use a regular map. If you don't, and you only intend to use objects as keys, then a weakmap may be the right choice.

### Browser Support

Both Firefox and Chrome have implemented `WeakMap`, however, in Chrome you need to manually enable ECMAScript 6 features: go to `chrome://flags` and enable "Experimental JavaScript Features". Both implementations are complete per the current strawman specification (weakmaps are not yet in the ECMAScript 6 draft).

## Summary

TODO

ECMAScript 6 sets are a welcome addition to the language. They allow you to easily create a collection of unique values without worrying about type coercion. You can add and remove items very easily from a set even though there is no direct access to items in the set. It's still possible, if necessary, to iterate over items in the set by using the ECMAScript 6 `for-of` statement.

Since ECMAScript 6 is not yet complete, it's also possible that the implementation and specification might change before other browsers start to include `Set`. At this point in time, it is still considered experimental API and shouldn't be used in production code. This post, and other posts about ECMAScript 6, are only intended to be a preview of functionality that is to come.
