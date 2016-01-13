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

The `Array.of()` method uses the `@@species` property (discussed in Chapter 9) to determine the type of return value. That means derived array classes inherit a static `of()` method that returns an instance of the derived class. For example:

```js
class MyArray extends Array {
    // empty
}

let items = MyArray.of(1, 2);

console.log(items instanceof MyArray);      // true
```

The `MyArray` derived class inherits `of()` from `Array` and returns an instance of `MyArray`. If you want `MyArray.of()` to return an instance of `Array`,  you can do so by overriding `@@species`, such as:

```js
class MyArray extends Array {
    static get [Symbol.species]() {
        return Array;
    }
}

let items = MyArray.of(1, 2);

console.log(items instanceof MyArray);      // false
console.log(items instanceof Array);        // true
```

This example ensures that `MyArray.of()` returns an instance of `Array` by overriding the `@@species` property. Keep in mind that doing so also changes any other inherited method that returns an array.

Whereas `Array.of()` makes it easier to deal with creating arrays from individual items, the `Array.from()` method is used to create arrays from already-existing data structures.

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

I> `Array.from()` also uses `@@species` to determine the type of array to return.

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

## Array Size Limits

Most developers are unaware that arrays have a limit on the number of elements they can contain. The limit is set at 2^53 elements for each array, and in ECMAScript

## New Methods

Continuing the trend from ECMAScript 5, ECMAScript 6 adds several new methods to arrays. While the first two methods, `find()` and `findIndex()`, were meant to aid all developers, the others, `fill()` and `copyWithin()` are inspired largely by use cases for typed arrays (discussed later in this chapter).

### The find() and findIndex() Methods

