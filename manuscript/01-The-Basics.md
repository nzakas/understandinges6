# The Basics



## Better Unicode Support

Prior to ECMAScript 6, JavaScript strings were based solely on the idea of 16-bit character encodings. All string properties and methods, such as `length` and `charAt()`, were based around the idea that every 16-bit sequence represented a single character. ECMAScript 5 allowed JavaScript engines to decide which of two encodings to use, either UCS-2 or UTF-16 (both encoding using 16-bit *code units*, making all observable operations the same). While it's true that all of the world's characters used to fit into 16 bits at one point in time, that is no longer the case thanks.

Keeping within 16 bits wasn't possible for Unicode's stated goal of providing a globally unique identifier to every character in the world. These globally unique identifiers are called *code points* are simply numbers starting at 0 (you might think of these as character codes, but there is subtle difference). A character encoding is responsible for encoding a code point into code units that are internally consistent. While UCS-2 had a one-to-one mapping of code point to code unit, UTF-16 is more variable.

The first 2^16 code points are represented as single 16-bit code units in UTF-16. This is called the *Basic Multilingual Plane* (BMP). Everything beyond that range is considered to be in a *supplementary plane*, where the code points can no longer be represented in just 16-bits. UTF-16 solves this problem by introducing *surrogate pairs* in which a single code point is represented by two 16-bit code units. That means any single character in a string can be either one code unit (for BMP, total of 16 bits) or two (for supplementary plane characters, total of 32 bits).

ECMAScript 5 kept all operations as working on 16-bit code units, meaning that you could get unexpected results from strings containing surrogate pairs. For example:

{lang=js}
    var text = "𠮷";

    console.log(text.length);           // 2
    console.log(/^.$/.test(text));      // false
    console.log(text.charAt(0));        // ""
    console.log(text.charAt(1));        // ""
    console.log(text.charCodeAt(0));    // 55362
    console.log(text.charCodeAt(1));    // 57271

In this example, a single Unicode character is represented using surrogate pairs, and as such, the JavaScript string operations treat the string as having two 16-bit characters. That means `length` is 2, a regular expression trying to match a single character fails, and `charAt()` is unable to return a valid character string. The `charCodeAt()` method returns the appropriate 16-bit number for each code unit, but that is the closest you could get to the real value in ECMAScript 5.

ECMAScript 6 enforces encoding of strings in UTF-16. Standardizing on this character encoding means that the language can now support functionality designed to work specifically with surrogate pairs.

### The codePointAt() Method

The first example of fully supporting UTF-16 is the `codePointAt()` method, which can be used to retrieve the Unicode code point that maps to a given character. This method accepts the character position (not the code unit position) and returns an integer value:

{lang=js}
    var text = "𠮷a";

    console.log(text.codePointAt(0));   // 134071
    console.log(text.codePointAt(1));   // 97

The value returned is the Unicode code point value. For BMP characters, this will be the same result as using `charCodeAt()`, so the `"a"` is returns 97. This method is the easiest way to determine if a given character is represented by one or two code points:

{lang=js}
    function is32Bit(c) {
        return c.codePointAt(0) > 0xFFFF;
    }

    console.log(is32Bit("𠮷"));         // true
    console.log(is32Bit("a"));          // false

The upper bound of 16-bit characters is represented in hexadecimal as `FFFF`, so any code point above that number must be represented by two code units.

### String.fromCodePoint()

When ECMAScript provides a way to do something, it also tends to provide a way to do the reverse. You can use `codePointAt()` to retrieve the code point for a character in a string while `String.fromCodePoint()` produces a single-character string for the given code point. For example:

{lang=js}
    console.log(String.fromCodePoint(134071));  // "𠮷"

You can think of `String.fromCodePoint()` as a more complete version of `String.fromCharCode()`. Each method has the same result for all characters in the BMP; the only difference is with characters outside of that range.

### Escaping Non-BMP Characters

