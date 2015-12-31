# Sets and Maps

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

Here, `map[key2]` is referencing the same value as `map[key1]`. Each object, `key1` and `key2`, are converted to strings because object properties must be strings. The default string representation for objects is `"[object Object]"`, so both `key1` and `key2` are converted to that string.

Another problem relates specifically to maps with a key whose value is falsy. A falsy value is automatically converted to false when used in situations where a boolean value is required, such as in the condition of an `if` statement. This alone isn't a problem so long as you're careful as to how you use values. For instance:

```js
let map = Object.create(null);

map.count = 1;

// checking for the existence of "count" or a nonzero value?
if (map.found) {
    // ...
}
```

This code has some ambiguity as to the usage of `map.count`, is the `if` statement intended to check for the existence of `map.count` or that the value is nonzero? The code inside the `if` statement will execute because the value 1 is truthy. However, if `map.count` is 0, or if `map.count` doesn't exist, the code inside the `if` statement would not be executed.

These are difficult problem to identify and debug when it occurs in large applications, and a prime reason that ECMAScript 6 adds both sets and maps to the language.

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

You can test to see which values are in the set using the `has()` method:

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true
console.log(set.has(6));    // false
```

It's possible to remove a single value from a set by using the `delete()` method or you can remove all values from the set by using `clear()`:

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true

set.delete(5);

console.log(set.has(5));    // false
console.log(set.size);      // 1

set.clear();

console.log(set.has("5"));  // false
console.log(set.size);      // 0
```

All of this amounts to a very easy mechanism for tracking unique ordered values. However, what if you want to add items into a set and then perform some operation on each item? That's where the `forEach()` method comes in.

### The forEach() Method

If you're using to working with arrays, then you may already be familiar with the `forEach()` method. ECMAScript 5 added `forEach()` to arrays to create an easier way to work on each item in an array without setting up a `for` loop. The method proved to be popular amongst developers and so the same method is available on sets and works the same way.

The `forEach()` method is passed a callback function that accepts three arguments:

1. The value from the next position in the set
1. The same value as the first argument
1. The set from which the value is read

The strange difference between the set version of `forEach()` and the array version is that the first and second arguments to the callback function are the same. While this might look like a mistake, there's a good reason for this behavior.

The other objects that have `forEach()` methods, arrays and maps, pass three arguments to their callback functions. The first two arguments for arrays and maps are the value and the key (the numeric index for arrays). Sets do not have keys, so you could either make the callback function accept two arguments (which would make it different from the others) or find a way to keep the callback function the same and accept three arguments. The decision was made to do the latter, and so sets consider each value to be both the key and the value. As such, the first and second argument are always the same in `forEach()` on sets in order to keep this functionality consistent with the other `forEach()` methods on arrays and maps.

Other than the difference in arguments, using `forEach()` is basically the same for a set as it is for an array:

```js
let set = new Set([1, 2]);

set.forEach(function(value, key, ownerSet) {
    console.log(key + " " + value);
    console.log(ownerSet === set);
});
```

This outputs:

```
1 1
true
2 2
true
```

Also the same as arrays, you can pass a `this` value as the second argument to `forEach()` if you need to use `this` in your callback function:

```js
let set = new Set([1, 2]),
    processor = {
        output(value) {
            console.log(value);
        },
        process(dataSet) {
            dataSet.forEach(function(value) {
                this.output(value);
            }, this);
        }
    };

processor.process(set);
```

In this example, the `processor.process()` method calls `forEach()` on the set and passes `this` as the `this` value for the callback. That's necessary so `this.output()` will correctly resolve to `processor.output()`. You can also use an arrow function to get the same effect without passing the second argument:

```js
let set = new Set([1, 2]),
    processor = {
        output(value) {
            console.log(value);
        },
        process(dataSet) {
            dataSet.forEach((value) => this.output(value));
        }
    };

processor.process(set);
```

The arrow function in this example reads `this` from the containing function and so it correctly resolves `this.output()` to `processor.output()`.

Keep in mind that while sets are great for tracking values and `forEach()` lets you work on each value sequentially, you can't randomly access a value contained in a set like you can with an array. If you need to do so, then the best option is to convert the set into an array.

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

