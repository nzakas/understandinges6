# Iterators and Generators

Iterators have been used in many programming languages as a way to more easily work with collections of data. In ECMAScript 6, JavaScript adds iterators as an important feature of the language. When coupled with new array methods and new types of collections (such as sets and maps), iterators become even more important for efficient processing of data.

## What are Iterators?

Iterators are nothing more than objects with a certain interface. That interface consists of a method called `next()` that returns a result object. The result object has two properties, `value`, which is the next value, and `done`, which is a boolean value that's `true` when there are no more values to return. The iterator keeps an internal pointer to a location within a collection of values and, with each call to `next()`, returns the next appropriate value.

If you call `next()` after the last value has been returned, the method returns `done` as `true` and `value` contains the return value for the iterator. The *return value* is not considered part of the data set, but rather a final piece of related data or `undefined` if no such data exists. (This concept will become clearer in the generators section later in this chapter.)

With that understanding, it's fairly easy to create an iterator using ECMAScript 5, for example:

```js
function createIterator(items) {

    let i = 0;

    return {
        next: function() {

            let done = (i >= items.length);
            let value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };

        }
    };
}

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// for all further calls
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

The `createIterator()` function in this example returns an object with a `next()` method. Each time the method is called, the next value in the `items` array is returned as `value`. When `i` is 4, `items[i++]` returns `undefined` and `done` is `true`, which fulfills the special last case for iterators in ECMAScript 6.

ECMAScript 6 makes use of iterators in a number of places to make dealing with collections of data easier, so having a good basic understanding allows you to better understand the language as a whole.

## for-of

The first place you'll see iterators in ECMAScript 6 is with the new `for-of` loop. The `for-of` loop works with iterators to return each successive value. The loop itself calls `next()` behind the scenes and exits when the `done` property of the returned object is `true`. For example:

```js
let iterator = createIterator([1, 2, 3]);

for (let i of iterator) {
    console.log(i);
}
```

This code outputs the following:

```
1
2
3
```

The `for-of` loop in this example is calling `iterator.next()` and assigning the variable `i` to the value returned on the `value` property. So `i` is first 1, then 2, and finally 3. When `done` is `true`, the loop exits, so `i` is never assigned the value of `undefined`.

W> The `for-of` statement will throw an error when used on a `null` or `undefined` iterator.

## Generators

You might be thinking that iterators sound interesting but they look like a bunch of work. Indeed, writing iterators so that they adhere to the correct behavior is a bit difficult, which is why ECMAScript 6 provides generators. A *generator* is a special kind of function that returns an iterator. Generator functions are indicated by inserting a star character (`*`) after the `function` keyword (it doesn't matter if the star is directly next to `function` or if there's some whitespace between them). The `yield` keyword is used inside of generators to specify the values that the iterator should return when `next()` is called. So if you want to return three different values for each successive call to `next()`, you can do so as follows:

For example:

```js
// generator
function *createIterator() {
    yield 1;
    yield 2;
    yield 3;
}

// generators are called like regular functions but return an iterator
let iterator = createIterator();

for (let i of iterator) {
    console.log(i);
}
```

This code outputs the following:

```
1
2
3
```

In this example, the `createIterator()` function is a generator (as indicated by the `*` before the name) and it's called like any other function. The value returned is an object that adheres to the iterator pattern. Multiple `yield` statements inside the generator indicate the progression of values that should be returned when `next()` is called on the iterator. First, `next()` should return `1`, then `2`, and then `3` before the iterator is finished.

Perhaps the most interesting aspect of generator functions is that they stop execution after each `yield` statement, so `yield 1` executes and then the function doesn't execute anything else until the iterator's `next()` method is called. At that point, execution resumes with the next statement after `yield 1`, which in this case is `yield 2`. This ability to stop execution in the middle of a function is extremely powerful and lends to some interesting uses of generator functions (discussed later in this chapter).

The `yield` keyword can be used with any value or expression, so you can do interesting things like use `yield` inside of a `for` loop:

```js
function *createIterator(items) {
    for (let i=0; i < items.length; i++) {
        yield items[i];
    }
}

