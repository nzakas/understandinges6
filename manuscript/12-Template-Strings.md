# Template Strings

JavaScript's strings have been fairly limited when compared to those in other languages. Template strings add new syntax to allow the creation of domain-specific languages (DSLs) for working with content in a way that is safer than the solutions we have today. The description on the template string strawman was as follows:

> This scheme extends ECMAScript syntax with syntactic sugar to allow libraries to provide DSLs that easily produce, query, and manipulate content from other languages that are immune or resistant to injection attacks such as XSS, SQL Injection, etc.

In reality, though, template strings are ECMAScript 6's answer to several ongoing problems in JavaScript:

* **Multiline strings** - JavaScript has never had a formal concept of multiline strings.
* **Basic string formatting** - The ability to substitute parts of the string for values contained in variables.
* **HTML escaping** - The ability to transform a string such that it is safe to insert into HTML.

Rather than trying to add more functionality to JavaScript's already-existing strings, template strings represent an entirely new approach to solving these problems.

## Basic Syntax

At their simplest, template strings act like regular strings that are delimited by backticks (`` ` ``) instead of double or single quotes. For example:

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

console.log(message);       // "Multiline string"
```

Note that the string has no newlines present when output, that's because the backslash is treated as a continuation rather than a newline. In order to have a newline at that point, you would need to manually include it, such as:

```js
let message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string"
```

Despite this working in all major JavaScript engines, the behavior was defined as a bug and many recommended avoiding its usage.

Other attempts to create multiline strings usually relied on arrays or string concatenation, such as:

```js
let message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
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
console.log(message.length);    // 31
```

In this code, all of the whitespace before the second line of the template string is considered to be a part of the string itself. If making the text line up with proper indentation is important to you, then you consider leaving nothing on the first line of a multiline template string and then indenting after that, such as this:

```js
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

This code begins the template string on the first line but doesn't have any text until the second. The HTML tags are indented to look correct and then the `trim()` method is called to remove the initial (empty) line.

A> If you prefer, you can also use `\n` in a template string to indicate where a newline should be inserted:
A> {:lang="js"}
A> ~~~~~~~~
A>
A> let message = `Multiline\nstring`;
A>
A> console.log(message);           // "Multiline
A>                                 //  string"
A> console.log(message.length);    // 16
A> ~~~~~~~~

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
    message = `${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

This code performs a calculation as part of the template string. The variables `count` and `price` are multiplied together to get a result, and then formatted to two decimal places using `.toFixed()`. The dollar sign before the second substitution is output as-is because it's not followed by an opening curly brace.

## Tagged Templates

To this point, you've seen how template strings can be used for multiline strings and to insert values into strings without using concatenation. The real power of template strings comes from tagged templates. A *template tag* performs a transformation on the template string and returns the final string value. This tag is specified at the start of the template, just before the first `` ` `` character, such as:

```js
let message = tag`Hello world`;
```

In this example, `tag` is the template tag to apply to `` `Hello world` ``.

### Defining Tags

A tag is simply a function that is called with the processed template string data. The function receives data about the template string as individual pieces that the tag must then combined to create the finished value. The first argument is an array containing the literal strings as they are interpreted by JavaScript. Each subsequent argument is the interpreted value of each substitution. Tag functions are typically defined using rest arguments to make dealing with the data easier:

```js
function tag(literals, ...substitutions) {
    // return a string
}
```

To better understand what is passed to tags, consider the following:

```js
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
```

If you had a function called `passthru()`, that function would receive three arguments:

1. `literals`, containing:
    * `""` - the empty string before the first substitution
    * `" items cost $"` - the string after the first substitution and before the second
    * `"."` - the string after the second substitution
1. `10` - the interpreted value for `count` (this becomes `substitutions[0]`)
1. `"2.50"` - the interpreted value for `(count * price).toFixed(2)` (this becomes `substitutions[1]`)

Note that the first item in `literals` is an empty string. This is to ensure that `literals[0]` is always the start of the string, just like `literals[literals.length - 1]` is always the end of the string. There is always one fewer substitution than literal, which is to say that `substitutions.length === literals.length - 1` all the time.

Using this pattern, the `literals` and `substitutions` arrays can be interweaved to create the result. The first item in `literals` comes first, then the first item in `substitutions`, and so on, until the string has been completed. So to mimic the default behavior of template, you need only define a function that performs this operation:

```js
function passthru(literals, ...substitutions) {
    let result = "";

    // run the loop only for the substitution count
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }

    // add the last literal
    result += literals[literals.length - 1];

    return result;
}

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

This example defines a `passthru` tag that performs the same transformation as the default template string behavior. The only trick is to use `substitutions.length` for the loop rather than `literals.length` to avoid accidentally going past the end of `substitutions`. This works because the relationship between `literals` and `substitutions` is well-defined.

I> The values contained in `substitutions` are not necessarily strings. If an expression is evaluated to be a number, as in the previous example, then the numeric value is passed in. It's part of the tag's job to determine how such values should be output in the result.

### Using Raw Values

Template tags also have access to raw string information, which primarily means access to character escapes before they are transformed into their character equivalents. The simplest way to work with raw string values is to the built-in `String.raw()` tag. For example:

```js
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring"
```

In this code, the `\n` in `message1` is interpreted as a newline while the `\n` in `message2` is returned in its raw form of `"\\n"` (two characters, the slash and `n`). Retrieving the raw string information in this way allows for more complex processing (when necessary).

The raw string information is also passed into template tags. The first argument in a tag function is an array with an extra property called `raw`. The `raw` property is an array containing the raw equivalent of each literal value. So the value in `literals[0]` always has an equivalent `literals.raw[0]` that contains the raw string information. Knowing that, it's possible to mimic `String.raw()` using the following:

```js
function raw(literals, ...substitutions) {
    let result = "";

    // run the loop only for the substitution count
    for (let i = 0; i < substitutions.length; i++) {
        result += literals.raw[i];      // use raw values instead
        result += substitutions[i];
    }

    // add the last literal
    result += literals.raw[literals.length - 1];

    return result;
}

let message = raw`Multiline\nstring`;

console.log(message);           // "Multiline\\nstring"
console.log(message.length);    // 17
```

This example uses `literals.raw` instead of `literals` to output the string result. That means any character escapes, including Unicode code point escapes, will be returned in their raw form.

## Summary

Template strings are an important addition to ECMAScript 6 that allows the creating of domain-specific languages (DSLs) to make creating strings easier. The ability to embed variables directly into template strings means that developers have a safer tool than string concatenation for composing long strings with variables.

Built-in support for multiline strings also makes template strings a useful upgrade over normal JavaScript strings, which have never had this ability. Despite allowing newlines directly inside the template string, you can still use `\n` and other character escape sequences.

Template tags are the most important part of this feature for creating DSLs. Tags are functions that receive the pieces of the template string as arguments. You can then use that data to return an appropriate string value. The data provided includes literals, their raw equivalents, and any substitution values. These pieces of information can then be used to determine the correct output for the tag.

ECMAScript 6 has only one built-in tag, which is `String.raw()`. This tag simply returns the template string in its raw form, with character escape sequences in their original form rather than transforming them into their character equivalents.
