# Collections

For most of JavaScript's history, there has been only one type of collection represented by the `Array` type. Arrays are used in JavaScript just like arrays and other languages but pull double and triple duty mimicking queues and stacks as well. Since arrays only use numeric indices, developers had to use objects whenever a non-numeric index was necessary. ECMAScript 6 introduces several new types of collections to allow better and more efficient storing of order data.

## Sets

Sets are nothing new if you come from languages such as Java, Ruby, or Python but have been missing from JavaScript. A set is in an ordered list of values that cannot contain duplicates. You typically don't access items in the set like you would items in an array, instead it's much more common to check the set to see if a value is present.

ECMAScript 6 introduces the `Set` type as a set implementation for JavaScript. You can add values to a set by using the `add()` method and see how many items are in the set using `size()`:

    var items = new Set();
    items.add(5);
    items.add("5");

    console.log(items.size());    // 2

ECMAScript 6 sets do not coerce values in determining whether or not to values are the same. So, a set can contain both the number `5` and the string `"5"` (internally, the comparison is done using `Object.is()`). If the `add()` method is called more than once with the same value, all calls after the first one are effectively ignored:

    var items = new Set();
    items.add(5);
    items.add("5");
    items.add(5);     // oops, duplicate - this is ignored

    console.log(items.size());    // 2

You can initialize the set using an array, and the `Set` constructor will ensure that only unique values are used:

    var items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
    console.log(items.size());    // 5

In this example, an array with feed items is used to initialize the set. The number `5` Only appears once in the set even though it appears four times in the array. This functionality makes it easy to convert existing code or JSON structures to use sets.

You can test to see which items are in the set using the `has()` method:

    var items = new Set();
    items.add(5);
    items.add("5");

    console.log(items.has(5));    // true
    console.log(items.has(6));    // false

Last, you can Remove an item from the set by using the `delete()` method:

    var items = new Set();
    items.add(5);
    items.add("5");

    console.log(items.has(5));    // true

    items.delete(5)

    console.log(items.has(5));    // false

All of this amounts to a very easy mechanism for tracking unique unordered values.

!!.values


Even though there is no random access to items in a set, it still possible to iterate over all of the sets values by using the new ECMAScript 6 `for-of` statement. The `for-of` statement is a loop that iterates over values of a collection, including arrays and array-like structures. you can output values in a set like this:

    var items = new Set([1, 2, 3, 4, 5]);

    for (let num of items) {
        console.log(num);
    }

This code outputs each item in the set to the console in the order in which they were added to the set.

If you would rather just convert the setback into an array, that can be accomplished using the `Array.from()` method:

    var items = new Set([1, 2, 3, 4, 5]);
    var array = Array.from(items);

In this way, you can easily remove duplicates from an array like this:

    function dedupe(array) {
        return Array.from(new Set(array));
    }



### Example

Currently, if you want to keep track of unique values, the most common approach is to use an object and assign the unique values as properties with some truthy value. For example, there is a CSS Lint rule that looks for duplicate properties. Right now, an object is used to keep track of CSS properties such as this:

    var properties = {
        "width": 1,
        "height": 1
    };

    if (properties[someName]) {
        // do something
    }

Using an object for this purpose means always assigning a truthy value to a property so that the `if` statement works correctly (the other option is to use the `in` operator, but developers rarely do). This whole process can be made easier by using a set:

    var properties = new Set();
    properties.add("width");
    properties.add("height");

    if (properties.has(someName)) {
        // do something
    }

Since it only matters if the property was used before and not how many times it was used (there is no extra metadata associated), it actually makes more sense to use a set.

Another downside of using object properties for this type of operation is that property names are always converted to strings. So you can't have an object with the property name of `5`, you can only have one with the property name of `"5"`. That also means you can't easily keep track of objects in the same manner because the objects get converted to strings when assigned as a property name. Sets, on the other hand, can contain any type of data without fear of conversion into another type.

### Browser Support

Both Firefox and Chrome have implemented `Set`, however, in Chrome you need to manually enable ECMAScript 6 features: go to `chrome://flags` and enable "Experimental JavaScript Features". Both implementations are incomplete. Neither browser implements set iteration using `for-of` and Chrome's implementation is missing the `size()` method.

## Maps

Maps are also a familiar topic  for those coming from other languages.  The basic idea is to map a value to a unique key in such a way that you can retrieve that value at any point in time by using the key. In JavaScript, developers have traditionally used regular objects as a replacement for maps. In fact, JSON is based on the premise that objects represent key-value pairs. However, the same limitation that affects objects used as sets also affects objects used as maps: the inability to have non-string keys.

Prior to ECMAScript 6, you might have seen code that looked like this:

    var map = {};

    // later
    if (!map[key]) {
        map[key] = value;
    }

