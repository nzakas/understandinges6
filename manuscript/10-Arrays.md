# Arrays

Arrays have been one of the foundational JavaScript objects since the language's early days. Unfortunately, arrays remained the same for most of their existence until ECMAScript 5 introduced some new array methods. ECMAScript 6 continues the trend by updating arrays with a lot more functionality.

## Creating Arrays

Prior to ECMAScript 6, the two primary ways of creating arrays were the `Array` constructor and array literal syntax. Both approaches require you to list out the array items individually and are otherwise fairly limited. If you had an array-like object (an object with numeric indices and a `length` property) and wanted to convert it into an array, your options were fairly limited and often required extra code. While these limitations might seem small, they turned out to be a growing pain point for large JavaScript applications that do a lot of array manipulation. To makes things easier, ECMAScript 6 adds two new methods: `Array.of()` and `Array.from()`.

### Array.of()

JavaScript has long had a quirk around creating arrays with the `Array` constructor. The behavior of `new Array()` behaves differently based on the type and number of arguments passed into it. For example:

```js
let items = new Array(1, 2);        // length is 2
console.log(items.length);          // 2
console.log(items[0]);              // 1
console.log(items[1]);              // 2

items = new Array(2);
console.log(items.length);          // 2
console.log(items[0]);              // undefined
console.log(items[1]);              // undefined

items = new Array(3, "2");
console.log(items.length);          // 2
console.log(items[0]);              // 3
console.log(items[1]);              // "2"

items = new Array("2");
console.log(items.length);          // 1
console.log(items[0]);              // "2"
```

When the `Array` constructor is passed a single numeric value, that value is set to be the `length` of the array; if a single non-numeric value is passed, then that value becomes the one and only item in the array; if multiple values are passed (numeric or not), then those values become items in the array. This behavior is both confusing and risky, as you may not always be aware of the type of data being passed.

ECMAScript 6 introduces `Array.of()` to solve this problem. `Array.of()` works in a manner that is similar to the `Array` constructor. The only difference is the removal of the special case regarding a single numeric value. The `Array.of()` method always creates an array containing its arguments regardless of the number of arguments or the argument types. Here are some examples:

```js
let items = Array.of(1, 2);         // length is 2
console.log(items.length);          // 2
console.log(items[0]);              // 1
console.log(items[1]);              // 2

items = Array.of(2);
console.log(items.length);          // 1
console.log(items[0]);              // 2

items = Array.of("2");
console.log(items.length);          // 1
console.log(items[0]);              // "2"
```

The `Array.of()` method is similar to using an array literal, which is to say, you can use an array literal instead of `Array.of()` for native arrays most of the time. If you ever need to pass the `Array` constructor into a function, then you might want to pass `Array.of()` instead to get consistent behavior. For example:

```js
function createArray(arrayCreator, value) {
    return arrayCreator(value);
}

let items = createArray(Array.of, value);
```

In this code, the `createArray()` function accepts an array creator function and a value to insert into the array. You can then pass `Array.of` as the first argument to `createArray()` to create a new array. It would be dangerous to pass `Array` directly if you cannot guarantee that `value` won't be a number.

Whereas `Array.of()` makes it easier to deal with creating arrays from individual items, the `Array.from()` method is used to create arrays from already-existing data structures.

I> While `Array.of()` might seem like a small addition to the language, it's much more powerful when used with derived array classes. This is discussed later in the chapter.


### Array.from()

One of the more cumbersome tasks in JavaScript has long been converting nonarray objects into actual arrays. For instance, if you have an `arguments` object (which is array-like) and want to work with it as if it's an array, then you'd need to convert it first. In ECMAScript 5,  you'd write a function such as:

```js
function makeArray(arrayLike) {
    var result = [];

    for (var i = 0, len = arrayLike.length; i < len; i++) {
        result.push(arrayLike[i]);
    }

    return result;
}

function doSomething() {
    var args = makeArray(arguments);

    // use args
}
```