ECMAScript 5 allows strings to contain 16-bit Unicode characters represented by an *escape sequence*. The escape sequence is the `\u` followed by four hexadecimal values. For example, the escape sequence `\u0061` represents the letter `"a"`:

{lang=js}
    console.log("\u0061");      // "a"

If you try to use an escape sequence with a number past `FFFF`, the upper bound of the BMP, then you can get some surprising results:

{lang=js}
    console.log("\u20BB7");     // "₻7"

Since Unicode escape sequences were defined as always having exactly four hexadecimal characters, ECMAScript evaluates `\u20BB7` as two characters: `\u20BB` and `"7"`. The first character is unprintable and the second is the number 7.

ECMAScript 6 solves this problem by introducing an extended Unicode escape sequence where the hexadecimal numbers are contained within curly braces. This allows up to 8 hexadecimal characters to specify a single character:

{lang=js}
    console.log("\u{20BB7}");     // "𠮷"

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

ECMAScript 6 supports the four Unicode normalization forms through a new `normalize()` method on strings. This method optionally accepts a single parameter, one of `"NFC"` (default), `"NFD"`, `"NFKC"`, or `"NFKD"`. It's beyond the scope of this book explain the differences between these four forms. Just keep in mind that, in order to be used, you must normally both strings that are being compared to the same form. For example:

{lang=js}
    var normalized = values.map(text => text.normalize());
    normalized.sort(function(first, second) {
        if (first < second) {
            return -1;
        } else if (first === second) {
            return 0;
        } else {
            return 1;
        }
    });

In this code, the strings in a `values` array are converted into a normalized form so that the array can be sorted appropriately. You can accomplish the sort on the original array by calling `normalize()` as part of the comparator:

{lang=js}
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

Once again, the most important thing to remember is that both values must be normalized in the same way. These examples have used the default, NFC, but you can just as easily specify one of the others:

{lang=js}
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

If you've never worried about Unicode normalization before, then you probably won't have much use for this method. However, knowing that it is available will help should you ever end up working on an internationalized application.

### The Regular Expression u Flag

Many common string operations are accomplished by using regular expressions. However, as noted earlier, regular expressions also work on the basis of 16-bit code units each representing a single character. That's why the single character match in the earlier example didn't work. To address this problem, ECMAScript 6 defines a new flag for regular expressions: `u` for "Unicode".

When a regular expression has the `u` flag set, it switches modes to work on characters and not code units. That means the regular expression will no longer get confused about surrogate pairs in strings and can behave as expected. For example:

{lang=js}
    var text = "𠮷";

    console.log(text.length);           // 2
    console.log(/^.$/.test(text));      // false
    console.log(/^.$/u.test(text));     // true

Adding the `u` flag allows the regular expression to correctly match the string by characters. Unfortunately, ECMAScript 6 does not have a way of determining how many code points are present in a string; fortunately, regular expressions can be used to figure it out:

{lang=js}
    function codePointLength(text) {
        var result = text.match(/[\s\S]/gu);
        return result ? result.length : 0;
    }

    console.log(codePointLength("abc"));    // 3
    console.log(codePointLength("𠮷bc"));   // 3

The regular expression in this example matches a both whitespace and non-whitespace characters, and is applied globally with Unicode enabled. The `result` contains an array of matches when there's at least one match, so the array length ends up being the number of code points in the string.

W> Although this approach works, it's not very fast, especially when applied to long strings. Try to minimize counting code points whenever possible. Hopefully ECMAScript 7 will bring a more performant means by which to count code points.

## More string methods

JavaScript strings have always lagged behind similar features of other languages. It was only in ECMAScript 5 that strings finally gained a `trim()` method, and ECMAScript 6 continues extending strings with new functionality.

## contains(), startsWith(), endsWith()

Developers have used `indexOf()` as a way to identify strings inside of other strings since time JavaScript was first introduced. ECMAScript 6 adds three new methods whose purpose is to identify strings inside of other strings:

