# The Basics

ECMAScript 6 makes a large number of changes on top of ECMAScript 5. Some of the changes are larger, such as adding new types or syntax, while others are quite small, providing incremental improvements on top of the language. This chapter covers those incremental improvements that likely won't gain a lot of attention but provide some important functionality that may make certain types of problems easier to solve.

## Better Unicode Support

Prior to ECMAScript 6, JavaScript strings were based solely on the idea of 16-bit character encodings. All string properties and methods, such as `length` and `charAt()`, were based around the idea that every 16-bit sequence represented a single character. ECMAScript 5 allowed JavaScript engines to decide which of two encodings to use, either UCS-2 or UTF-16 (both encodings use 16-bit *code units*, making all observable operations the same). While it's true that all of the world's characters used to fit into 16 bits at one point in time, that is no longer the case.

Keeping within 16 bits wasn't possible for Unicode's stated goal of providing a globally unique identifier to every character in the world. These globally unique identifiers, called *code points*, are simply numbers starting at 0 (you might think of these as character codes, though there is a subtle difference). A character encoding is responsible for encoding a code point into code units that are internally consistent. While UCS-2 had a one-to-one mapping of code point to code unit, UTF-16 is more variable.

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

The first example of fully supporting UTF-16 is the `codePointAt()` method, which can be used to retrieve the Unicode code point that maps to a given position in a string. This method accepts the code unit position (not the character position) and returns an integer value:

```js
var text = "𠮷a";

console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
console.log(text.charCodeAt(2));    // 97

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 57271
console.log(text.codePointAt(2));   // 97
```

The `codePointAt()` method works in the same manner as `charCodeAt()` except for non-BMP characters. The first character in `text` is non-BMP and is therefore comprised of two code units, meaning the entire length of the string is 3 rather than 2. The `charCodeAt()` method returns only the first code unit for position 0 whereas `codePointAt()` returns the full code point even though it spans multiple code units. Both methods return the same value for positions 1 (the second code unit of the first character) and 2 (the `"a"`).

This method is the easiest way to determine if a given character is represented by one or two code points:

```js
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false
```

The upper bound of 16-bit characters is represented in hexadecimal as `FFFF`, so any code point above that number must be represented by two code units.

### String.fromCodePoint()

When ECMAScript provides a way to do something, it also tends to provide a way to do the reverse. You can use `codePointAt()` to retrieve the code point for a character in a string, while `String.fromCodePoint()` produces a single-character string for the given code point. For example:

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

ECMAScript 6 solves this problem by introducing an extended Unicode escape sequence where the hexadecimal numbers are contained within curly braces. This allows any number of hexadecimal characters to specify a single character:

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
W>         eval("'\\u{00FF1}'");
W>         return true;
W>     } catch (ex) {
W>         return false;
W>     }
W> }
W> ~~~~~~~~

### The normalize() Method

Another interesting aspect of Unicode is that different characters may be considered equivalent for the purposes of sorting or other comparison-based operations. There are two ways to define these relationships. First, *canonical equivalence* means that two sequences of code points are considered interchangeable in all respects. That even means that a combination of two characters can be canonically equivalent to one character. The second relationship is *compatibility*, meaning that two sequences of code points having different appearances but can be used interchangeably in certain situations.

The important thing to understand is that due to these relationships, it's possible to have two strings that represent fundamentally the same text and yet have them contain different code point sequences. For example, the character "æ" and the string "ae" may be used interchangeably even though they are different code points. These two strings would therefore be unequal in JavaScript unless they are normalized in some way.

ECMAScript 6 supports Unicode normalization forms through a new `normalize()` method on strings. This method optionally accepts a single string parameter indicating the Unicode normalization form to apply: `"NFC"` (default), `"NFD"`, `"NFKC"`, or `"NFKD"`. It's beyond the scope of this book to explain the differences between these four forms. Just keep in mind that, when comparing strings, both strings must be normalized to the same form. For example:

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

If you've never worried about Unicode normalization before, then you probably won't have much use for this method. However, knowing that it is available will help, should you ever end up working on an internationalized application.

### The Regular Expression u Flag

Many common string operations are accomplished by using regular expressions. However, as noted earlier, regular expressions also work on the basis of 16-bit code units each representing a single character. That's why the single character match in the earlier example didn't work. To address this problem, ECMAScript 6 defines a new flag for regular expressions: `u` for "Unicode".