This code uses a regular object to act like a map, checking to see if a given key exists. The biggest limitation here is that `key` will always be converted into a string. That's not a big deal until you want to use a non-string value as a key. For example, maybe you want to store some data that relates to particular DOM element. You could try to do this:

    // element gets converted to a string
    var data = {},
        element = document.getElementById("my-div");

    data[element] = metadata;

Unfortunately, `element` will be converted into the string `"[Object HTMLDivElement]"` or something similar (the exact values may be different depending on the browser). That's problematic because every `<div>` element gets converted into the same string, meaning you will constantly be overwriting the same key even though you're technically using different elements. For this reason, the `Map` type is a welcome addition to JavaScript.

The ECMAScript 6 `Map` type is an ordered list of key-value pairs where both the key and the value can be of any type. A key of `5` is different than a key of `"5"`, and keys are determined to be the same using `Object.is()`. You can store and retrieve data from a map using the `set()` and `get()` methods, respectively:

    var map = new Map();
    map.set("name", "Nicholas");
    map.set(document.getElementById("my-div"), { flagged: false });

    // later
    var name = map.get("name"),
        meta = map.get(document.getElementById("my-div"));

In this example, two key-value pairs are stored. The key `"name"` stores a string while the key `document.getElementById("my-div")` is used to associate meta data with a DOM element. If the key doesn't exist in the map, then the special value `undefined` is returned when calling `get()`.

Maps shared a couple of methods with sets, such as `has()` for determining if a key exists in the map and `delete()` for removing a key-value pair from the map. You can also use `size` to determine how many items are in the map:

    var map = new Map();
    map.set("name", "Nicholas");

    console.log(map.has("name"));   // true
    console.log(map.get("name"));   // "Nicholas"
    console.log(map.size);        // 1

    map.delete("name");
    console.log(map.has("name"));   // false
    console.log(map.get("name"));   // undefined
    console.log(map.size);        // 0

In order to make it easier to add large amounts of data into a map, you can pass an array of arrays to the `Map` constructor. Internally, each key-value pair is stored as an array with two items, the first being the key and the second being the value. The entire map, therefore, is an array of these two-item arrays and so maps can be initialized using that format:

    var map = new Map([ ["name", "Nicholas"], ["title", "Author"]]);

    console.log(map.has("name"));   // true
    console.log(map.get("name"));   // "Nicholas"
    console.log(map.has("title"));  // true
    console.log(map.get("title"));  // "Author"
    console.log(map.size);        // 2

When you want to work with all of the data in the map, you have several options. There are actually three generator methods to choose from: `keys`, which iterates over the keys in the map, `values`, which iterates over the values in the map, and `items`, which iterates over key-value pairs by returning an array containing the key and the value (`items` is the default iterator for maps). The easiest way to make use of these is to use a `for-of` loop:

    for (let key of map.keys()) {
        console.log("Key: %s", key);
    }

    for (let value of map.values()) {
        console.log("Value: %s", value);
    }

    for (let item of map.items()) {
        console.log("Key: %s, Value: %s", item[0], item[1]);
    }

    // same as using map.items()
    for (let item of map) {
        console.log("Key: %s, Value: %s", item[0], item[1]);
    }

When iterating over keys or values, you receive a single value each time through the loop. When iterating over items, you receive an array whose first item is the key and the second item is the value.

Another way to iterate over items is to use the `forEach()` method. This method works in a similar manner to `forEach()` on arrays. You pass in a function that gets called with three arguments: the value, the key, and the map itself. For example:

    map.forEach(function(value, key, map)) {
        console.log("Key: %s, Value: %s", key, value);
    });

Also similar to the arrays version of `forEach()`, you can pass in an optional second argument to specify the `this` value to use inside the callback:

    var reporter = {
        report: function(key, value) {
            console.log("Key: %s, Value: %s", key, value);
        }
    };

    map.forEach(function(value, key, map) {
        this.report(key, value);
    }, reporter);

Here, the `this` value inside of the callback function is equal to `reporter`. That allows `this.report()` to work correctly.

Compare this to the clunky way of iterating over values and a regular object:

    for (let key in object) {

        // make sure it's not from the prototype!
        if (object.hasOwnProperty(key)) {
            console.log(&quot;Key: %s, Value: %s&quot;, key, object[key]);
        }

    }

When using objects as maps, it was always a concern that properties from the prototype might leak through in a `for-in` loop. You always need to use `hasOwnProperty()` to be certain that you are getting only the properties that you wanted. Of course, if there were methods on the object,  you would also have to filter those:

    for (let key in object) {

        // make sure it's not from the prototype or a function!
        if (object.hasOwnProperty(key) && typeof object[key] !== "function") {
            console.log(&quot;Key: %s, Value: %s&quot;, key, object[key]);
        }

    }

The iteration features of maps allow you to focus on just the data  without worrying about extra pieces of information slipping into your code. This is another big benefit of maps over regular objects for storing key-value pairs.

