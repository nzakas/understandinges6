# Appendix A: Other Changes

Along with the changes already mentioned in the book, ECMAScript 6 has made some very small changes and improvements. This appendix lists out those changes.

## Working with Integers

A lot of confusion has been related to JavaScript's single number encoding, IEEE 754, that is used to represent both integers and floats. The language goes through great pains to ensure that developers don't need to worry about the details, but problems still leak through from time to time. ECMAScript 6 seeks to address this by making it easier to identify and work with integers.

### Identifying Integers

The first addition is `Number.isInteger()`, which allows you to determine if a value represents an integer in JavaScript. Since integers and floats are stored differently, the JavaScript engine looks at the underlying representation of the value to make this determination. That means numbers that look like floats might actually be stored as integers and therefore return `true` from `Number.isInteger()`. For example:

```js
console.log(Number.isInteger(25));      // true
console.log(Number.isInteger(25.0));    // true
console.log(Number.isInteger(25.1));    // false
```

In this code, `Number.isInteger()` returns `true` for both `25` and `25.0` even though the latter looks like a float. Simply adding a decimal point to a number doesn't automatically make it a float in JavaScript. Since `25.0` is really just `25`, it is stored as an integer. The number `25.1`, however, is stored as a float because there is a fraction value.

### Safe Integers

IEEE 754 can only accurately represent integers between -2^53^ and 2^53^, and outside of this "safe" range, binary representations end up reused for multiple numeric values. That means JavaScript can only safely represent integers within the IEEE 754 range before problems become apparent. For example:

```js
console.log(Math.pow(2, 53));      // 9007199254740992
console.log(Math.pow(2, 53) + 1);  // 9007199254740992
```

This example doesn't contain a typo, two different numbers end up represented by the same JavaScript integer. The effect becomes more prevalent the further the value is outside of the safe range.

ECMAScript 6 introduces `Number.isSafeInteger()` to better identify integers that can accurately be represented in the language, and also adds `Number.MAX_SAFE_INTEGER` and `Number.MIN_SAFE_INTEGER` that represent the upper and lower bounds of the same range, respectively. The `Number.isSafeInteger()` method ensures that a value is an integer and falls within the safe range of integer values:

```js
var inside = Number.MAX_SAFE_INTEGER,
    outside = inside + 1;

console.log(Number.isInteger(inside));          // true
console.log(Number.isSafeInteger(inside));      // true

console.log(Number.isInteger(outside));         // true
console.log(Number.isSafeInteger(outside));     // false
```

The number `inside` is the largest safe integer, so it returns `true` for both `Number.isInteger()` and `Number.isSafeInteger()`. The number `outside` is the first questionable integer value, so it is no longer considered safe even though it's still an integer.

Most of the time, you only want to deal with safe integers when doing integer arithmetic or comparisons in JavaScript, so it's a good idea to use `Number.isSafeInteger()` as part of input validation.

## New Math Methods

The new emphasis on gaming and graphics that led to the inclusion of typed arrays in JavaScript also led to the realization that many mathematical calculations could be done more efficiently by a JavaScript engine. Optimization strategies like asm.js, which works on a subset of JavaScript to improve performance, need more information to perform calculations in the fastest way possible. It's important, for instance, to know whether the numbers should be treated as 32-bit integers or as 64-bit floats so hardware-based operations can be used (which are much faster than software-based operations).

As a result, ECMAScript 6 adds several new methods to the `Math` object. These new methods are important for improving the speed of common mathematical calculations, and therefore, improving the speed of applications that must perform many calculations (such as graphics programs). The new methods are listed below.

| Method | Description |
|--------|-------------|
|`Math.acosh(x)`| Returns the inverse hyperbolic cosine of `x`. |
|`Math.asinh(x)`| Returns the inverse hyperbolic sine of `x`. |
|`Math.atanh(x)`| Returns the inverse hyperbolic tangent of `x` |
|`Math.cbrt(x)`| Returns the cubed root of `x`. |
|`Math.clz32(x)`| Returns the number of leading zero bits in the 32-bit integer representation of `x`. |
|`Math.cosh(x)`| Returns the hyperbolic cosine of `x`. |
|`Math.expm1(x)`| Returns the result of subtracting 1 from the exponential function of `x`|
|`Math.fround(x)`| Returns the nearest single-precision float of `x`.|
|`Math.hypot(...values)`| Returns the square root of the sum of the squares of each argument. |
|`Math.imul(x, y)`| Returns the result of performing true 32-bit multiplication of the two arguments. |
|`Math.log1p(x)`| Returns the natural logarithm of `1 + x`. |
|`Math.log10(x)`| Returns the base 10 logarithm of `x`. |
|`Math.log2(x)`| Returns the base 2 logarithm of `x`. |
|`Math.sign(x)`| Returns -1 if the `x` is negative, 0 if `x` is +0 or -0, or 1 if `x` is positive.|
|`Math.sinh(x)`| Returns the hyperbolic sine of `x`. |
|`Math.tanh(x)`| Returns the hyperbolic tangent of `x`. |
|`Math.trunc(x)`| Removes fraction digits from a float and returns an integer.|