When a regular expression has the `u` flag set, it switches modes to work on characters and not code units. That means the regular expression will no longer get confused about surrogate pairs in strings and can behave as expected. For example:

```js
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```

Adding the `u` flag allows the regular expression to correctly match the string by characters. Unfortunately, ECMAScript 6 does not natively have a way of determining how many code points are present in a string; fortunately, regular expressions with the `u` flag can be used to figure it out:

```js
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("𠮷bc"));   // 3
```

The regular expression in this example matches both whitespace and non-whitespace characters, and is applied globally with Unicode enabled. The `result` contains an array of matches when there's at least one match, so the array length ends up being the number of code points in the string.

W> Although this approach works, it's not very fast, especially when applied to long strings. Try to minimize counting code points whenever possible. Hopefully ECMAScript 7 will bring a more performant means by which to count code points.

Since the `u` flag is a syntax change, attempting to use it in non-compliant JavaScript engines means a syntax error is thrown. The safest way to determine if the `u` flag is supported is with a function:

```js
function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        return true;
    } catch (ex) {
        return false;
    }
}
```

This function uses the `RegExp` constructor to pass in the `u` flag as an argument. This is valid syntax even in older JavaScript engines, however, the constructor will throw an error if `u` isn't supported.

I> If your code still needs to work in older JavaScript engines, it's best to use the `RegExp` constructor exclusively when using the `u` flag. This will prevent syntax errors and allow you to optionally detect and use the `u` flag without aborting execution.

### Unicode Identifiers

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

## Other String Changes

JavaScript strings have always lagged behind similar features of other languages. It was only in ECMAScript 5 that strings finally gained a `trim()` method, and ECMAScript 6 continues extending strings with new functionality.

### includes(), startsWith(), endsWith()

Developers have used `indexOf()` as a way to identify strings inside of other strings since JavaScript was first introduced. ECMAScript 6 adds three new methods whose purpose is to identify strings inside of other strings:

* `includes()` - returns true if the given text is found anywhere within the string or false if not.
* `startsWith()` - returns true if the given text is found at the beginning of the string or false if not.
* `endsWith()` - returns true if the given text is found at the end of the string or false if not.

Each of these methods accepts two arguments: the text to search for and an optional location from which to start the search. When the second argument is omitted, `includes()` and `startsWith()` start search from the beginning of the string while `endsWith()` starts from the end. In effect, the second argument results in less of the string being searched. Here are some examples:

```js
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.includes("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.includes("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.includes("o", 8));          // false
```

These three methods make it much easier to identify substrings without needing to worry about identifying their exact position.

I> All of these methods return a boolean value. If you need to find the position of a string within another, use `indexOf()` or `lastIndexOf()`.

W> The `startsWith()`, `endsWith()`, and `includes()` methods will throw an error if you pass a regular expression instead of a string. This stands in contrast to `indexOf()` and `lastIndexOf()`, which both convert a regular expression argument into a string and then search for that string.

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

## Other Regular Expression Changes

Regular expressions are an important part of working with strings in JavaScript, and like many parts of the language, haven't really changed very much in recent versions. ECMAScript 6, however, makes several improvements to regular expressions to go along with the updates to strings.

### The Regular Expression y Flag

ECMAScript 6 standardized the `y` flag after it had been implemented in Firefox as a proprietary extension to regular expressions. The `y` (sticky) flag starts matching at the position specified by its `lastIndex` property. If there is no match at that location, then the regular expression stops matching. For example:

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text);
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

pattern.lastIndex = 1;
globalPattern.lastIndex = 1;
stickyPattern.lastIndex = 1;

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // Error! stickyResult is null
```

In this example, three regular expressions are used, one each with the `y` flag, the `g` flag, and no flags. When used the first time, all three regular expressions return the same value `"hello1 "` (with a space at the end). After that, the `lastIndex` property is changed to 1, meaning that the regular expression should start matching from the second character. The regular expression with no flags completely ignores the change to `lastIndex` and still matches `"hello1 "`; the regular expression with the `g` flag goes on to match `"hello2 "` because it is searching forward from the second character of the string ("e"); the sticky regular expression doesn't match anything beginning at the second character so `stickyResult` is `null`.

The sticky flag saves the index of the next character after the last match in `lastIndex` whenever an operation is performed. If an operation results in no match then `lastIndex` is set back to 0. This behavior is the same as the global flag:

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text);
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 7
console.log(stickyPattern.lastIndex);   // 7

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // "hello2 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 14
console.log(stickyPattern.lastIndex);   // 14
```