Prior to ECMAScript 5, searching through arrays was cumbersome because there were no builtin methods for doing so. ECMAScript 5 added `indexOf()` and `lastIndexOf()`, finally allowing developers to search for specific values inside of an array. As big of an improvement as these two methods were, they were still fairly limited because you could only search for one value at a time (meaning if you wanted to find the first even number in a series of numbers, for example, you'd need to write your own code to do so). ECMAScript 6 solved that problem by introducing two new methods: `find()` and `findIndex()`.

The `find()` and `findIndex()` methods work the same way. They both accepts two arguments, a callback function and an optional value to use for `this` inside of that function. The callback function is passed an array element, the index of that element in the array, and the array itself (same arguments as methods such as `map()` and `forEach()`) and should return `true` if the given value matches some criteria. Both `find()` and `findIndex()` stop searching the array the first time the callback function returns `true`, and the only difference is that `find()` returns the value whereas `findIndex()` returns the index at which the value was found. Here's an example:

```js
let numbers = [25, 30, 35, 40, 45];

console.log(numbers.find(n => n > 33));         // 35
console.log(numbers.findIndex(n => n > 33));    // 2
```

This code uses both `find()` and `findIndex()` to locate the first value in the `numbers` array that is greater than 33. The call to `find()` returns 35 while `findIndex()` returns 2, the location of 35 in the `numbers` array.

Both `find()` and `findIndex()` are useful to find an array element that matches a condition rather than a value. If you only want to find a value, then `indexOf()` and `lastIndexOf()` are better choices.

### The fill() Method

The `fill()` method fills one or more array elements with a specific value. When passed a value, `fill()` overwrites all of the values in an array with that value. For example:

```js
let numbers = [1, 2, 3, 4];

numbers.fill(1);

console.log(numbers.toString());    // 1,1,1,1
```

Here, the call to `numbers.fill(1)` changes all of the values in `numbers` to `1`. If you only want to change some of the elements, rather than all of them, you can optionally include a start index and an exclusive end index, such as:

```js
let numbers = [1, 2, 3, 4];

numbers.fill(1, 2);

console.log(numbers.toString());    // 1,2,1,1

numbers.fill(0, 1, 3);

console.log(numbers.toString());    // 1,0,0,1
```

In this example, the last two array elements are filled with `1` by `numbers.fill(1, 2)` as `2` indicates the index at which to start filling elements. The end index is considered to be `numbers.length` because it isn't specified. The next operation, `numbers.fill(0, 1, 3)`, fills array elements at indices 1 and 2 with `0`. In this way, you're able to fill multiple array elements at once without overwriting the entire array.

I> If either the start or end index are negative, then those values are added to the array's length to determine the final location. For instance, a start location of `-1` means that the index will be `array.length-1` where `array` is the array on which `fill()` is called.

### The copyWithin() Method

The `copyWithin()` method is similar to `fill()` in that it changes multiple array elements at the same time. However, instead of specifying a single value to assign to array elements, `copyWithin()` lets you copy array element values from the array itself. To accomplish that, you need to pass two arguments to `copyWithin()`, the index at which the values should be filled and the index starting at which values should be copied. For instance, if you want to copy the values from the first two elements in the array into the last two items in the array, you can do so as follows:

```js
let numbers = [1, 2, 3, 4];

// paste values into array starting at index 2
// copy values from array starting at index 0
numbers.copyWithin(2, 0);

console.log(numbers.toString());    // 1,2,1,2
```

This code copies the values in `numbers` beginning from index 2, so both indices 2 and 3 will be overwritten. The second argument to `copyWithin()` is `0`, which indicates to start copying values from index 0 and continue until there are no more elements to copy into.

By default, `copyWithin()` always copies values up to the end of the array, but you can provide an optional third argument to limit how many elements will be overwritten. That third argument is an exclusive end index at which copying of values stops. Here's an example:

```js
let numbers = [1, 2, 3, 4];

// paste values into array starting at index 2
// copy values from array starting at index 0
// stop copying values when you hit index 1
numbers.copyWithin(2, 0, 1);

console.log(numbers.toString());    // 1,2,1,4
```

In this example, the optional end index is set to `1` so that only the value in index 0 is copied. The last element in the array remains unchanged.

I> As with `fill()`, if you pass a negative number for any argument to `copyWithin()`, the array's length is automatically added to that value to determine the index to use.

The use cases for `fill()` and `copyWithin()` may not be obvious to you at this point. That's because these methods originated on typed arrays and were then added to regular arrays for consistency. However, if you end up using typed arrays for manipulating the bits of a number, these methods become a lot more useful.

## Typed Arrays

Typed arrays are special-purpose arrays designed to work with numeric types (not all types, as it may seem from the name). The origin of typed arrays can be traced back to WebGL, a port of Open GL ES 2.0 designed for use in web pages with the `<canvas>` element. Typed arrays were created as part of this port to provide fast bitwise arithmetic in JavaScript. The native JavaScript numbers were too slow to due to being stored in a 64-bit floating-point format and converted into 32-bit integers as needed, so typed arrays were introduced to circumvent this limitation and provide better performance for these operations. The concept is that any single number can be treated like arrays of bits, and in doing so, can make use of the familiar methods available on JavaScript arrays.

ECMAScript 6 adopted typed arrays as a formal part of the language to ensure better compatibility across JavaScript engines and interoperability with JavaScript arrays. While the ECMAScript 6 version is of typed arrays is not exactly the same as the WebGL version, there are enough similarities to make the ECMAScript 6 version an evolution of the WebGL version rather than a different approach.

### Numeric Data Types

JavaScript numbers are stored in IEEE 754 format, which uses 64 bits to store a floating-point representation of the number. This format represents both integers and floats in JavaScript, with conversion between the two formats happening frequently as numbers are changed. Typed arrays allow the storage and manipulation of eight different numeric types:

1. Signed 8-bit integer (int8)
1. Unsigned 8-bit integer (uint8)
1. Signed 16-bit integer (int16)
1. Unsigned 16-bit integer (uint16)
1. Signed 32-bit integer (int32)
1. Unsigned 32-bit integer (uint32)
1. 32-bit float (float32)
1. 64-bit float (float64)

If you want to represent an int8 today in a JavaScript number, you would be wasting 56 bits. Those bits might better be used to store additional int8's (or any other number that requires less than 56 bits). This is one of the use cases typed arrays address.

All of the operations and objects related to typed arrays are centered around these eight data types. In order to use them, though, you'll need to create an array buffer to store the data.

### Array Buffers

The foundational piece underlying all typed arrays is an array buffer. An array buffer is a memory location for any number of bytes. Creating an array buffer is akin to calling something like `malloc()` in C to allocate memory without specifying what is contained within. You can create an array buffer by using the `ArrayBuffer` constructor and passing in the number of bytes it should contain:

```js
let buffer = new ArrayBuffer(10);   // allocate 10 bytes
```

Once created, you can retrieve the number of bytes in the array buffer by using the `byteLength` property:

```js
let buffer = new ArrayBuffer(10);   // allocate 10 bytes
console.log(buffer.byteLength);     // 10
```

The only other thing you can do is create a new array buffer that contains part of an existing array buffer using the `slice()` method. The `slice()` method works in a similar manner to the array `slice()` method in that you pass in the start index and end index as arguments and then return a new `ArrayBuffer` instance that is comprised of those elements from the original. For example:

```js
let buffer = new ArrayBuffer(10);   // allocate 10 bytes


let buffer2 = buffer.slice(4, 6);
console.log(buffer2.byteLength);    // 2
```

In this code, `buffer2` is created by extract the bytes at index 4 and 5 (the second argument to `slice()`, just like with arrays, is exclusive).

Of course, creating a storage location isn't very helpful without being able to write data into that location. To do so, you'll need to create a view.

I> Keep in mind, you cannot change the number of bytes that an array buffer represents. It always represents the exact number specified. You can change the data contained within an array buffer but never the size of the array buffer itself.

### Views

While array buffers represent a memory location, views are the interface through which you manipulate that memory. A view operates on an array buffer, or a subset of an array buffer's bytes, reading and writing data in a particular format. The `DataView` type is a generic view on an array buffer that allows you to operate on all eight numeric data types. To do so, first create an `ArrayBuffer` and then use it to create a new `DataView`. Here's an example:

```js
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer);
```

The `view` object in this example has access to the entire 10 bytes of `buffer`. You can alternately create a view over just a portion of a buffer by providing a byte offset and, optionally, the number of bytes to include from that offset (defaults to the end of the buffer when not present). For example:

```js
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer, 5, 2);      // cover bytes 5 and 6
```

Here, `view` operates only on the bytes at indices 5 and 6. This approach allows you to create several views over the same array buffer, which can be useful if you want to have a single memory location for an entire application rather than dynamically allocating space as needed.

#### Retrieving View Information

You can retrieve information about the view by using the following read-only properties:

* `buffer` - the array buffer that the view is tied to
* `byteOffset` - the second argument to the `DataView` constructor if provided (0 by default)
* `byteLength` - the third argument to the `DataView` constructor if provided (the buffer's `byteLength` by default)

Using these properties, you can inspect exactly where a view is operating, such as:

```js
let buffer = new ArrayBuffer(10),
    view1 = new DataView(buffer),           // cover all bytes
    view2 = new DataView(buffer, 5, 2);     // cover bytes 5 and 6

console.log(view1.buffer === buffer);       // true
console.log(view2.buffer === buffer);       // true
console.log(view1.byteOffset);              // 0
console.log(view2.byteOffset);              // 5
console.log(view1.byteLength);              // 10
console.log(view2.byteLength);              // 2
```

This code creates two views, `view1` that is a view over the entire array buffer and `view2` that operates on a small section of the array buffer. The `buffer` property for each view is the same because they both work on the same array buffer. The `byteOffset` and `byteLength` are different for each view, reflecting where in the array buffer the view operates.

Of course, reading information about memory isn't very useful on its own. You need to write data into and read data out of that memory to get any benefit.

#### Reading and Writing Data

There are two methods for each of the eight numeric data types, one to write data and one to read data. The methods have a name beginning with either "set" or "get" and followed by the data type abbreviation. The "set" methods accept three arguments, the byte offset at which to write, the value to write, and an optional boolean value indicating the value should be stored in little-endian format (with the least significant byte at byte 0 instead of in the last byte). The "get" methods accept two arguments, the byte offset to read from and an optional boolean value indicating the value should be read as little-endian. Here's an example:

```js
let buffer = new ArrayBuffer(2),
    view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt8(0));       // 5
console.log(view.getInt8(1));       // -1
```

This example uses a two-byte array buffer to store two int8 values. The first value is set at offset 0 and the second at offset 1 reflecting that each is taking up a full byte (8 bits). Those values are later retrieved from their positions. While this example uses int8 values, you can use any of the eight numeric types with the corresponding methods. The complete list of methods is:

* `getInt8(byteOffset, littleEndian)` - read an int8 starting at `byteOffset`
* `setInt8(byteOffset, value, littleEndian)` - write an int8 starting at `byteOffset`
* `getUint8(byteOffset, littleEndian)` - read an uint8 starting at `byteOffset`
* `setUint8(byteOffset, value, littleEndian)` - write an uint8 starting at `byteOffset`
* `getInt16(byteOffset, littleEndian)` - read an int16 starting at `byteOffset`
* `setInt16(byteOffset, value, littleEndian)` - write an int16 starting at `byteOffset`
* `getUint16(byteOffset, littleEndian)` - read an uint16 starting at `byteOffset`
* `setUint16(byteOffset, value, littleEndian)` - write an uint16 starting at `byteOffset`
* `getInt32(byteOffset, littleEndian)` - read an int32 starting at `byteOffset`
* `setInt32(byteOffset, value, littleEndian)` - write an int32 starting at `byteOffset`
* `getUint32(byteOffset, littleEndian)` - read an uint32 starting at `byteOffset`
* `setUint32(byteOffset, value, littleEndian)` - write an uint32 starting at `byteOffset`
* `getFloat32(byteOffset, littleEndian)` - read a float32 starting at `byteOffset`
* `setFloat32(byteOffset, value, littleEndian)` - write a float32 starting at `byteOffset`
* `getFloat64(byteOffset, littleEndian)` - read a float64 starting at `byteOffset`
* `setFloat64(byteOffset, value, littleEndian)` - write a float64 starting at `byteOffset`

The interesting aspect of views is that you can read and write in any format at any point in time, regardless of how data was previously stored. For instance, what happens if you write two int8 values and read the buffer as int16? It works just fine, as in this example:

```js
let buffer = new ArrayBuffer(2),
    view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt16(0));      // 1535
console.log(view.getInt8(0));       // 5
console.log(view.getInt8(1));       // -1
```

The call to `view.getInt16(0)` reads all of the bytes in the view and interprets those bytes as the number 1535. To understand why this happens, take a look at what each line does to the array buffer.

<!-- TODO: Maybe this is better as a graphic? - NZ -->

```
new ArrayBuffer(2)      0000000000000000
view.setInt8(0, 5);     0000010100000000
view.setInt8(1, -1);    0000010111111111
```

The array buffer starts out with 16 bits that are all zero. Adding the int8 value of 5 at the start of the array buffer introduces a couple of ones by writing the 8-bit representation (`00000101`). When -1 is written to the second byte, it sets all bits to one (the two's complement representation). At the end, the array buffer contains 16 bits that are then read out as a 16-bit integer using `getInt16()`. The interpretation of those 16 bits as a single number is 1535.

The `DataView` object is perfect for use cases that mix different data types in this way. However, if you're only using one specific data type, then the type-specific views are a better choice.

#### Type-Specific Views

ECMAScript 6 typed arrays are actually type-specific views for array buffers. Instead of using a generic `DataView` object to operate on an array buffer, you can use objects that enforce specific data types. There are nine type-specific views, corresponding to the eight numeric data types plus one additional optional for uint8 values. The following table is an abbreviated version of the one found in the specification (section 22.2) and lists out the various types:

|Constructor Name|Element Size|Description                        |Equivalent C Type|
|----------------|------------|-----------------------------------|-----------------|
|`Int8Array`     |1           |8-bit 2's complement signed integer|signed char|
|`Uint8Array`    |1           |8-bit unsigned integer             |unsigned char|
|`Uint8ClampedArray`|1        |8-bit unsigned integer (clamped conversion)|unsigned char|
|`Int16Array`    |2           |16-bit 2's complement signed integer|short|
|`Uint16Array`   |2           |16-bit unsigned integer             |unsigned short|
|`Int32Array`    |4           |32-bit 2's complement signed integer|int|
|`Uint32Array`   |4           |32-bit unsigned integer unsigned    |int|
|`Float32Array`  |4           |32-bit IEEE floating point          |float|
|`Float64Array`  |8           |64-bit IEEE floating point          |double|

The `Uint8ClampedArray` is the same as `Uint8Array` except when values are less than 0 or greater than 255. In that case, a `Uint8ClampedArray` will convert values lower than 0 to be 0 (-1 becomes 0, for example) and values higher than 255 to be 255 (300 becomes 255, for example).

Each of the typed arrays limits operations to working on a particular type of data, so all operations on `Int8Array` use int8 values. That means each typed array also has a difference byte size per element. Whereas `Int8Array` has a single byte element, `Float64Array` uses eight bytes per element. The elements are accessed using numeric indices just like regular arrays, allowing you to avoid the somewhat awkward calls to the "set" and "get" methods of `DataView`.

W> While typed arrays looks and behave similar to JavaScript arrays, they do not inherit from `Array`.

##### Creating Type-Specific Views

<!-- I'm a bit unhappy that this is a heading level 5, not sure what to do about it, though. - NZ -->

Typed array constructors accept multiple different types of arguments. First, you can create a new typed array by passing the same arguments as you would to `DataView`, meaning an array buffer, an optional byte offset, and an optional byte length. For example:

```js
let buffer = new ArrayBuffer(10),
    view1 = new Int8Array(buffer),
    view2 = new Int8Array(buffer, 5, 2);

console.log(view1.buffer === buffer);       // true
console.log(view2.buffer === buffer);       // true
console.log(view1.byteOffset);              // 0
console.log(view2.byteOffset);              // 5
console.log(view1.byteLength);              // 10
console.log(view2.byteLength);              // 2
```

In this code, the two views are both `UInt8Array` instances that use `buffer`. Both `view1` and `view2` have the same `buffer`, `byteOffset`, and `byteLength` properties that exist on `DataView` instances. It's easy to swap in using a typed array whenever you use a `DataView` so long as you only work with one numeric type.

The second way to create a typed array is to pass a single number to the constructor. That number represents the number of elements (not bytes) to allocate to the array. In doing so, the constructor creates a new buffer that has the correct number of bytes to represent the number of array elements. You can then access the number of elements in the array by using the `length` property. For example:

```js
let ints = new Int16Array(2),
    floats = new Float32Array(5);

console.log(ints.byteLength);       // 4
console.log(ints.length);           // 2

console.log(floats.byteLength);     // 20
console.log(floats.length);         // 5
```

The `ints` array is created to have two elements. Each 16-bit integer requires two bytes per value, so the array is allocated four bytes. The `floats` array is created to have five elements, so the number of bytes required is 20 (four bytes per element). In both cases, a new buffer is created and can be accessed using the `buffer` property if necessary.

W> If no argument is passed to a typed array constructor, the constructor acts as if `0` was passed. This effectively creates a typed array that cannot hold any data because zero bytes are allocated to the buffer.

A> ### Element Size
A>
A> Each typed array is made up of a number of elements, and the element size is the number of bytes each element represents. This value is stored on a `BYTES_PER_ELEMENT` property on each constructor and each instance, so you can easily query the element size:
A>
A> ```js
A> console.log(UInt8Array.BYTES_PER_ELEMENT);      // 1
A> console.log(UInt16Array.BYTES_PER_ELEMENT);     // 2
A>
A> let ints = new Int8Array(5);
A> console.log(ints.BYTES_PER_ELEMENT);            // 1
A> ```

The third way to create a typed array is to pass an object as the only argument to the constructor. The object can be any of the following:

* **Typed Array** - each element is copied into a new element on the new typed array (for example, an int8 is copied into an int16). The new typed array has a different array buffer than the one that was passed in.
* **Iterable** - the iterator is called to retrieve the items to insert into the typed array. The constructor will throw an error if any of the elements are invalid for the type.
* **Array** - the elements of the array are copied into a new typed array. The constructor will throw an error if any of the elements are invalid for the type.
* **Array-Like Object** - behaves the same as an array.

In each of these cases, a new typed array is created with the data from the source object. This can be especially useful when you want to initialize a typed array with some values, such as:

```js
let ints1 = new Int16Array([25, 50]),
    ints2 = new Int32Array(ints1);

console.log(ints1.buffer === ints2.buffer);     // false

console.log(ints1.byteLength);      // 4
console.log(ints1.length);          // 2
console.log(ints1[0]);              // 25
console.log(ints1[1]);              // 50

console.log(ints2.byteLength);      // 8
console.log(ints2.length);          // 2
console.log(ints2[0]);              // 25
console.log(ints2[1]);              // 50
```

This example creates an `Int16Array` and initializes it with an array of two values. Then, an `Int32Array` is created and passed the `Int16Array`. The values 25 and 50 are copied from `ints1` into `ints2` as the two typed arrays have completely separate buffers. The same numbers are represented in both typed arrays but `ints2` has eight bytes to represent the data while `ints1` has only four.

##### Similarities with Arrays

As you've already seen, typed arrays can be used like regular arrays in many situations. You can see how many elements are in the array using the `length` property and you can access the elements directly using numeric indices, such as:

```js
let ints = new Int16Array([25, 50]);

console.log(ints.length);          // 2
console.log(ints[0]);              // 25
console.log(ints[1]);              // 50

ints[0] = 1;
ints[1] = 2;

console.log(ints[0]);              // 1
console.log(ints[1]);              // 2
```

In this example, a new `Int16Array` with two items is created. The items are read from and written to using their numeric indices, and those values are automatically stored and converted into int16 values as part of the operation.

Typed arrays are also similar to regular arrays due to the availability of a large number of array methods. Here are the array methods you can use on typed arrays:

* `copyWithin()`
* `entries()`
* `fill()`
* `filter()`
* `find()`
* `findIndex()`
* `forEach()`
* `indexOf()`
* `join()`
* `keys()`
* `lastIndexOf()`
* `map()`
* `reduce()`
* `reduceRight()`
* `reverse()`
* `slice()`
* `some()`
* `sort()`
* `values()`

Keep in mind that while all of these methods act the same as those on `Array.prototype`, they are not the same methods. The typed array methods have additional checks for numeric type safety and, when an array is returned, will return a typed array instead of a regular array. Here's a simple example:

```js
let ints = new Int16Array([25, 50]),
    mapped = ints.map(v => v * 2);

console.log(mapped.length);        // 2
console.log(mapped[0]);            // 50
console.log(mapped[1]);            // 100

console.log(mapped instanceof Int16Array);  // true
```

This example uses the `map()` method to create a new array based on the values in `ints`. The mapping function doubles each value in the array and returns a new `Int16Array`.

Also note that typed arrays have the same three iterators as regular arrays: `entries()`, `keys()`, and `values()`. These allow you to use the spread operator and `for-of` loops in the same way as you would regular arrays. For example:

```js
let ints = new Int16Array([25, 50]),
    intsArray = [...ints];

console.log(intsArray instanceof Array);    // true
console.log(intsArray[0]);                  // 25
console.log(intsArray[1]);                  // 50
```

This code creates a new array `intsArray` containing the same data as the typed array `ints`. As with other iterables, the spread operator is an easy way to convert typed arrays into regular arrays.

Lastly, all typed arrays have static `of()` and `from()` methods that work the same way as `Array.of()` and `Array.from()`. The only difference is that the result is a typed array instead of a regular array. Otherwise, you can use these methods in the same way to create various typed arrays, such as:

```js
let ints = Int16Array.of(25, 50),
    floats = Float32Array.from([1.5, 2.5]);

console.log(ints instanceof Int16Array);        // true
console.log(floats instanceof Float32Array);    // true

console.log(ints.length);       // 2
console.log(ints[0]);           // 25
console.log(ints[1]);           // 50

console.log(floats.length);     // 2
console.log(floats[0]);         // 1.5
console.log(floats[1]);         // 2.5
```

The `of()` and `from()` methods in this example are used to create an `Int16Array` and `Float32Array`, respectively. These methods ensure that typed arrays can be created just as easily as regular arrays.

#### Differences from Arrays

The most importance difference between typed arrays and regular arrays is that typed arrays are not regular arrays. That means they do not inherit from `Array` and `Array.isArray()` returns `false` when passed a typed array. For example:

```js
let ints = new Int16Array([25, 50]);

console.log(ints instanceof Array);     // false
console.log(Array.isArray(ints));       // false
```

The `ints` variable is a typed array, so it's not an instance of `Array` and cannot otherwise be identifier as an array. This distinction is important because there are many ways in which typed arrays do not act like regular arrays.

Whereas regular arrays can grow and shrink as you interact with them, typed arrays always remain the same size. You cannot assign a value to a nonexistent numeric index like you can with regular arrays, as typed arrays will ignore the operation. Here's an example:

```js
let ints = new Int16Array([25, 50]);

console.log(ints.length);          // 2
console.log(ints[0]);              // 25
console.log(ints[1]);              // 50

ints[2] = 5;

console.log(ints.length);          // 2
console.log(ints[2]);              // undefined
```

Despite assigning to the numeric index `2` in this example, the `ints` array does not grow at all. The `length` remains the same and the value is thrown away.

Typed arrays also have checks to ensure that only valid data types are used. Zero is used in place of any invalid values. For example:

```js
let ints = new Int16Array(["hi"]);

console.log(ints.length);       // 1
console.log(ints[0]);           // 0
```

This code attempts to use the string value `"hi"` in an `Int16Array`. Of course, strings are invalid data types in typed arrays, so the value is inserted as zero instead. The `length` of the array is still one, and the `ints[0]` slot exists, it's just filled with zero instead of the string. The same restriction applies to all methods that modify values in a typed array.For example, if the function passed to `map()` returns an invalid value for the type array, then zero is used instead:

```js
let ints = new Int16Array([25, 50]),
    mapped = ints.map(v => "hi");

console.log(mapped.length);        // 2
console.log(mapped[0]);            // 0
console.log(mapped[1]);            // 0

console.log(mapped instanceof Int16Array);  // true
console.log(mapped instanceof Array);       // false
```

Since the string value `"hi"` isn't a 16-bit integer, it's replaced with `0` in the resulting array. All of the array methods have similar error correction behavior to avoid throwing errors when invalid data is present.

The last difference between typed arrays and regular arrays is that typed arrays are missing several array methods. The following methods are not available on typed arrays:

* `concat()`
* `pop()`
* `push()`
* `shift()`
* `splice()`
* `unshift()`

With the exception of `concat()`, the other methods can change the size of an array and so are not available for typed arrays (since they cannot change size). The `concat()` method isn't available because it is unclear what concatenating two typed arrays, especially if they dealt with different data types, would mean for the result.

##### Additional Methods

There are a couple of typed arrays methods that are not present on regular arrays: `set()` and `subarray()`. These two methods are opposites in that `set()` allows you to copy another array into an existing typed array whereas `subarray()` let's you extract part of an existing typed array into a new typed array.

The `set()` method accepts an array (either typed or regular) and an optional offset at which to insert the data (default to zero). The data from the array argument is copied into the destination typed array while ensuring only valid data types are used. Here's an example:

```js
let ints = new Int16Array(4);

ints.set([25, 50]);
ints.set([75, 100], 2);

console.log(ints.toString());   // 25,50,75,100
```

This code creates an `Int16Array` with four elements. The first call to `set()` copies two values to the first and second elements in the array. The second call to `set()` uses an offset of `2` to indicate that the values should be placed in the array starting at the third element.

Whereas `set()` inserts new values into a typed array, `subarray()` extracts values into a new typed array. The `subarray()` method accepts an optional start and end index (the end index is exclusive, as in methods like `slice()`) and returns a new typed array. You can also omit both arguments to create a clone of the typed array. For example:

```js
let ints = new Int16Array([25, 50, 75, 100]),
    subints1 = ints.subarray(),
    subints2 = ints.subarray(2),
    subints3 = ints.subarray(1, 3);

console.log(subints1.toString());   // 25,50,75,100
console.log(subints2.toString());   // 75,100
console.log(subints3.toString());   // 50,75
```

There are three typed arrays created from the original `ints` array in this example. The `subints1` array is a clone of `ints`, containing all the same information. The `subints2` array starts copying data from index 2, and so contains only the last two elements of the array (75 and 100). The `subints3` array contains only the middle two elements of the `ints` array, as both arguments were used for `subarray()`.

## Summary

ECMAScript 6 continues the work of ECMAScript 5 by continuing to update and change arrays to be more useful. There are now two new ways to create arrays, `Array.of()` and `Array.from()`. These methods can each be used to create arrays and, in the case of `Array.from()`, convert iterables and arraylike objects into arrays. Both methods are inherited by derived array classes and use the `@@species` property to determine what type of value should be returned.

There are also several new methods on arrays. The `fill()` and `copyWithin()` methods allow you to alter array elements in-place. The `find()` and `findIndex()` methods are useful for finding the first element in an array that matches some criteria. The former returns the first element that fits the criteria and the latter returns the index at which the element is found.

Typed arrays are not actually arrays, as they do not inherit from `Array`, but do look and behave a lot like arrays. Typed arrays contain one of eight different numeric data types and are built upon `ArrayBuffer` objects that represent the underlying bits of a number or series of numbers. Typed arrays are a more efficient way of doing bitwise arithmetic because the values are not converted back and forth between formats, as is the case with the JavaScript number type.