### Browser Support

Both Firefox and Chrome have implemented `Map`, however, in Chrome you need to manually enable ECMAScript 6 features: go to `chrome://flags` and enable "Experimental JavaScript Features". Both implementations are incomplete. Neither browser implements any of the generator method for use with `for-of` and Chrome's implementation is missing the `size` method (which is part of the ECMAScript 6 draft specification).

## Weakmaps

Weakmaps are similar to regular maps in that they map a value to a unique key. That key can later be used to retrieve the value it identifies. Weakmaps are different because the key must be an object and cannot be a primitive value. This may seem like a strange constraint but it's actually the core of what makes weakmaps different and useful.

A weakmap holds only a weak reference to a key, which means the reference inside of the weakmap doesn't prevent garbage collection of that object. When the object is destroyed by the garbage collector, the weakmap automatically removes the key-value pair identified by that object. The canonical example for using weakmaps is to create an object related to a particular DOM element. For example, jQuery maintains a cache of objects internally, one for each DOM element that has been referenced. Using a weakmap would allow jQuery to automatically free up memory associated with a DOM element when it is removed from the document.

The ECMAScript 6 `WeakMap` type is an unordered list of key-value pairs where the key must be a non-null object and the value can be of any type. The interface for `WeakMap` is very similar to that of `Map` in that `set()` and `get()` are used to add data and retrieve data, respectively:

    var map = new WeakMap(),
        element = document.querySelector(".element");

    map.set(element, "Original");

    // later
    var value = map.get(element);
    console.log(value);             // "Original"

    // later still - remove reference
    element.parentNode.removeChild(element);
    element = null;

    value = map.get(element);
    console.log(value);             // undefined

In this example, one key-value pair is stored. The key is a DOM element used to store a corresponding string value. That value was later retrieved by passing in the DOM element to `get()`. If the DOM element is then removed from the document and the variable referencing it is set to `null`, then the data is also removed from the weakmap and the next attempt to retrieve data associated with the DOM element fails.

This example is a little bit misleading because the second call to `map.get(element)` is using the value of `null` (which `element` was set to) rather than a reference to the DOM element. You can't use `null` as a key in weakmaps, so this code isn't really doing a valid lookup. Unfortunately, there is no part of the interface that allows you to query whether or not a reference has been cleared (because the reference no longer exists).

A> The weakmap `set()` method will throw an error if you try to use a primitive value as a key. If you want to use a primitive value as a key, then it's best to use `Map` instead.

Weakmaps also have `has()` for determining if a key exists in the map and `delete()` for removing a key-value pair.

    var map = new WeakMap(),
        element = document.querySelector(".element");

    map.set(element, "Original");

    console.log(map.has(element));   // true
    console.log(map.get(element));   // "Original"

    map.delete(element);
    console.log(map.has(element));   // false
    console.log(map.get(element));   // undefined

Here, a DOM element is once again used as the key in a weakmap. The `has()` method is useful for checking to see if a reference is currently being used as a key in the weakmap. Keep in mind that this only works when you have a non-null reference to a key. The key is  forcibly removed from the weakmap by using `delete()`, at which point `has()` returns `false` and `get()` returned `undefined`.

### Uses and Limitations

Weakmaps have a very specific use case in mind, and that is mapping values to objects that might disappear in the future. The ability to free up memory related to these objects is useful for JavaScript libraries that wrap DOM elements with custom objects such as jQuery and YUI. There'll likely be more use cases discovered once implementations are complete and widespread, but in the short term, don't feel bad if you can't figure out a good spot for using weakmaps.

In many cases, a regular map is probably what you want to use. Weakmaps are limited in that they aren't enumerable and you can't keep track of how many items are contained within. There also isn't a way to retrieve a list of all keys. If you need this type of functionality, then you'll need to use a regular map.  If you don't, and you only intend to use objects as keys, then a weakmap may be the right choice.

### Browser Support

Both Firefox and Chrome have implemented `WeakMap`, however, in Chrome you need to manually enable ECMAScript 6 features: go to `chrome://flags` and enable "Experimental JavaScript Features". Both implementations are complete per the current strawman specification (weakmaps are not yet in the ECMAScript 6 draft).

## Summary

TODO

ECMAScript 6 sets are a welcome addition to the language. They allow you to easily create a collection of unique values without worrying about type coercion. You can add and remove items very easily from a set even though there is no direct access to items in the set. It's still possible, if necessary, to iterate over items in the set by using the ECMAScript 6 `for-of` statement.

Since ECMAScript 6 is not yet complete, it's also possible that the implementation and specification might change before other browsers start to include `Set`. At this point in time, it is still considered experimental API and shouldn't be used in production code. This post, and other posts about ECMAScript 6, are only intended to be a preview of functionality that is to come.
