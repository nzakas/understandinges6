# Template Strings

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

JavaScript's strings have been fairly limited when compared to those in other languages. Template strings add new syntax to allow the creation of domain-specific languages (DSLs) for working with content in a way that is safer than the solutions we have today. The description on the template string strawman was as follows:

> This scheme extends ECMAScript syntax with syntactic sugar to allow libraries to provide DSLs that easily produce, query, and manipulate content from other languages that are immune or resistant to injection attacks such as XSS, SQL Injection, etc.

In reality, though, template strings are ECMAScript 6's answer to several ongoing problems in JavaScript:

* **Multiline strings** - JavaScript has never had a formal concept of multiline strings.
* **Basic string formatting** - The ability to substitute parts of the string for values contained in variables.
* **HTML escaping** - The ability to transform a string such that it is safe to insert into HTML.
* **Localization of strings** - The ability to easily swap out a string from one language into a string from another language.

Rather than trying to add more functionality to JavaScript's already-existing strings, template strings represent an entirely new approach to solving these problems.

## Basic Syntax

At their simplest, template strings act like regular strings that are delimited by backticks (``) instead of double or single quotes. For example:

```js
let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12
```

This code demonstrates that the variable `message` contains a normal JavaScript string. The template string syntax only is used to create the string value, which is then assigned to `message`.

If you want to use a backtick in your string, then you need only escape it by using a backslash (`\`):

```js
let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14
```

There's no need to escape either double or single quotes inside of template strings.

## Multiline Strings

Ever since the first version of JavaScript, developers have longed for a way to create multiline strings in JavaScript. When using double or single quotes, strings must be completely contained on a single line. JavaScript has long had a syntax bug that would allow multiline strings by using a backslash (`\`) before a newline, such as:

```js
let message = "Multiline \
string";
```

Despite this working in all major JavaScripte engines, the behavior was defined as a bug and many recommended avoiding its usage.

Other attempts to create multiline strings usually relied on arrays or string concatenation, such as:

```js
let message = [
    "Multiline ",
    "string"
].join("");

let message = "Multiline " +
    "string";
```

All of the ways developers worked around JavaScript's lack of multiline strings left something to be desired.

Template strings make multiline strings easy because there is no special syntax. Just include a newline where you want and it shows up in the result. For example:

```js
let message = `Multiline
string`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

All whitespace inside of the backticks is considered to be part of the string, so be careful with indentation. For example:

```js
let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 16
```

In this code, all of the whitespace before the second line of the template string is considered to be a part of the string itself. If making the text line up with proper indentation is important to you, then you consider leaving nothing on the first line of a multiline template string and then indenting after that, such as this:

```js
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

This code begins the template string on the first line but doesn't have any text until the second. The HTML tags are indented to look correct and then the `trim()` method is called to remove the initial (empty) line.

## Substitutions

To this point, template strings may look like a fancier way of defining normal JavaScript strings. The real difference is with template string substitutions. Substitutions allow you to embed any valid JavaScript expression inside of a template string and have the result be output as part of the string.

Substitutions are delimited by an opening `${` and a closing `}`, within which you can use any JavaScript expression. At its simplest, substitutions let you embed local variables directly into the result string, like this:

```js
let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas."
```

The substitution `${name}` accessed the local variable `name` to insert it into the string. The `message` variable then holds the result of the substitution immediately.

I> Template strings can access any variable that is accessible in the scope in which it is defined. Attempting to use an undeclared variable in a template string results in an error being thrown in both strict and non-strict modes.

Since all substitutions are JavaScript expressions, it's possible to substitute more than just simple variable names. You can easily embed calculations, function calls, and more. For example:

```js
let count = 10,
    price = 0.25,
    message = `The total for ${count} items is $${(count * price).toFixed(2)}.`;

console.log(message);       // "The total for 10 items is $2.50."
```

This code performs a calculation as part of the template string. The variables `count` and `price` are multiplied together to get a result, and then formatted to two decimal places using `.toFixed()`. The dollar sign before the second substitution is output as-is because it's not followed by an opening curly brace.

## Tagged Templates

TODO

## Summary

TODO