* `contains()` - returns true if the given text is found anywhere within the string or false if not.
* `startsWith()` - returns true if the given text is found at the beginning of the string or false if not.
* `endsWith()` - returns true if the given text is found at the end of the string or false if not.

Each of these methods accepts two arguments: the text to search for and an optional location from which to start the search. When the second argument is omitted, `contains()` and `startsWith()` start search from the beginning of the string while `endsWith()` starts from the end. In effect, the second argument results in less of the string being searched. Here are some examples:

{lang=js}
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

These three methods make it much easier to identifier substrings without needing to worry about identifying their exact position.

I> All of these methods return a boolean value. If you need to find the position of a string within another, use `indexOf()` or `lastIndexOf()`.

### repeat()

ECMAScript 6 also adds a `repeat()` method to strings. This method accepts a single argument, which is the number of times to repeat the string, and returns a new string that has the original string repeated the specified number of times. For example:

{lang=js}
    console.log("x".repeat(3));         // "xxx"
    console.log("hello".repeat(2));     // "hellohello"
    console.log("abc".repeat(4));       // "abcabcabcabc"

This method is really a convenience function above all else, which can be especially useful when dealing with text manipulation. One example where this functionality comes in useful is with code formatting utilities where you need to create indentation levels:

{lang=js}
    // indent using a specified number of spaces
    var indent = " ".repeat(size),
        indentLevel = 0;

    // whenever you increase the indent
    var newIndent = indent.repeat(++indentLevel);


## Object.is()

When you want to compare two values, you're probably used to using either the equals operator (`==`) or the identically equals operator (`===`). Many prefer to use the latter to avoid type coercion during the comparison. However, even the identically equals operator isn't entirely accurate. For example, the values +0 and -0 are considered equal by `===` even though they are represented differently in the JavaScript engine. Also `NaN === NaN` returns `false`, which necessitates using `isNaN()` to detect `NaN` properly.

ECMAScript 6 introduces `Object.is()` to make up for the remaining quirks of the identically equals operator. This method accepts two arguments and returns `true` if the values are equivalent. Two values are considered equivalent they are of the same type and have the same value. In many cases, `Object.is()` works the same as `===`. The only differences are that +0 and -0 are considered not equivalent and `NaN` is considered equivalent to `NaN`. Here are some examples:

    console.log(+0 == -0);              // true
    console.log(+0 === -0);             // true
    console.log(Object.is(+0, -0));     // true

    console.log(NaN == NaN);            // false
    console.log(NaN === NaN);           // false
    console.log(Object.is(NaN, NaN));   // true

    console.log(5 == 5);                // true
    console.log(5 == "5");              // true
    console.log(5 === 5);               // true
    console.log(5 === "5");             // false
    console.log(Object.is(5, 5));       // true
    console.log(Object.is(5, "5"));     // false

In most cases you will probably still want to use `==` or `===` for comparing values, as special cases covered by `Object.is()` may not affect you directly.



## Block bindings

Traditionally, one of the tricky parts of JavaScript has been the way that `var` declarations work. In most C-based languages, variables are created at the spot where the declaration occurs. In JavaScript, however, this is not the case. Variables declared using `var` are *hoisted* to the top of the function (or global scope) regardless of where the actual declaration occurs. For example:

    function getValue(condition) {

        if (condition) {
            var value = "blue";

            // other code

            return value;
        } else {

            return null;
        }
    }

If you are unfamiliar with JavaScript, you might expect that the variable `value` is only defined if `condition` evaluates to true. In fact, the variable `value` is declared regardless. The JavaScript engine changes the function to look like this:

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

The declaration of `value` is moved to the top (hoisted) while the initialization remains in the same spot. That means the variable `value` is actually still accessible from within the `else` clause, it just has a value of `undefined` because it hasn't been initialized.

It often takes new JavaScript developers some time to get used to declaration hoisting and this unique behavior can end up causing bugs. For this reason, ECMAScript 6 introduces block level scoping options to make the control of variable lifecycle of little more powerful.