The biggest difference between weak sets and regular sets is the weak reference held to the object value. Here's an example of what that means:

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

From these examples, you can see that weak sets share some characteristics with regular sets. The key differences are:

1. The `add()`, `has()`, and `delete()` methods throw an error when passed a non-object.
1. Weak sets are not iterables and therefore cannot be used in a `for-of` loop.
1. Weak sets do not expose any iterators (such as `keys()` and `values()`), so there is no way to programmatically determine the contents of a weak set.
1. Weak sets do not have a `forEach()` method.

The seemingly limited functionality of weak sets is necessary in order to properly handle memory. In general, if you only need to track object references, then you should use a weak set instead of a regular set.

Sets give you a new way to handle lists of values, but they aren't useful when you need to associate additional information with those values. That's why ECMAScript 6 also adds maps.

## Maps in ECMAScript 6

The ECMAScript 6 `Map` type is an ordered list of key-value pairs where both the key and the value can be of any type. Keys are considered to be the same by using `Object.is()`, so you can have both a key of `5` and a key of `"5"` because they are different types. This is quite different than using object properties as keys, which always coerce values into strings.

Items are added to maps by using the `set()` method and passing in the key and the value to associate with the key. You can later retrieve a value by passing the key to `get()`. For example:

```js
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2016);

console.log(map.get("title"));      // "Understanding ES6"
console.log(map.get("year"));       // 2016
```

In this example, two key-value pairs are stored. The key `"title"` stores a string while the key `"year"` stores a number. These are later retrieved using `get()`. If the key doesn't exist in the map, then the special value `undefined` is returned when calling `get()`.

You can also use objects as keys, something that isn't possible using object properties. Here's an example:

```js
let map = new Map(),
    key1 = {},
    key2 = {};

map.set(key1, 5);
map.set(key2, 42);

console.log(map.get(key1));         // 5
console.log(map.get(key2));         // 42
```

This code uses the objects `key1` and `key2` as keys in the map to store two different values. Because these keys are not coerced into another form, each object is considered unique. This allows you to associate additional data to an object without modifying the object itself.

### Map Methods

Maps share several methods with sets, and that is intentional to allow maps and sets to be interacted with in similar ways. These three methods are available on both maps and sets:

* `has(key)` - determines if the given key exists in the map
* `delete(key)` - removes the key and its associated value from the map
* `clear()` - removes all keys and values from the map

Additionally, maps have a `size` property that indicates how many key-value pairs it contains. Here's an example:

```js
let map = new Map();
map.set("name", "Nicholas");
map.set("age", 25);

console.log(map.size);          // 2

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"

console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25

map.delete("name");
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.size);          // 1

map.clear();
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.has("age"));    // false
console.log(map.get("age"));    // undefined
console.log(map.size);          // 0

```

As with sets, the `size` property always contains the number of key-value pairs in the map. This example starts with two keys `"name"` and `"age"`, so `has()` returns `true` when passed either key. The `"name"` is then removed by using `delete()`, so `has()` returns `false` when passed `"name"` and the `size` property indicates one less item. The `clear()` method then removes the remaining key, as indicated by `has()` returning `false` for both keys and `size` being 0.

The `clear()` method is a fast way to remove a lot of data from a map, but there's also a way to add a lot of data to a map at one time.

### Map Initialization

Also similar to sets, you can initialize a map with data by passing an array to the `Map` constructor. Each item in the array must itself be an array where the first item is the key and the second is the value. The entire map, therefore, is an array of these two-item arrays, for example:

```js
let map = new Map([ ["name", "Nicholas"], ["age", 25]]);

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25
console.log(map.size);          // 2
```

The keys `"name"` and `"age"` are added into the map through initialization in the constructor. While the array-of-arrays may look a bit strange, it is necessary in order to accurately represent keys as they can be any data type. Storing these keys in an array is the only way to ensure that they are not coerced into another data type before storage.

### The forEach Method

The `forEach()` method for maps is similar to `forEach()` for sets and arrays in that it accepts a callback function that receives three arguments:

1. The value from the next position in the map
1. The key for that value
1. The map from which the value is read

The callback arguments more closely match the `forEach()` behavior in arrays where the first argument is the value and the second is the key (numeric index in arrays). Here's an example:

```js
let map = new Map([ ["name", "Nicholas"], ["age", 25]]);

map.forEach(function(value, key, ownerMap) {
    console.log(key + " " + value);
    console.log(ownerMap === map);
});
```

This outputs:

```
name Nicholas
true
age 25
true
```

The callback receives each key-value pair in the order in which they were inserted, which is slightly different from arrays, where the callback receives each item in order of numeric index.

I> You can also provide a second argument to `forEach()` to specify the `this` value inside of the callback function. This behaves the same as with the set version of `forEach()`.

### Weak Maps

Weak maps are to maps what weak sets are to sets, which is a way to store weak object references. In weak maps, every key must be an object (and error is thrown if you try to use a non-object key), and those object references are held weakly so as not to interfere with garbage collection. When there are no other references to a weak map key, the key-value pair is removed from the weak map.

The best example for using weak maps is to create an object related to a particular DOM element in a web page. For example, some JavaScript libraries for web pages maintain one custom object for every DOM element that is referenced through the library, and that mapping is stored in a cache of objects internally. The difficult part of this approach is determining when a DOM element no longer exists in the web page so that the library can remove its associated object (and most importantly, not hold onto the DOM element reference and cause a memory leak). Using a weak map would allow the library to associate a custom object with every DOM element and then automatically destroy that object when the DOM element no longer exists.

I> It's important to note that only weak map keys, and not weak map values, are weak references. An object stored as a weak map value will prevent garbage collection if all other references are removed.

#### Using Weak Maps

The ECMAScript 6 `WeakMap` type is an unordered list of key-value pairs where the key must be a non-null object and the value can be of any type. The interface for `WeakMap` is very similar to that of `Map` in that `set()` and `get()` are used to add and retrieve data respectively:

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

let value = map.get(element);
console.log(value);             // "Original"

// remove the element
element.parentNode.removeChild(element);
element = null;

// the weak map is empty at this point
```

In this example, one key-value pair is stored. The key is a DOM element used to store a corresponding string value. That value is then retrieved by passing in the DOM element to `get()`. If the DOM element is then removed from the document and the variable referencing it is set to `null`, then the data is also removed from the weak map.

Similar to weak sets, there is no way to verify that the weak map is empty because it doesn't have a `size` property. Because there are no remaining references to the key, you can use `get()` to attempt to retrieve the value. The weak map has cut off access to the value for that key and, when the garbage collector runs, that memory will be freed.

#### Weak Map Initialization

Weak maps can be initialized the same way as regular maps: by passing an array of arrays to the `WeakMap` constructor. Once again, each item in the array should be a two-item array where the first item is the key (an object) and the second item is the value (any data type) . For example:

```js
let key1 = {},
    key2 = {},
    map = new WeakMap([ [key1, "Hello"], [key2, 42]]);

console.log(map.has(key1));     // true
console.log(map.get(key1));     // "Hello"
console.log(map.has(key2));     // true
console.log(map.get(key2));     // 42
```

The objects `key1` and `key2` are used as keys in the weak map and they can be accessed using `get()` and `has()`. An error is thrown if there is a non-object key in data passed to `WeakMap`.

#### Weak Map Methods

Weak maps have only a couple of additional methods available to interact with its items. There is a `has()` method to determine if the given key exists in the map and a `delete()` method to remove a specific key-value pair. There is no `clear()` method because that would require enumerating keys, and like weak sets, that is not possible with weak maps. Here's an example:

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

Here, a DOM element is once again used as the key in a weak map. The `has()` method is useful for checking to see if a reference is currently being used as a key in the weak map. Keep in mind that this only works when you have a non-null reference to a key. The key is forcibly removed from the weak map by using `delete()`, at which point `has()` returns `false` and `get()` returns `undefined`.

#### Private Object Data

While most developers consider the main use case of weak maps to be associated data with DOM elements, there are many possible uses (and no doubt, some that have yet to be discovered). One practical use of weak maps is to store data that is private to object instances. All object properties are public in ECMAScript 6 and so you need to use some creativity to make data access to objects but not accessible to everything. Consider the following example:

```js
function Person(name) {
    this._name = name;
}

