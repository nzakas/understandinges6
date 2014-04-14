# The Basics

**Note:** This chapter is a work-in-progress. All of the information it contains is accurate, however, there are pieces of content missing. Wherever possible, a "TODO" has been left indicating where more content will be added.

ECMAScript 6 makes a large number of changes on top of ECMAScript 5. Some of the changes are larger, such as adding new types or syntax, while others are quite small, providing incremental improvements on top of the language. This chapter covers those incremental improvements that likely won't gain a lot of attention but provide some important functionality that may make certain types of problems easier to solve.

## Better Unicode Support

Prior to ECMAScript 6, JavaScript strings were based solely on the idea of 16-bit character encodings. All string properties and methods, such as `length` and `charAt()`, were based around the idea that every 16-bit sequence represented a single character. ECMAScript 5 allowed JavaScript engines to decide which of two encodings to use, either UCS-2 or UTF-16 (both encoding using 16-bit *code units*, making all observable operations the same). While it's true that all of the world's characters used to fit into 16 bits at one point in time, that is no longer the case.

Keeping within 16 bits wasn't possible for Unicode's stated goal of providing a globally unique identifier to every character in the world. These globally unique identifiers, called *code points*, are simply numbers starting at 0 (you might think of these as character codes, but there is subtle difference). A character encoding is responsible for encoding a code point into code units that are internally consistent. While UCS-2 had a one-to-one mapping of code point to code unit, UTF-16 is more variable.

The first 2^16 code points are represented as single 16-bit code units in UTF-16. This is called the *Basic Multilingual Plane* (BMP). Everything beyond that range is considered to be in a *supplementary plane*, where the code points can no longer be represented in just 16-bits. UTF-16 solves this problem by introducing *surrogate pairs* in which a single code point is represented by two 16-bit code units. That means any single character in a string can be either one code unit (for BMP, total of 16 bits) or two (for supplementary plane characters, total of 32 bits).

ECMAScript 5 kept all operations as working on 16-bit code units, meaning that you could get unexpected results from strings containing surrogate pairs. For example:

```js
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(text.charAt(0));        // ""
console.log(text.charAt(1));        // ""
console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
```

In this example, a single Unicode character is represented using surrogate pairs, and as such, the JavaScript string operations treat the string as having two 16-bit characters. That means `length` is 2, a regular expression trying to match a single character fails, and `charAt()` is unable to return a valid character string. The `charCodeAt()` method returns the appropriate 16-bit number for each code unit, but that is the closest you could get to the real value in ECMAScript 5.

ECMAScript 6 enforces encoding of strings in UTF-16. Standardizing on this character encoding means that the language can now support functionality designed to work specifically with surrogate pairs.

### The codePointAt() Method

The first example of fully supporting UTF-16 is the `codePointAt()` method, which can be used to retrieve the Unicode code point that maps to a given character. This method accepts the character position (not the code unit position) and returns an integer value:

```js
var text = "𠮷a";

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 97
```

The value returned is the Unicode code point value. For BMP characters, this will be the same result as using `charCodeAt()`, so the `"a"` returns 97. This method is the easiest way to determine if a given character is represented by one or two code points:

```js
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false
```

The upper bound of 16-bit characters is represented in hexadecimal as `FFFF`, so any code point above that number must be represented by two code units.

### String.fromCodePoint()

When ECMAScript provides a way to do something, it also tends to provide a way to do the reverse. You can use `codePointAt()` to retrieve the code point for a character in a string while `String.fromCodePoint()` produces a single-character string for the given code point. For example:

```js
console.log(String.fromCodePoint(134071));  // "𠮷"
```
You can think of `String.fromCodePoint()` as a more complete version of `String.fromCharCode()`. Each method has the same result for all characters in the BMP; the only difference is with characters outside of that range.

### Escaping Non-BMP Characters

ECMAScript 5 allows strings to contain 16-bit Unicode characters represented by an *escape sequence*. The escape sequence is the `\u` followed by four hexadecimal values. For example, the escape sequence `\u0061` represents the letter `"a"`:

```js
console.log("\u0061");      // "a"
```

If you try to use an escape sequence with a number past `FFFF`, the upper bound of the BMP, then you can get some surprising results:

```js
console.log("\u20BB7");     // "₻7"
```

