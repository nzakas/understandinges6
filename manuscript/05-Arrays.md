# Arrays

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

## Changes

Array.prototype.concat/push/splice throw TypeError if new length would be 2^53 or greater.


TODO

## New Methods

TODO



## Species Pattern

TODO

## Creating Arrays




### Array.of()

### Array.from()

One of the more cumbersome tasks in JavaScript has long been converting array-like objects into actual arrays. For instance, if you have an `arguments` object (which is array-like) and want to work with it as if it's an array, then you'd need to convert it first. Traditionally,  you'd write a function such as:

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

However, developers soon discovered that you could shorten the amount of code by using the native array `slice()` method:

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

The `Array.from()` method was added in ECMAScript 6 as a more obvious way of converting array-like objects into arrays. Simple usage is as follows:

```js
function doSomething() {
    var args = Array.from(arguments);

    // use args
}
```

The `Array.from()` call in this example creates a new array based on the items in `arguments`. So `args` is an instance of `Array` that contains the same values in the same positions as `arguments`.

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

The `Array.from()` method works on both array-like objects and iterables. That means any object with an `@@iterator` property can be converted into an array using `Array.from()`. For example:

```js
let collection = {
    items: [],
    *[Symbol.iterator]() {
        yield *this.items.values();
    }

};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

let numbers = Array.from(collection, (value) => value + 1);

console.log(numbers);               // 2,3,4
```

In this code, the `collection` object is an iterable so it can be passed directly to `Array.from()` to convert its values into an array.

I> If an object is both array-like and iterable, then the iterator is used by `Array.from()` to determine the values to convert.

## The Spread Operator

TODO

Also works on strings.

## Typed Arrays

TODO