This approach manually creates an array and copies over each item from `arguments` into that new array. While that works, it's a decent amount of code to perform a relatively simple operation. As such, developers soon discovered that you could shorten the amount of code by using the native array `slice()` method on array-like objects:

```js
function makeArray(arrayLike) {
    return Array.prototype.slice.call(arrayLike);
}

function doSomething() {
    var args = makeArray(arguments);

    // use args
}
```

Even though this required less typing, it's not at all obvious that `Array.prototype.slice.call()` means "convert to an array." This works because you're setting the `this` value for `slice()` to the array-like object. Since `slice()` needs only numeric indices and a `length` property to function correctly, any array-like object will work.

The `Array.from()` method was added in ECMAScript 6 as a more obvious way of converting objects into arrays. You can pass either an iterable or an array-like object as the first argument and `Array.from()` returns an array. Here's a simple example:

```js
function doSomething() {
    var args = Array.from(arguments);

    // use args
}
```

The `Array.from()` call in this example creates a new array based on the items in `arguments`. So `args` is an instance of `Array` that contains the same values in the same positions as `arguments`.

### Mapping Conversion

If you want to take this conversion a step further, you can provide a second argument to `Array.from()` that is a mapping function used to convert each value into a final form. For example:

```js
function translate() {
    return Array.from(arguments, (value) => value + 1);
}

let numbers = translate(1, 2, 3);

console.log(numbers);               // 2,3,4
```

Here, `Array.from()` is used with a mapping function that adds one to each item in the array. If the mapping function is on an object, you can optionally pass a third argument to `Array.from()` that represents the `this` value for the mapping function, such as:

```js
let helper = {
    diff: 1,

    add(value) {
        return value + this.diff;
    }
};

function translate() {
    return Array.from(arguments, helper.add, helper);
}

let numbers = translate(1, 2, 3);

console.log(numbers);               // 2,3,4
```

This example uses the `helper.add()` method as the mapping function for the conversion. Since `helper.add()` uses `this.diff`, you need to provide the third argument to `Array.from()` specifying the value of `this`. In this way, `Array.from()` can easily handle conversion of data without needing to use `bind()` or another way of specifying the `this` value.

### Use on Iterables

The `Array.from()` method works on both array-like objects and iterables. That means any object with an `@@iterator` property can be converted into an array using `Array.from()`. For example:

```js
let numbers = {
    *[Symbol.iterator]() {
        yield 1;
        yield 2;
        yield 3;
    }
};

let numbers2 = Array.from(numbers, (value) => value + 1);

console.log(numbers2);              // 2,3,4
```

In this code, the `numbers` object is an iterable so it can be passed directly to `Array.from()` to convert its values into an array. The mapping function adds one to each number so the resulting array contains 2, 3, and 4 instead of 1, 2, and 3.

I> If an object is both array-like and iterable, then the iterator is used by `Array.from()` to determine the values to convert.





## Changes

Array.prototype.concat/push/splice throw TypeError if new length would be 2^53 or greater.


TODO

## New Methods

TODO



## Species Pattern

TODO











Additionally, `MyArray` inherits all static members from `Array`, which means that `MyArray.of()` already exists and can be used:

```js
class MyArray extends Array {
    // ...
}

var colors = MyArray.of(["red", "green", "blue"]);
console.log(colors instanceof MyArray);     // true
```

The inherited `MyArray.of()` behaves the same as `Array.of()` except that it creates an instance of `MyArray` rather than an instance of `Array`. Many built-in objects are specifically defined so that their static methods work appropriately for derived classes.

This same approach can be used to inherit from any of the built-in JavaScript objects, with a full guarantee that it will work the same way as the built-ins.

### @@species

TODO TODO


The `@@species` property of an object, represented by `Symbol.species` in code, contains a reference to the constructor function used to create that object. For example:

```js
let book = {
    title: "Understanding ES6"
};

var species = book[Symbol.species];

console.log(Object === species);        // true
```


## Typed Arrays

TODO