The value of `lastIndex` changed to 7 after the first call to `exec()` and to 14 after the second call for both the sticky and global patterns.

There are also a couple other subtle details to the sticky flag:

1. The `lastIndex` property is only honored when calling methods on the regular expression object such as `exec()` and `test()`. Passing the regular expression to a string method, such as `match()`, will not result in the sticky behavior.
1. When using the `^` character to match the start of a string, sticky regular expressions will only match from the start of the string (or start of line in multiline mode). So long as `lastIndex` is 0, the `^` makes a sticky regular expression no different from a non-sticky one. If `lastIndex` doesn't correspond to the beginning of the string (in single-line mode) or the beginning of a line (in multiline mode), the sticky regular expression will never match

As with other regular expression flags, you can detect the presence of `y` by using a property. The `sticky` property is set to true with the sticky flag is present and false if not:

```js
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```

The `sticky` property is read-only based on the presence of the flag and so cannot be changed in code.


Similar to the `u` flag, the `y` flag is a syntax change, so it will cause a syntax error in older JavaScript engines. You can use the same approach to detect support:

```js
function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        return true;
    } catch (ex) {
        return false;
    }
}
```

Also similar to `u`, if you need to use `y` in code that runs in older JavaScript engines, be sure to use the `RegExp` constructor when defining those regular expressions to avoid a syntax error.

### Duplicating Regular Expressions

In ECMAScript 5, you can duplicate regular expressions by passing them into the `RegExp` constructor, such as:

```js
var re1 = /ab/i,
    re2 = new RegExp(re1);
```

However, if you provide the second argument to `RegExp`, which specifies the flags for the regular expression, then an error is thrown:

```js
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");
```

If you execute this code in an ECMAScript 5 environment, you'll get an error stating that the second argument cannot be used when the first argument is a regular expression. ECMAScript 6 changed this behavior such that the second argument is allowed and will override whichever flags are present on the first argument. For example:

```js
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");


console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false

```

In this code, `re1` has the case-insensitive `i` flag present while `re2` has only the global `g` flag. The `RegExp` constructor duplicated the pattern from `re1` and then substituted `g` for `i`. If the second argument was missing then `re2` would have the same flags as `re1`.

### The `flags` Property

In ECMAScript 5, it's possible to get the text of the regular expression by using the `source` property, but to get the flag string requires parsing the output of `toString()`, such as:

```js
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() is "/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g"
```

ECMAScript 6 adds a `flags` property to go along with `source`. Both properties are prototype accessor properties with only a getter assigned (making them read-only). The addition of `flags` makes it easier to inspect regular expressions for both debugging and inheritance purposes.

A late addition to ECMAScript 6, the `flags` property returns the string representation of any flags applied to a regular expression. For example:

```js
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
```

Using `source` and `flags` together allow you to extract just the pieces of the regular expression that are necessary without needing to parse the regular expression string directly.


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

        // value exists here with a value of undefined

        return null;
    }

    // value exists here with a value of undefined
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

        // value doesn't exist here

        return null;
    }

    // value doesn't exist here
}
```

This function now behaves much closer to other C-based languages. The variable `value` is declared using `let` instead of `var`. That means the declaration is not hoisted to the top, and the variable `value` is destroyed once execution has flowed out of the `if` block. If `condition` evaluates to false, then `value` is never declared or initialized.

Perhaps one of the areas where developers most want block level scoping of variables is with `for` loops. It's not uncommon to see code such as this:

```js
for (var i=0; i < items.length; i++) {
    process(items[i]);
}

// i is still accessible here and is equal to items.length
```

In other languages, where block level scoping is the default, code like this works as intended. In JavaScript, the variable `i` is still accessible after the loop is completed because the `var` declaration was hoisted. Using `let` allows you to get the intended behavior:

```js
for (let i=0; i < items.length; i++) {
    process(items[i]);
}