### Let declarations

The `let` declaration syntax is the same as for `var`. You can basically replace `var` with `let` to declare a variable but keep its scope to the current code block. For example:

    function getValue(condition) {

        if (condition) {
            let value = "blue";

            // other code

            return value;
        } else {

            return null;
        }
    }

This function now behaves much closer to other C-based languages. The variable `value` is declared using `let` instead of `var`. That means the declaration is not hoisted to the top, and the variable `value` is destroyed once execution has flowed out of the `if` block. If `condition` evaluates to false, then `value` is never declared or initialized.

Perhaps one of the areas where developers most want block level scoping of variables is with `for` loops. It's not uncommon to see code such as this:

    for (var i=0; i < items.length; i++) {
        process(items[i]);
    }

    // i is still accessible here

In other languages, where block level scoping is the default, code like this works as intended. In JavaScript, the variable `i` is still accessible after the loop is completed because the `var` declaration was hoisted. Using `let` allows you to get the intended behavior:

    for (let i=0; i < items.length; i++) {
        process(items[i]);
    }

    // i is not accessible here

In this example, the variable `i` only exists within the `for` loop. Once the loop is complete, the variable is destroyed and is no longer accessible elsewhere.

One tricky thing to keep in mind about `let` is that the declarations are actually hoisted to the top of the block in which they reside. For example:

    if (condition) {
        console.log(value);     // undefined
        let value = "blue";
    }

In this code, the variable `value` is defined and initialized using `let`. The declaration is actually hoisted to the top of the `if` block and so it's possible to access the variable `value` before it appears to be declared (if it weren't declared, using `console.log(value)` would result in an error being thrown). In fact, the JavaScript engine has interpreted the code as if it were written like this:

    if (condition) {
        let value;
        console.log(value);     // undefined
        value = "blue";
    }

Similar to `var`, the declaration is hoisted while the initialization stays in place. That means the value of `value` is actually `undefined` when `console.log(value)` is executed.

If an identifier has already been defined in the block, then using the identifier in a `let` declaration causes an error to be thrown. For example:

    var count = 30;

    // Throws an error
    let count = 40;

In this example, `count` is declared twice, once with `var` and once with `let`. Because `let` will not redefine an identifier that already exists in the same scope, the declaration throws an error. No error is thrown if a `let` declaration creates a new variable in a scope with the same name as a variable in the containing scope, such as:

    var count = 30;

    // Does not throw an error
    if (condition) {

        let count = 40;

        // more code
    }

Here, the `let` declaration will not throw an error because it is creating a new variable called `count` within the `if` statement. This new variable shadows the global `count`, preventing access to it from within the `if` block.

The intent of `let` is to replace `var` long term, as the former behaves more like variable declarations in other languages. If you are writing JavaScript that will execute only in an ECMAScript 6 or higher environment, you may want to try using `let` exclusively and leaving `var` for other scripts that require backwards compatibility.

Note: Since all `let` declarations are hoisted to the top of the enclosing block, you may want to always place `let` declarations first in the block.

### Constant declarations

Another new way to define variables is to use the `const` declaration syntax. Variables declared using `const` are considered to be *constants*, so the value cannot be changed once set. For this reason, every `const` variable must be initialized. For example:

    // Valid constant
    const MAX_ITEMS = 30;

    // Syntax error: missing initialization
    const NAME;

Constants are also block-level declarations, similar to `let`. That means constants are destroyed once execution flows out of the block in which they were declared and declarations are hoisted to the top of the block. For example:

    if (condition) {
        const MAX_ITEMS = 5;

        // more code
    }

    // MAX_ITEMS isn't accessible here

In this code, the constant `MAX_ITEMS` is declared within and `if` statement. Once the statement finishes executing, `MAX_ITEMS` is destroyed and is not accessible outside of that block.