Since Unicode escape sequences were defined as always having exactly four hexadecimal characters, ECMAScript evaluates `\u20BB7` as two characters: `\u20BB` and `"7"`. The first character is unprintable and the second is the number 7.

ECMAScript 6 solves this problem by introducing an extended Unicode escape sequence where the hexadecimal numbers are contained within curly braces. This allows up to 8 hexadecimal characters to specify a single character:

```js
console.log("\u{20BB7}");     // "𠮷"
```

Using the extended escape sequence, the correct character is contained in the string.

W> Make sure that you use this new escape sequence only in an ECMAScript 6 environment. In all other environments, doing so causes a syntax error. You may want to check and see if the environment supports the extended escape sequence using a function such as:
W>
W> {lang=js}
W> ~~~~~~~~
W> function supportsExtendedEscape() {
W>     try {
W>         "\u{00FF1}";
W>         return true;
W>     } catch (ex) {
W>         return false;
W>     }
W> }
W> ~~~~~~~~

### The normalize() Method

Another interesting aspect of Unicode is that different characters may be considered equivalent for the purposes of sorting or other comparison-based operations. There are two ways to define these relationships. First, *canonical equivalence* means that two sequences of code points are considered interchangeable in all respects. That even means that a combination of two characters can be canonically equivalent to one character. The second relationship is *compatibility*, meaning that two sequences of code points having different appearances but can be used interchangeably in certain situations.

The important thing to understand is that due to these relationships, it's possible to have two strings that represent fundamentally the same text and yet have them contain different code point sequences. For example,
the character "æ" and the string "ae" may be used interchangeably even though they are different code points. These two strings would therefore be unequal in JavaScript unless they are normalized in some way.

ECMAScript 6 supports the four Unicode normalization forms through a new `normalize()` method on strings. This method optionally accepts a single parameter, one of `"NFC"` (default), `"NFD"`, `"NFKC"`, or `"NFKD"`. It's beyond the scope of this book to explain the differences between these four forms. Just keep in mind that, in order to be used, you must normalize both strings that are being compared to the same form. For example:

```js
var normalized = values.map(function(text) {
    return text.normalize();
});
normalized.sort(function(first, second) {
    if (first < second) {
        return -1;
    } else if (first === second) {
        return 0;
    } else {
        return 1;
    }
});
```

In this code, the strings in a `values` array are converted into a normalized form so that the array can be sorted appropriately. You can accomplish the sort on the original array by calling `normalize()` as part of the comparator:

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize(),
        secondNormalized = second.normalize();

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

Once again, the most important thing to remember is that both values must be normalized in the same way. These examples have used the default, NFC, but you can just as easily specify one of the others:

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize("NFD"),
        secondNormalized = second.normalize("NFD");

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

If you've never worried about Unicode normalization before, then you probably won't have much use for this method. However, knowing that it is available will help should you ever end up working on an internationalized application.

### The Regular Expression u Flag

Many common string operations are accomplished by using regular expressions. However, as noted earlier, regular expressions also work on the basis of 16-bit code units each representing a single character. That's why the single character match in the earlier example didn't work. To address this problem, ECMAScript 6 defines a new flag for regular expressions: `u` for "Unicode".

When a regular expression has the `u` flag set, it switches modes to work on characters and not code units. That means the regular expression will no longer get confused about surrogate pairs in strings and can behave as expected. For example:

```js
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```

Adding the `u` flag allows the regular expression to correctly match the string by characters. Unfortunately, ECMAScript 6 does not have a way of determining how many code points are present in a string; fortunately, regular expressions can be used to figure it out:

```js
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("𠮷bc"));   // 3
```

The regular expression in this example matches a both whitespace and non-whitespace characters, and is applied globally with Unicode enabled. The `result` contains an array of matches when there's at least one match, so the array length ends up being the number of code points in the string.

W> Although this approach works, it's not very fast, especially when applied to long strings. Try to minimize counting code points whenever possible. Hopefully ECMAScript 7 will bring a more performant means by which to count code points.

## Additional String-Related Changes

JavaScript strings have always lagged behind similar features of other languages. It was only in ECMAScript 5 that strings finally gained a `trim()` method, and ECMAScript 6 continues extending strings with new functionality.

### contains(), startsWith(), endsWith()