let iterator = createIterator([1, 2, 3]);

for (let i of iterator) {
    console.log(i);
}
```

In this example, an array is iterated over, yielding each item as the loop progresses. Each time `yield` is encountered, the loop stops, and each time `next()` is called on `iterator`, the loop picks back up where it left off.

Of course, you can still call `iterator.next()` directly:

```
function *createIterator(items) {
    for (let i=0; i < items.length; i++) {
        yield items[i];
    }
}

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// for all further calls
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

Of course, collections such as arrays naturally lend themselves to iteration, and that's why ECMAScript 6 has a number of built-in iterators.

## Built-in Iterators

Another way that ECMAScript 6 makes using iterators easier is by making iterators available on many objects by default. You don't actually need to create your own iterators for many of the built-in types because the language has them already. You only need to create iterators when you find that the built-in ones don't serve your purpose.

### Collection Iterators

The ECMAScript 6 collection objects, arrays, maps, and sets, all have three default iterators to help you navigate data. You can retrieve an iterator for the array by calling one of these methods:

* `entries()` - returns an iterator whose values are a key-value pair.
* `values()` - returns an iterator whose values are the values of the collection.
* `keys()` - returns an iterator whose values are the keys contained in the collection.

The `entries()` iterator actually returns a two-item array where the first item is the key and the second item is the value. For arrays, the first item is the numeric index; for sets, the first item is also the value (since values double as keys in sets). Here are some examples:

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let entry of colors.entries()) {
    console.log(entry);
}

for (let entry of tracking.entries()) {
    console.log(entry);
}

for (let entry of data.entries()) {
    console.log(entry);
}
```

This example outputs the following:

```
[0, "red"]
[1, "green"]
[2, "blue"]
[1234, 1234]
[5678, 5678]
[9012, 9012]
["title", "Understanding ECMAScript 6"]
["format", "ebook"]
```

The `values()` iterator simply returns the values as they are stored in the collection. For example:

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let value of colors.values()) {
    console.log(value);
}

for (let value of tracking.values()) {
    console.log(value);
}

for (let value of data.values()) {
    console.log(value);
}
```

This example outputs the following:

```
"red"
"green"
"blue"
1234
5678
9012
"Understanding ECMAScript 6"
"ebook"
```

In this case, using `values()` returns the exact data contained in the `value` property returned from `next()`.

The `keys()` iterator returns each key present in the collection. For arrays, this is the numeric keys only (it never returns other own properties of the array); for sets, the keys are the same as the values and so `keys()` and `values()` return the same iterator.

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let key of colors.keys()) {
    console.log(key);
}

for (let key of tracking.keys()) {
    console.log(key);
}

for (let key of data.keys()) {
    console.log(key);
}
```

This example outputs the following:

```
0
1
2
1234
5678
9012
"title"
"format"
```

Additionally, each collection type has a default iterator that is used by `for-of` whenever an iterator isn't explicitly specified. The default iterator for array and set is `values()` while the default iterator for maps is `entries()`. This makes it a little bit easier to use collection objects in `for-of`:

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

// same as using colors.values()
for (let value of colors) {
    console.log(value);
}

// same as using tracking.values()
for (let num of tracking) {
    console.log(num);
}

// same as using data.entries()
for (let entry of data) {
    console.log(entry);
}
```

This example outputs the following:

```
"red"
"green"
"blue"
1234
5678
9012
["title", "Understanding ECMAScript 6"]
["format", "ebook"]
```

### String Iterators

TODO

### NodeList Iterators

TODO

## Advanced Functionality

TODO

### Passing Arguments to Iterators

TODO

### Delegating Generators

TODO

### Asynchronous Task Scheduling

TODO

## Summary

TODO