Also similar to `let`, an error is thrown whenever a `const` declaration is made with an identifier for an already-defined variable in the same scope. It doesn't matter if that variable was declared using `var` (for global or function scope) or `let` (for block scope). For example:

    var message = "Hello!";
    let age = 25;

    // Each of these would cause an error given the previous declarations
    const message = "Goodbye!";
    const age = 30;

Note: Several browsers implement pre-ECMAScript 6 versions of `const`. Implementations range from being simply a synonm for `var` (allowing the value to be overwritten) to actually defining constants but only in the global or function scope. for this reason, be especially careful with using `const` in a production system. It may not be providing you with the functionality you expect.






## Statements


### for-of

If you want to iterate over values in an array, you have a couple of options in ECMAScript five and earlier. You can, of course, use the tried-and-true `for` loop:

    var i, len;

    for (i=0, len=values.length; i < len; i++) {
        process(values[i]);
    }

You can also use the `forEach()` method:

    values.forEach(function(value) {
        process(value);
    });

However, these approaches have some limitations. The `for` loop requires some setup in the form of variables; `forEach()` only works on arrays and not other array-like objects. ECMAScript 6 solves for all of these problems with the new statement.

The `for-of` statement is designed to iterate over values in any array-like object. Instead of setting up a variable to monitor which item in the array to process, you simply use one variable that is assigned each subsequent value in the array until all values have been through the loop. For example:

    for (let value of values) {
        process(value);
    }

In this code, the `for-of` loop is defined using the variable `value` to hold each value in the array `values`. The loop starts from the first value and move sequentially through the array until reaching the end. Because the statement works with any array like values, you can also use it on objects such as `arguments` and DOM `NodeList` objects (neither of which have the `forEach()` method):

    // Print out all arguments
    function printArgs() {
        for (let arg of arguments) {
            console.log(arg);
        }
    }

    // Iterate over all <div> elements
    var divs = document.getElementsByTagName("div");

    for (let div of divs) {
        div.innerHTML = "Processed...";
    }

The capability to work with any array-like object makes `for-of` a big improvement over other iteration methods that had to account for different types of objects.

Note: The `for-of` statement works with any iterable object, which includes arrays, arguments, DOM `NodeList` objects, and generators. It will not work with regular objects.

The `for-of` statement is available in Firefox 13+.








## Changes to String

JavaScript is used quite frequently for string manipulation, however, string support has who leave evolved very slowly. ECMAScript 5 finally added a `trim()` methodbut aside from that, strings have remained mostly the same since JavaScript first came into being. ECMAScript 6 attempts to bring strings into modern times by adding some new functionality.

Locating substrings has long been the territory of the `indexOf()` method, which returns the zero-based position of a substring within a string or -1 if the substring doesn't exist within the string. This works fine most of the time but it's quite common for developers to use this method incorrectly, such as:

    var message = "Hello world!";

    // Error: "Hello" is at position 0, so this returns false
    if (message.indexOf("Hello")) {
        // this doesn't get executed
    }

In this code, the intent is to test if "Hello" exists within the variable `message`. The `indexOf()` method returns 0 in this case and is therefore considered false so the body of the `if` statement is never executed. What the code should do is ensure the value returned from `indexOf()` is greater than -1:

    var message = "Hello world!";

    // Correct
    if (message.indexOf("Hello") > -1) {
        // this executes
    }

Dealing with indices into strings  is a common source of error. In many cases, you just want to know if the string exists at a particular point within another string. ECMAScript 6 adds several new methods to make locating substrings easier.

First, there's a new method `startsWith()` that is available on all strings. As the name suggests, the method returns true if the string begins with the specified characters. For example:

    var message = "Hello world!";

    if (message.startsWith("Hello")) {
        // this executes
    }

The `startsWith()` method is really just a convenience over using `indexOf()`. In fact, you could write your own `startsWith()` method that acts the same way by using `indexOf()`:

    String.prototype.startsWith = function(value) {
        return this.indexOf(value) === 0;
    };

All `startsWith()` does is ensure that the given value is located at index 0 in the string.