It's beyond the scope of this book to explain each new method and what it does in detail. However, if you are looking for a reasonably common calculation, be sure to check the new `Math` methods before implementing it yourself.

## Unicode Identifiers

Better Unicode support in ECMAScript 6 also means changes to what characters may be used for an identifier. In ECMAScript 5, it was already possible to use Unicode escape sequences for identifiers, such as:

```js
// Valid in ECMAScript 5 and 6
var \u0061 = "abc";

console.log(\u0061);        // "abc"

// equivalent to
// console.log(a);          // "abc"
```

In ECMAScript 6, you can also use Unicode code point escape sequences as identifiers:

```js
// Valid in ECMAScript 5 and 6
var \u{61} = "abc";

console.log(\u{61});        // "abc"

// equivalent to
// console.log(a);          // "abc"
```

Additionally, ECMAScript 6 formally specifies valid identifiers in terms of [Unicode Standard Annex #31: Unicode Identifier and Pattern Syntax](http://unicode.org/reports/tr31/):

1. The first character must be `$`, `_`, or any Unicode symbol with a derived core property of `ID_Start`.
1. Each subsequent character must be `$`, `_`, `\u200c` (zero-width non-joiner), `\u200d` (zero-width joiner), or any Unicode symbol with a derived core property of `ID_Continue`.

The `ID_Start` and `ID_Continue` derived core properties are defined in Unicode Identifier and Pattern Syntax as a way to identify symbols that are appropriate for use in identifiers such as variables and domain names (the specification is not specific to JavaScript).

## Formalizing the `__proto__` Property

Even before ECMAScript 5 was finished, several JavaScript engines already implemented a custom property called `__proto__` that could be used to both get and set `[[Prototype]]`. Effectively, `__proto__` was an early precursor to both the `Object.getPrototypeOf()` and `Object.setPrototypeOf()` methods. It was unrealistic to expect all JavaScript engines to remove this property, so ECMAScript 6 also formalized the `__proto__` behavior. However, the formalization is in Appendix B along with this warning:

> These features are not considered part of the core ECMAScript language. Programmers should not use or assume the existence of these features and behaviours when writing new ECMAScript code. ECMAScript implementations are discouraged from implementing these features unless the
implementation is part of a web browser or is required to run the same legacy ECMAScript code that web browsers encounter.

While it's best to avoid using `__proto__`, it's interesting to see how the specification defined it. In ECMAScript 6 engines, `Object.prototype.__proto__` is defined as an accessor property whose `get` method calls `Object.getPrototypeOf()` and whose `set` method calls the `Object.setPrototypeOf()` method. This leaves no real difference between using `__proto__` and the other methods, except that `__proto__` allows you to set the prototype of an object literal directly. Here's how that works:

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// prototype is person
let friend = {
    __proto__: person
};
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true
console.log(friend.__proto__ === person);               // true

// set prototype to dog
friend.__proto__ = dog;
console.log(friend.getGreeting());                      // "Woof"
console.log(friend.__proto__ === dog);                  // true
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

This example is functionally equivalent to the `getGreeting()` example. The call to `Object.create()` is replaced with an object literal that assigns a value to the `__proto__` property. The only real difference between creating an object with `Object.create()` or an object literal with `__proto__` is that the former requires you to specify full property descriptors for any additional object properties. The latter is just a standard object literal.

W> The `__proto__` property is special in a number of ways:
W>
W> 1. You can only specify it once in an object literal. If you specify two `__proto__` properties, then an error is thrown. This is the only object literal property with that restriction.
W> 1. The computed form `["__proto__"]` acts like a regular property and doesn't set or return the current object's prototype. All rules related to object literal properties apply in this form, as opposed to the non-computed form, which has exceptions.
W>
W> For these reasons, it's recommend to avoid using `__proto__` when you can use `Object.getPrototypeOf()` and `Object.setPrototypeOf()` instead.