// i is not accessible here
```

In this example, the variable `i` only exists within the `for` loop. Once the loop is complete, the variable is destroyed and is no longer accessible elsewhere.

A> ### Using let in loops
A>
A> The behavior of `let` inside of loops is slightly different than with other blocks. Instead of creating a variable that is used with each iteration of the loop, each iteration actually gets its own variable to use. This is to solve an old problem with JavaScript loops. Consider the following:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  var funcs = [];
A>
A>  for (var i=0; i < 10; i++) {
A>      funcs.push(function() { console.log(i); });
A>  }
A>
A>  funcs.forEach(function(func) {
A>      func();     // outputs the number "10" ten times
A>  });
A> ~~~~~~~~
A>
A> This code will output the number `10` ten times in a row. That's because the variable `i` is shared across each iteration of the loop, meaning the closures created inside the loop all hold a reference to the same variable. The variable `i` has a value of `10` once the loop completes, and so that's the value each function outputs.
A>
A> To fix this problem, developers use immediately-invoked function expressions (IIFEs) inside of loops to force a new copy of the variable to be created:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  var funcs = [];
A>
A>  for (var i=0; i < 10; i++) {
A>      funcs.push((function(value) {
A>          return function() {
A>              console.log(value);
A>          }
A>      }(i)));
A>  }
A>
A>  funcs.forEach(function(func) {
A>      func();     // outputs 0, then 1, then 2, up to 9
A>  });
A> ~~~~~~~~
A>
A> This version of the example uses an IIFE inside of the loop. The `i` variable is passed to the IIFE, which creates its own copy and stores it as `value`. This is the value used of the function for that iteration, so calling each function returns the expected value.
A>
A> A `let` declaration does this for you without the IIFE. Each iteration through the loop results in a new variable being created and initialized to the value of the variable with the same name from the previous iteration. That means you can simplify the process by using this code:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  var funcs = [];
A>
A>  for (let i=0; i < 10; i++) {
A>      funcs.push(function() { console.log(i); });
A>  }
A>
A>  funcs.forEach(function(func) {
A>      func();     // outputs 0, then 1, then 2, up to 9
A>  })
A> ~~~~~~~~
A>
A>  This code works exactly like the code that used `var` and an IIFE but is, arguably, cleaner.

Unlike `var`, `let` has no hoisting characteristics. A variable declared with `let` cannot be accessed until after the `let` statement. Attempting to do so results in a reference error:

```js
if (condition) {
    console.log(value);     // ReferenceError!
    let value = "blue";
}
```

In this code, the variable `value` is defined and initialized using `let`, but that statement is never executed because the previous line throws an error. The issue is that `value` exists in what has become known as the *temporal dead zone* (TDZ). The TDZ is never named explicitly in the specification, but is a term often used to describe the non-hoisting behavior of `let`.

When a JavaScript parser looks through the upcoming block to find variable declarations, it results in either hoisting the declaration (for `var`) or placing the declaration in the TDZ. Any attempt to access a variable in the TDZ results in a runtime error. Only once execution flows to the variable declaration is that variable removed from the TDZ and therefore safe to use.

The same is true anytime you attempt to use a variable declared with `let` inside of the same block prior to it being defined. Even the normally safe-to-use `typeof` operator isn't safe:

```js
if (condition) {
    console.log(typeof value);     // ReferenceError!
    let value = "blue";
}
```

Here, `typeof value` throws the same error as the previous example. You cannot use a `let` variable before its declaration within the same block. However, you can use `typeof` outside of the block:

```js
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

The variable `value` isn't in the TDZ when the `typeof` operation executes because it occurs outside of the block in which `value` is declared. That means there is no `value` binding and `typeof` simply returns `"undefined"`.

If an identifier has already been defined in the block, then using the identifier in a `let` declaration causes an error to be thrown. For example:

```js
var count = 30;

// Syntax error
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

A> ### Global let Declarations
A>
A> There is the potential for naming collisions when using `let` in the global scope because the global object has predefined properties. Using `let` to define a variable that shares a name with a property of the global object can produce an error because global object properties may be nonconfigurable. Since `let` doesn't allow redefinition of the same identifier in the same scope, it's not possible to shadow nonconfigurable global properties. Attempting to do so will result in an error. For example:
A>
A> {:lang="js"}
A> ~~~~~~~~
A>  let RegExp = "Hello!";          // ok
A>  let undefined = "Hello!";       // throws error
A> ~~~~~~~~
A>
A> The first line of this example redefines the global `RegExp` as a string. Even though this would be problematic, there is no error thrown. The second line throws an error because `undefined` is a nonconfigurable own property of the global object. Since its definition is locked down by the environment, the `let` declaration is illegal.
A>
A> It's unusual to use `let` in the global scope, but if you do, it's important to understand this situation.

The long term intent is for `let` to replace `var`, as the former behaves more like variable declarations in other languages. If you are writing JavaScript that will execute only in an ECMAScript 6 or higher environment, you may want to try using `let` exclusively and leaving `var` for other scripts that require backwards compatibility.

I> Since `let` declarations are *not* hoisted to the top of the enclosing block, you may want to always place `let` declarations first in the block so that they are available to the entire block.

### Constant declarations

Another new way to define variables is to use the `const` declaration syntax. Variables declared using `const` are considered to be *constants*, so the value cannot be changed once set. For this reason, every `const` variable must be initialized. For example:

```js
// Valid constant
const MAX_ITEMS = 30;