Developers have used `indexOf()` as a way to identify strings inside of other strings since JavaScript was first introduced. ECMAScript 6 adds three new methods whose purpose is to identify strings inside of other strings:

* `contains()` - returns true if the given text is found anywhere within the string or false if not.
* `startsWith()` - returns true if the given text is found at the beginning of the string or false if not.
* `endsWith()` - returns true if the given text is found at the end of the string or false if not.

Each of these methods accepts two arguments: the text to search for and an optional location from which to start the search. When the second argument is omitted, `contains()` and `startsWith()` start search from the beginning of the string while `endsWith()` starts from the end. In effect, the second argument results in less of the string being searched. Here are some examples:

```js
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.contains("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.contains("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.contains("o", 8));          // false
```

These three methods make it much easier to identify substrings without needing to worry about identifying their exact position.

I> All of these methods return a boolean value. If you need to find the position of a string within another, use `indexOf()` or `lastIndexOf()`.

W> The `startsWith()`, `endsWith()`, and `contains()` methods will throw an error if you pass a regular expression instead of a string. This stands in contrast to `indexOf()` and `lastIndexOf()`, which both convert a regular expression argument into a string and then search for that string.

### repeat()

ECMAScript 6 also adds a `repeat()` method to strings. This method accepts a single argument, which is the number of times to repeat the string, and returns a new string that has the original string repeated the specified number of times. For example:

```js
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
```

This method is really a convenience function above all else, which can be especially useful when dealing with text manipulation. One example where this functionality comes in useful is with code formatting utilities where you need to create indentation levels:

```js
// indent using a specified number of spaces
var indent = " ".repeat(size),
    indentLevel = 0;

// whenever you increase the indent
var newIndent = indent.repeat(++indentLevel);
```

### The Regular Expression y Flag

ECMAScript 6 standardized the `y` flag after it had been implemented in Firefox as a proprietary extension to regular expressions. The `y` (sticky) flag indicates that the next match should be made starting with the value of `lastIndex` on the regular expression.

The `lastIndex` property indicates the position at which to start the match of a string and is set to `0` by default, meaning matches always start at the beginning of a string. You can, however, overwrite `lastIndex` to have it start from somewhere else:

```js
var pattern = /hello\d\s?/g,
    text = "hello1 hello2 hello3",
    result = pattern.exec(text);

console.log(result[0]);     // "hello1 "

pattern.lastIndex = 7;
result = pattern.exec(text);

console.log(result[0]);     // "hello2 "
```

In this example, the regular expression matches the string `"hello"` followed by a number and optionally a whitespace character. The `g` flag is important as it allows the regular expression to use `lastIndex` when set (without it, matches always start at `0` regardless of the `lastIndex` value). The first call to `exec()` results in matching "hello1" first while the second call, with a `lastIndex` of 6, matches "hello2" first.

The sticky flag tells the regular expression to save the index of the next character after the last match in `lastIndex` whenever an operation is performed (in the previous example, 7 is the location of next character after "hello1 "). If an operation results in no match then `lastIndex` is set back to 0.

```js
var pattern = /hello\d\s?/y,
    text = "hello1 hello2 hello3",
    result = pattern.exec(text);

console.log(result[0]);             // "hello1 "
console.log(pattern.lastIndex);     // 7

result = pattern.exec(text);

console.log(result[0]);             // "hello2 "
console.log(pattern.lastIndex);     // 14
```

Here, the same pattern is used but with the sticky flag instead of the global flag. The value of `lastIndex` changed to 7 after the first call to `exec()` and to 14 after the second call. Since the sticky flag is updating `lastIndex` for you, there's no need to keep track and manually update it yourself.

Perhaps the most important thing to understand about the sticky flag is that sticky regular expressions have an implied `^` at the beginning, indicating that the pattern should match from the beginning of the input. For example, if the previous example is changed to not match the whitespace character, there are different results:

```js
var pattern = /hello\d/y,
    text = "hello1 hello2 hello3",
    result = pattern.exec(text);

console.log(result[0]);             // "hello1 "
console.log(pattern.lastIndex);     // 6

result = pattern.exec(text);

console.log(result);                // null
console.log(pattern.lastIndex);     // 0
```

Without matching the whitespace character, the `lastIndex` is set to 6 after the first call to `exec()`. That means the regular expression will be evaluating the string as if it were this:

```js
" hello2 hello3"
```

Since there is an implied `^` at the beginning of the regular expression pattern, the pattern starts by matching `"h"` against the space and sees that they are not equivalent. The matching stops there and `null` is returned. The `lastIndex` property is reset to 0.

As with other regular expression flags, you can detect the presence of `y` by using a property. The `sticky` property is set to true with the sticky flag is present and false if not:

```js
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```

The `sticky` property is read-only based on the presence of the flag and so cannot be changed in code.

I> The `lastIndex` property is only honored when calling methods on the regular expression object such as `exec()` and `test()`. Passing the regular expression to a string method, such as `match()`, will not result in the sticky behavior.


## Object.is()

When you want to compare two values, you're probably used to using either the equals operator (`==`) or the identically equals operator (`===`). Many prefer to use the latter to avoid type coercion during the comparison. However, even the identically equals operator isn't entirely accurate. For example, the values +0 and -0 are considered equal by `===` even though they are represented differently in the JavaScript engine. Also `NaN === NaN` returns `false`, which necessitates using `isNaN()` to detect `NaN` properly.

ECMAScript 6 introduces `Object.is()` to make up for the remaining quirks of the identically equals operator. This method accepts two arguments and returns `true` if the values are equivalent. Two values are considered equivalent when they are of the same type and have the same value. In many cases, `Object.is()` works the same as `===`. The only differences are that +0 and -0 are considered not equivalent and `NaN` is considered equivalent to `NaN`. Here are some examples:

```js
console.log(+0 == -0);              // true
console.log(+0 === -0);             // true
console.log(Object.is(+0, -0));     // false

console.log(NaN == NaN);            // false
console.log(NaN === NaN);           // false
console.log(Object.is(NaN, NaN));   // true

console.log(5 == 5);                // true
console.log(5 == "5");              // true
console.log(5 === 5);               // true
console.log(5 === "5");             // false
console.log(Object.is(5, 5));       // true
console.log(Object.is(5, "5"));     // false
```

In most cases you will probably still want to use `==` or `===` for comparing values, as special cases covered by `Object.is()` may not affect you directly.

## Block bindings

Traditionally, one of the tricky parts of JavaScript has been the way that `var` declarations work. In most C-based languages, variables are created at the spot where the declaration occurs. In JavaScript, however, this is not the case. Variables declared using `var` are *hoisted* to the top of the function (or global scope) regardless of where the actual declaration occurs. For example:

```js
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

If you are unfamiliar with JavaScript, you might expect that the variable `value` is only defined if `condition` evaluates to true. In fact, the variable `value` is declared regardless. The JavaScript engine changes the function to look like this:

```js
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

The declaration of `value` is moved to the top (hoisted) while the initialization remains in the same spot. That means the variable `value` is actually still accessible from within the `else` clause, it just has a value of `undefined` because it hasn't been initialized.

It often takes new JavaScript developers some time to get used to declaration hoisting and this unique behavior can end up causing bugs. For this reason, ECMAScript 6 introduces block level scoping options to make the control of variable lifecycle a little more powerful.

### Let declarations

The `let` declaration syntax is the same as for `var`. You can basically replace `var` with `let` to declare a variable but keep its scope to the current code block. For example:

```js
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

This function now behaves much closer to other C-based languages. The variable `value` is declared using `let` instead of `var`. That means the declaration is not hoisted to the top, and the variable `value` is destroyed once execution has flowed out of the `if` block. If `condition` evaluates to false, then `value` is never declared or initialized.

Perhaps one of the areas where developers most want block level scoping of variables is with `for` loops. It's not uncommon to see code such as this:

```js
for (var i=0; i < items.length; i++) {
    process(items[i]);
}

// i is still accessible here
```

In other languages, where block level scoping is the default, code like this works as intended. In JavaScript, the variable `i` is still accessible after the loop is completed because the `var` declaration was hoisted. Using `let` allows you to get the intended behavior:

```js
for (let i=0; i < items.length; i++) {
    process(items[i]);
}

// i is not accessible here
```

In this example, the variable `i` only exists within the `for` loop. Once the loop is complete, the variable is destroyed and is no longer accessible elsewhere.

## Using let in loops

The behavior of `let` inside of loops is slightly different than with other blocks. Instead of creating a variable that is used with each iteration of the loop, each iteration actually gets its own variable to use. This is to solve an old problem with JavaScript loops. Consider the following:

```js
var funcs = [];