Person.prototype.getName = function() {
    return this._name;
};
```

This code uses the common convention of a leading underscore to indicate that a property is considered private and should not be modified outside of the object instance. The intent is to use `getName()` to read `this._name` and not allow that value to be changed. However, there is nothing standing in the way of someone writing to the `_name` property, so it can be overwritten either intentionally or accidentally.

Using ECMAScript 5, it's possible to get close to having truly private data using a pattern such as:

```js
let Person = (function() {

    let privateData = {},
        privateId = 0;

    function Person(name) {
        Object.defineProperty(this, "_id", { value: privateId++ });

        privateData[this._id] = {
            name: name
        };
    }

    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };

    return Person;
}());
```

This example wraps the definition of `Person` is an IIFE that contains two private variables, `privateData` and `privateId`. The `privateData` object stores private information for each instance while `privateId` is used to generate a unique ID for each instance. When the `Person` constructor is called, a `_id` property is added so that it's nonenumerable, nonconfigurable, and nonwritable (mean it can't accidentally be overwritten). Then, a entry is made into the `privateData` object that corresponds to the ID for the object instance; that's where the name information is stored. Later, in `getName()`, the name can be restored by using `this._id` as the key into `privateData`. Because `privateData` is not accessible outside of the IIFE, the actual data is safe even though `this._id` is exposed publicly. The big problem with this approach is that the data in `privateData` always remains because there is no way to know when an object instance is destroyed, meaning that extra data will always be left in `privateData`. This problem can be solved by using a weak map instead:

```js
let Person = (function() {

    let privateData = new WeakMap();

    function Person(name) {
        privateData.set(this, { name: name });
    }

    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };

    return Person;
}());
```

This version of the code uses a weak map for the private data instead of an object. Because the object instance itself can be used as a key, there's no need to keep track of a separate ID. When the `Person` constructor is called, a new entry is made into the weak map with a key of `this` and a value of an object containing private information (in this case, just `name`). That information is retrieve inside of `getName()` by passing `this` to `privateData.get()` in order to retrieve the data object and access the `name` property. In this way, the private information is kept private and will be destroyed whenever an object instance is destroyed.

#### Uses and Limitations

When deciding whether to use a weak map or a regular map, the primary decision is whether you want to use only object keys. Anytime you're going to use only object keys then the best choice is a weak map. That will allow you to optimize memory usage and avoid memory leaks by ensuring that extra data isn't kept around after it's no longer accessible.

Keep in mind that weak maps give you very little visibility into their contents, so you can't use `forEach()`, `size`, or `clear()` to manage the items. If you need some inspection capabilities, then regular maps are a better choice. Just be sure to keep an eye on memory usage.

Of course, if you only want to use non-object keys, then regular maps are your only choice.

## Summary

ECMAScript 6 formally introduces sets and maps into the language. Prior to this, developers frequently used objects to mimic both sets and maps, often running into problems due to the limitations associated with object properties.

Sets are unordered lists of unique values. Values are considered unique if they are not equivalent according to `Object.is()`. Sets automatically remove duplicate values, so you can use a set to filter an array for duplicates and return the result. Sets aren't subclasses of arrays, so you cannot randomly access its values. Instead, you can use the `has()` method to determine if a value is contained in the set and the `size` property to inspect the number of values in the set. There's also a `forEach()` method to process each set value.

Weak sets are a special type of set that can contain only objects. The objects are stored with weak references, meaning that they will not block garbage collection if it is the only remaining reference. It's not possible to inspect weak set contents due to the complexities of memory management, so it's best to use them only for tracking objects that need to be grouped together.

Maps are unordered key-value pairs where the key can be any data type. Similar to sets, duplicate keys are determined by `Object.is()`, which means you can have a numeric key `5` and a string `"5"` as two separate keys. Any data type can be used for a value to associate with the key using `set()` and the value can later be retrieved by using `get()`. Maps also have a `size` property and a `forEach()` method to allow for easier item access.

Weak maps are a special type of map that can only have object keys. As with weak sets, these object key references are weak and do no prevent garbage collection when they are the only remaining reference. When a key is garbage collected, the value associated with it is also removed from the weak map. This memory management aspect makes weak maps uniquely suited for correlating additional information with objects whose lifecycles are managed outside of the code accessing them.