// Syntax error: missing initialization
const NAME;
```

Constants are also block-level declarations, similar to `let`. That means constants are destroyed once execution flows out of the block in which they were declared, and declarations are not hoisted to the top of the block. For example:

```js
if (condition) {
    const MAX_ITEMS = 5;

    // more code
}

// MAX_ITEMS isn't accessible here
```

In this code, the constant `MAX_ITEMS` is declared within an `if` statement. Once the statement finishes executing, `MAX_ITEMS` is destroyed and is not accessible outside of that block.

Also similar to `let`, an error is thrown whenever a `const` declaration is made with an identifier for an already-defined variable in the same scope. It doesn't matter if that variable was declared using `var` (for global or function scope) or `let` (for block scope). For example:

```js
var message = "Hello!";
let age = 25;

// Each of these would cause an error given the previous declarations
const message = "Goodbye!";
const age = 30;
```

The big difference between `let` and `const` is that attempting to assign to a previously defined constant will throw an error in both strict and non-strict modes:

```js
const MAX_ITEMS = 5;

MAX_ITEMS = 6;      // throws error
```

W> Several browsers implement pre-ECMAScript 6 versions of `const`. Implementations range from being simply a synonym for `var` (allowing the value to be overwritten) to actually defining constants but only in the global or function scope. For this reason, be especially careful with using `const` in a production system. It may not be providing you with the functionality you expect.

## Destructuring Assignment

JavaScript developers spend a lot of time pulling data out of objects and arrays. It's not uncommon to see code such as this:

```js
var options = {
        repeat: true,
        save: false
    };

// later

var localRepeat = options.repeat,
    localSave = options.save;
```

Frequently, object properties are stored into local variables for more succinct code and easier access. ECMAScript 6 makes this easy by introducing *destructuring assignment*, which systematically goes through an object or array and stores specified pieces of data into local variables.

W> If the right side value of a destructuring assignment evaluates to `null` or `undefined`, an error is thrown.

### Object Destructuring

Object destructuring assignment syntax uses an object literal on the left side of an assignment operation. For example:

```js
var options = {
        repeat: true,
        save: false
    };

// later

var { repeat: localRepeat, save: localSave } = options;

console.log(localRepeat);       // true
console.log(localSave);         // false
```

In this code, the value of `options.repeat` is stored in a variable called `localRepeat` and the value of `options.save` is stored in a variable called `localSave`. These are both specified using the object literal syntax where the key is the property to find on `options` and the value is the variable in which to store the property value.

I> If the property with the given name doesn't exist on the object, then the local variable gets a value of `undefined`.

If you want to use the property name as the local variable name, you can omit the colon and the identifier, such as:

```js
var options = {
        repeat: true,
        save: false
    };

// later

var { repeat, save } = options;

console.log(repeat);        // true
console.log(save);          // false
```

Here, two local variables called `repeat` and `save` are created. They are initialized with the value of `options.repeat` and `options.save`, respectively. This shorthand is helpful when there's no need to have different variable names.

Destructuring can also handle nested objects, such as the following:

```js
var options = {
        repeat: true,
        save: false,
        rules: {
            custom: 10,
        }
    };

// later

var { repeat, save, rules: { custom }} = options;

console.log(repeat);        // true
console.log(save);          // false
console.log(custom);        // 10
```

In this example, the `custom` property is embedded in another object. The extra set of curly braces allows you to descend into a nested object and pull out its properties.

A> #### Syntax Gotcha
A>
A> If you try use destructuring assignment without a `var`, `let`, or `const`, you may be surprised by the result:
A>
A> {:lang="js"}
A> ~~~~~~~~
A> // syntax error
A> { repeat, save, rules: { custom }} = options;
A> ~~~~~~~~
A>
A> This causes a syntax error because the opening curly brace is normally the beginning of a block and blocks can't be part of assignment expressions.
A>
A> The solution is to wrap the left side literal in parentheses:
A>
A> {:lang="js"}
A> ~~~~~~~~
A> // no syntax error
A> ({ repeat, save, rules: { custom }}) = options;
A> ~~~~~~~~
A>
A> This now works without any problems.

### Array Destructuring

Similarly, you can destructure arrays using array literal syntax on the left side of an assignment operation. For example:

```js
var colors = [ "red", "green", "blue" ];