for (var i=0; i < 10; i++) {}
  funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
  func();     // outputs 11 ten times
});
```

This code will output the number `11` ten times in a row. That's because the variable `i` is shared across each iteration of the loop, meaning the closures created inside the loop all hold a reference to the same variable. The variable `i` has a value of `11` once the loop completes, and so that's the value each function outputs.

To fix this problem, developers use immediately-invoked function expressions (IIFEs) inside of loops to force a new copy of the variable to be created:

```js
var funcs = [];

for (var i=0; i < 10; i++) {}
  funcs.push((function(value) {
    return function() {
      console.log(value);
    };
  }(i)));
}

funcs.forEach(function(func) {
  func();     // outputs 1, then 2, then 3, up to 10
})
```

This version of the example uses an IIFE inside of the loop. The `i` variable is passed to the IIFE, which creates it's own copy and stores it as `value`. This is the value used of the function for that iteration, so calling each function returns the expected value.

A `let` declaration does this for you without the IIFE. Each iteration through the loop results in a new variable being created and initialized to the value of the variable with the same name from the previous iteration. That means you can simplify the process by using this code:

```js
var funcs = [];

for (let i=0; i < 10; i++) {}
  funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
  func();     // outputs 1, then 2, then 3, up to 10
})
```

This code works exactly like the code that used `var` and an IIFE but is, arguably, cleaner.

Unlike `var`, `let` has no hoisting characteristics. A variable declared with `let` cannot be accessed until after the `let` statement. Attempting to do so results in a reference error:

```js
if (condition) {
    console.log(value);     // ReferenceError!
    let value = "blue";
}
```

In this code, the variable `value` is defined and initialized using `let`, but that statement is never executed because the previous line throws an error.

If an identifier has already been defined in the block, then using the identifier in a `let` declaration causes an error to be thrown. For example:

```js
var count = 30;

// Throws an error
let count = 40;
```

In this example, `count` is declared twice, once with `var` and once with `let`. Because `let` will not redefine an identifier that already exists in the same scope, the declaration throws an error. No error is thrown if a `let` declaration creates a new variable in a scope with the same name as a variable in the containing scope, such as:

```js
var count = 30;

// Does not throw an error
if (condition) {

    let count = 40;

    // more code
}
```

Here, the `let` declaration will not throw an error because it is creating a new variable called `count` within the `if` statement. This new variable shadows the global `count`, preventing access to it from within the `if` block.

The intent of `let` is to replace `var` long term, as the former behaves more like variable declarations in other languages. If you are writing JavaScript that will execute only in an ECMAScript 6 or higher environment, you may want to try using `let` exclusively and leaving `var` for other scripts that require backwards compatibility.

I> Since `let` declarations are *not* hoisted to the top of the enclosing block, you may want to always place `let` declarations first in the block so that they are available to the entire block.

### Constant declarations

Another new way to define variables is to use the `const` declaration syntax. Variables declared using `const` are considered to be *constants*, so the value cannot be changed once set. For this reason, every `const` variable must be initialized. For example:

```js
// Valid constant
const MAX_ITEMS = 30;

// Syntax error: missing initialization
const NAME;
```

Constants are also block-level declarations, similar to `let`. That means constants are destroyed once execution flows out of the block in which they were declared and declarations are hoisted to the top of the block. For example:

```js
if (condition) {
    const MAX_ITEMS = 5;

    // more code
}

// MAX_ITEMS isn't accessible here
```

In this code, the constant `MAX_ITEMS` is declared within and `if` statement. Once the statement finishes executing, `MAX_ITEMS` is destroyed and is not accessible outside of that block.

Also similar to `let`, an error is thrown whenever a `const` declaration is made with an identifier for an already-defined variable in the same scope. It doesn't matter if that variable was declared using `var` (for global or function scope) or `let` (for block scope). For example:

```js
var message = "Hello!";
let age = 25;