To accompany `startsWith()` is `endsWith()`, which simply checks to see if the substring is located at the end of the string. For example:


    var message = "Hello world!";

    if (message.endsWith("world!")) {
        // this executes
    }

Once again, `endsWith()` is simply a wrapper around `indexOf()` that makes this particular use case a little bit easier. You can create your own implementation in this way:

    String.prototype.endsWith = function(value) {
        return this.indexOf(value) === this.length - value.length;
    };

The third method that helps you locate substrings is `contains()`, which returns true if the substring is located anywhere within the string:

    var message = "Hello world!";

    if (message.contains("wo")) {
        // this executes
    }

The `contains()` method is the same as ensuring the result of `indexOf()` is greater than -1. You can create your own version using this code:

    String.prototype.contains = function(value) {
        return this.indexOf(value) > -1;
    };

All three methods, `startsWith()`, `endsWith()`, and `contains()` allow you to more easily locate substrings. Of course, if you actually need to know the index of the substring, then you'll still need to use `indexOf()`, but for all other cases you're likely to find yourself using these new methods much more frequently.





## Object Literal Extensions

One of the most popular patterns in JavaScript is the object literal. It's the syntax upon which JSON is built and can be seen in nearly every JavaScript file on the Internet. The reason for the popularity is clear: Asus synced syntax for creating objects that otherwise would take several lines of code to accomplish. ECMAScript 6 recognized the popularity of the object literal and extends the syntax in several ways to make object literals more powerful and even more succinct.

### Property Initializer Shorthand

In ECMAScript 5 and earlier, object literals were simply collections of name-value pairs. That meant there could be some duplication when property values are being initialized. For example:

    function createPerson(name, age) {
        return {
            name: name,
            age: age
        };
    }

The `createPerson()` function creates an object whose property names are the same as the function parameter names. The result is what appears to be duplication of `name` and `age` even though each represents a different aspect of the process.

In ECMAScript 6, you can eliminate the duplication that exists around property names and local variables by using an property initializer shorthand. When the property name is going to be the same as the local variable name, you can simply include the name without a colon and value. For example, `createPerson()` can be rewritten as follows:

    function createPerson(name, age) {
        return {
            name,
            age
        };
    }

When a property in an object literal only has a name and no value, the JavaScript engine looks into the surrounding scope for a variable of the same name. If found, that value is assigns to the same name on the object literal. So in this example, the object literal property `name` is assigned the value of the local variable `name`.

The purpose of this extension is to make object literal initialization even more so since then it already was. Assigning a property with the same name as a local variable is a very common pattern in JavaScript and so this extension is a welcome addition.

### Method Initializer Shorthand

ECMAScript 6 also improves syntax for assigning methods to object literals. In ECMAScript 5 and earlier, you must specify a name and then the full function definition to add a method to an object. For example:

    var person = {
        name: "Nicholas",
        sayName: function() {
            console.log(this.name);
        }
    };

In ECMAScript 6, the syntax is made more sustained by eliminating the colon and the `function` keyword. you can then rewrite this example as follows:

    var person = {
        name: "Nicholas",
        sayName() {
            console.log(this.name);
        }
    };

This shorthand syntax creates a method on the `person` object just as the previous example did.

There is several important differences between the two ways of assigning methods to an object literal. First, methods assigned using the shorthand syntax cannot be used as a constructor. That means if you try to use the `new` operator with such a method, you'll get an error. For example:

    var person = {
        name: "Nicholas",
        sayName() {
            console.log(this.name);
        }
    };

    // TypeError: Cannot use shorthand method as a constructor
    var something = new person.sayName();

Here, an error is thrown when the `new` operator is used with `person.sayName()` because the method cannot be used as a constructor.

Methods defined using the shorthand syntax are also not enumerable, so they will not show up in a `for-in` loop nor in the array returned from `Object.keys()`. They will still show up in the array returned from `Object.getOwnPropertyNames()`.