// later

var [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

In this example, array destructuring pulls out the first and second values in the `colors` array. Keep in mind that the array itself isn't changed in any way.

Similar to object destructuring, you can also nest array destructuring. Just use another set of square brackets to descend into a subarray:

```js
var colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// later

var [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

Here, the `secondColor` variable refers to the `"green"` value inside of the `colors` array. That item is contained within a second array, so the extra square brackets around `secondColor` in the destructuring assignment is necessary.

### Mixed Destructuring

It's possible to mix objects and arrays together in a destructuring assignment expression using a mix of object and array literals. For example:

```js
var options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

var { repeat, save, colors: [ firstColor, secondColor ]} = options;

console.log(repeat);            // true
console.log(save);              // false
console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

This example extracts two property values, `repeat` and `save`, and then two items from the `colors` array, `firstColor` and `secondColor`. Of course, you could also choose to retrieve the entire array:

```js
var options = {
        repeat: true,
        save: false,
        colors: [ "red", "green", "blue" ]
    };

var { repeat, save, colors } = options;

console.log(repeat);                        // true
console.log(save);                          // false
console.log(colors);                        // "red,green,blue"
console.log(colors === options.colors);     // true
```

This modified example retrieves `options.colors` and stores it in the `colors` variable. Notice that `colors` is a direct reference to `options.colors` and not a copy.

Mixed destructuring is very useful for pulling values out of JSON configuration structures without navigating the entire structure.

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

ECMAScript 6 took things a step further by reintroducing an octal literal notation, along with a binary literal notation. Both of these notations take a hint from the hexadecimal literal notation of prepending `0x` or `0X` to a value. The new octal literal format begins with `0o` or `0O` while the new binary literal format begins with `0b` or `0B`. Each literal type must be followed by one or more digits, 0-7 for octal, 0-1 for binary. Here's an example:

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

### isFinite() and isNaN()

JavaScript has long had a couple of global methods for identifying certain types of numbers:

* `isFinite()` determines if a value represents a finite number (not `Infinity` or `-Infinity`)
* `isNaN()` determines if a value is `NaN` (since `NaN` is the only value that is not equal to itself)

Although intended to work with numbers, these methods are capable of inferring a numeric value from any value that is passed in. That means both methods can return incorrect results when passed a value that isn't a number. For example:

```js
console.log(isFinite(25));      // true
console.log(isFinite("25"));    // true

console.log(isNaN(NaN));        // true
console.log(isNaN("NaN"));      // true
```

Both `isFinite()` and `isNaN()` pass their arguments through `Number()` to get a numeric value and then perform their comparisons on that numeric value rather than the original. This confusing outcome can lead to errors when value types are not checked before being used with one of these methods.

ECMAScript 6 adds two new methods that perform the same comparison but only for number values: `Number.isFinite()` and `Number.isNaN()`. These methods always return `false` when passed a non-number value and return the same values as their global counterparts when passed a number value:

```js
console.log(isFinite(25));              // true
console.log(isFinite("25"));            // true
console.log(Number.isFinite(25));       // true
console.log(Number.isFinite("25"));     // false

console.log(isNaN(NaN));                // true
console.log(isNaN("NaN"));              // true
console.log(Number.isNaN(NaN));         // true
console.log(Number.isNaN("NaN"));       // false
```

In this code, `Number.isFinite("25")` returns `false` even though `isFinite("25")` returns `true`; likewise `Number.isNaN("NaN")` returns `false` even though `isNaN("NaN")` returns `true`.

These two new methods are aimed at eliminating certain types of errors that can be caused when non-number values are used with `isFinite()` and `isNaN()` without dramatically changing the language.

### parseInt() and parseFloat()

The global functions `parseInt()` and `parseFloat()` now also reside at `Number.parseInt()` and `Number.parseFloat()`. These functions behave exactly the same as the global functions of the same name. The only purpose in making this move is to categorize purely global functions that clearly relate to a specific data type. Since these functions both create numbers from strings, they are now on `Number` along with the other functions that relate to numbers.

### Working with Integers

A lot of confusion has been caused over the years related to JavaScript's single number type that is used to represent both integers and floats. The language goes through great pains to ensure that developers don't need to worry about the details, but problems still leak through from time to time. ECMAScript 6 seeks to address this by making it easier to identify and work with integers.

#### Identifying Integers

The first addition is `Number.isInteger()`, which allows you to determine if a value represents an integer in JavaScript. Since integers and floats are stored differently, the JavaScript engine looks at the underlying representation of the value to make this determination. That means numbers that look like floats might actually be stored as integers and therefore return `true` from `Number.isInteger()`. For example:

```js
console.log(Number.isInteger(25));      // true
console.log(Number.isInteger(25.0));    // true
console.log(Number.isInteger(25.1));    // false
```

In this code, `Number.isInteger()` returns `true` for both `25` and `25.0` even though the latter looks like a float. Simply adding a decimal point to a number doesn't automatically make it a float in JavaScript. Since `25.0` is really just `25`, it is stored as an integer. The number `25.1`, however, is stored as a float because there is a fraction value.

#### Safe Integers

However, all is not so simple with integers. JavaScript can only accurately represent integers between -2^53^ and 2^53^, and outside of this "safe" range, binary representations end up reused for multiple numeric values. For example:

```js
console.log(Math.pow(2, 53));      // 9007199254740992
console.log(Math.pow(2, 53) + 1);  // 9007199254740992
```

This example doesn't contain a typo, two different numbers end up represented by the same JavaScript integer. The effect becomes more prevalent the further the value is outside of the safe range.

ECMAScript 6 introduces `Number.isSafeInteger()` to better identify integers that can accurately be represented in the language. There is also `Number.MAX_SAFE_INTEGER` and `Number.MIN_SAFE_INTEGER` that represent the upper and lower bounds of the same range, respectively. The `Number.isSafeInteger()` method ensures that a value is an integer and falls within the safe range of integer values:

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
|`Math.sign(x)`| Returns -1 if the `x` is negative, 0 if `x` is +0 or -0, or 1 if `x` is positive.|
|`Math.sinh(x)`| Returns the hyperbolic sine of `x`. |
|`Math.tanh(x)`| Returns the hyperbolic tangent of `x`. |
|`Math.trunc(x)`| Removes fraction digits from a float and returns an integer.|

It's beyond the scope of this book to explain each new method and what it does in detail. However, if you are looking for a reasonably common calculation, be sure to check the new `Math` methods before implementing it yourself.

## Summary

ECMAScript 6 makes a lot of changes, both large and small, to JavaScript. Some of the smaller changes detailed in this chapter will likely be overlooked by many but they are just as important to the evolution of the language as the big changes.

Full Unicode support allows JavaScript to start dealing with UTF-16 characters in logical ways. The ability to transfer between code point and character via `codePointAt()` and `String.fromCodePoint()` is an important step for string manipulation. The addition of the regular expression `u` flag makes it possible to operate on code points instead of 16-bit characters, and the `normalize()` method allows for more appropriate string comparisons.

Additional methods for working with strings were added, allowing you to more easily identify substrings no matter where they are found, and more functionality was added to regular expressions. The `Object.is()` method performs strict equality on any value, effectively becoming a safer version of `===` when dealing with special JavaScript values.

The `let` and `const` block bindings introduce lexical scoping to JavaScript. These declarations are not hoisted and only exist within the block in which they are declared. That means behavior that is more like other languages and less likely to cause unintentional errors, as variables can now be declared exactly where they are needed. It's expected that the majority of JavaScript code going forward will use `let` and `const` exclusively, effectively making `var` a deprecated part of the language.

ECMAScript 6 makes it easier to work with numbers through the introduction of new syntax and new methods. The binary and octal literal forms allow you to embed numbers directly into source code while keeping the most appropriate representation visible. `Number.isFinite()` and `Number.isNaN()` are introduced as safer versions of their respective global methods. You can more easily identify integers using `Number.isInteger()` and `Number.isSafeInteger()`, as well as perform more mathematical operations thanks to new methods on `Math`.

Though many of these changes are small, they will make a significant difference in the lives of JavaScript developers for years to come. Each change addresses a particular concern that can otherwise require a lot of custom code to address. By building this functionality into the language, developers can focus on writing the code for their product rather than low-level utilities.