// Each of these would cause an error given the previous declarations
const message = "Goodbye!";
const age = 30;
```

W> Several browsers implement pre-ECMAScript 6 versions of `const`. Implementations range from being simply a synonym for `var` (allowing the value to be overwritten) to actually defining constants but only in the global or function scope. For this reason, be especially careful with using `const` in a production system. It may not be providing you with the functionality you expect.

## Numbers

JavaScript numbers can be particularly complex due to the dual usage of a single type for both integers and floats. Numbers are stored in the IEEE 754 double precision floating point format, and that same format is used to represent both types of numbers. As one of the foundational data types of JavaScript (along with strings and booleans), numbers are quite important to JavaScript developers. Given the new emphasis on gaming and graphics in JavaScript, ECMAScript 6 sought to make working with numbers easier and more powerful.

### Octal and Binary Literals

ECMAScript 5 sought to simplify some common numerical errors by removing the previously-included octal integer literal notation in two places: `parseInt()` and strict mode. In ECMAScript 3 and earlier, octal numbers were represented with a leading `0` followed by any number of digits. For example:

```js
// ECMAScript 3
var number = 071;       // 57 in decimal

var value1 = parseInt("71");    // 71
var value2 = parseInt("071");   // 57
```

Many developers were confused by this version of octal literal numbers, and many mistakes were made as a result of misunderstanding the effects of a leading zero in various places. The most egregious was in `parseInt()`, where a leading zero meant the value would be treated as an octal rather than a decimal. This led to one of Douglas Crockford's first JSLint rules: always use the second argument of `parseInt()` to specify how the string should be interpreted.

ECMAScript 5 cut down on the use of octal numbers. First, `parseInt()` was changed so that it ignores leading zeros in the first argument when there is no second argument. This means a number cannot accidentally be treated as octal anymore. The second change was to eliminate octal literal notation in strict mode. Attempting to use an octal literal in strict mode results in a syntax error.

```js
// ECMAScript 5
var number = 071;       // 57 in decimal

var value1 = parseInt("71");        // 71
var value2 = parseInt("071");       // 71
var value3 = parseInt("071", 8);    // 57

function getValue() {
    "use strict";
    return 071;     // syntax error
}
```

By making these two changes, ECMAScript 5 sought to eliminate a lot of the confusion and errors associated with octal literals.

ECMAScript 6 took things a step further by reintroducing an octal literal notation, along with a binary literal notation. Both of these notations take a hint for the hexadecimal literal notation of prepending `0x`or `0X` to a value. The new octal literal format begins with `0o` or 0O` while the new binary literal format begins with `0b` or `0B`. Each literal type must be followed by one or more digits, 0-7 for octal, 0-1 for binary. Here's an example:

```js
// ECMAScript 6
var value1 = 0o71;      // 57 in decimal
var value2 = 0b101;     // 5 in decimal
```

Adding these two literal types allows JavaScript developers to quickly and easily include numeric values in binary, octal, decimal, and hexadecimal formats, which is very important in certain types of mathematical operations.

The `parseInt()` method doesn't handle strings that look like octal or binary literals:

```js
console.log(parseInt("0o71"));      // 0
console.log(parseInt("0b101"));     // 0
```

However, the `Number()` function will convert a string containing octal or binary literals correctly:

```js
console.log(Number("0o71"));      // 57
console.log(Number("0b101"));     // 5
```

When using octal or binary literal in strings, be sure to understand your use case and use the most appropriate method for converting them into numeric values.

### New Math Methods

The aforementioned new emphasis on gaming and graphics in JavaScript led to the realization that many mathematical calculations could be done more efficiently by a JavaScript engine than with pure JavaScript code. Optimization strategies like asm.js, which works on a subset of JavaScript to improve performance, need more information to perform calculations in the fastest way possible. It's important, for instance, to know whether the numbers should be treated as 32-bit integers or as 64-bit floats.

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
|`Math.sign(x)`| Returns -1 if the `x` is negative 0 if `x` is +0 or -0, or 1 if `x` is positive.||`Math.sinh(x)`| Returns the hyperbolic sine of `x`. |
|`Math.tanh(x)`| Returns the hyperbolic tangent of `x`. |
|`Math.trunc(x)`| Removes fraction digits from a float and returns an integer.|

It's beyond the scope of this book to explain each new method and what it does in detail. However, if you are looking for a reasonably common calculation, be sure to check the new `Math` methods before implementing it yourself.

## Summary

TODO

