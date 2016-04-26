# ECMAScript 7 (2016)

The development of ECMAScript 6 took about four years, and after that, TC39 decided that this process was not sustainable. As a result, they moved to a yearly release cycle that would ensure new language features could make it into development sooner. The more frequent releases meant that each new edition would have a much smaller number of features than ECMAScript 6. To signify this change, future versions of the specification no longer prominently feature the edition number, and instead refer to the year in which the specification was published. As a result, ECMAScript 6 is also known as ECMAScript 2015, and ECMAScript 7 is formally known as ECMAScript 2016. Going forward, it's expected that the year-based name will be used for all future ECMAScript editions.

ECMAScript 2016 was finalized in March 2016 and contained only two additions to the language, which are covered in this chapter.


## Exponentiation Operator

The only change to JavaScript syntax introduced in ECMAScript 2016 is the exponentiation operator, which is a mathematical operation that applies an exponent to a base. JavaScript already had the `Math.pow()` method to perform exponentiation, however, JavaScript was one of the only languages that required a method rather than a formal operator (which some argue is easier to read and reason about).

The exponentiation operator is two asterisks (`**`) where the left operand is the base and the right operand is the exponent. For example:

```js
let result = 5 ** 2;

console.log(result);                        // 25
console.log(result === Math.pow(5, 2));     // true
```

This example calculates 5^2^, which is equal to 25. You can still use `Math.pow()` to achieve the same result.

### Order of Operations

The exponentiation operator has the highest precedence of all operators in JavaScript. That means it is applied first to any compound operation, for example:

```js
let result = 2 * 5 ** 2;
console.log(result);        // 50
```

In this example, the calculation of 5^2^ happens first and then the result is multiplied by 2 for a result of 50.

### Operand Restriction

The exponentiation operator does have a somewhat unusual restriction that isn't present for other operators.The left side of an exponentiation operation cannot be a unary expression other than `++` or `--`. For example, this is invalid syntax:

```js
// syntax error
let result = -5 ** 2;
```

The `-5` in this example is a syntax error because the order of operations is ambiguous. Does the `-` apply just to `5` or the result of `5 ** 2`? Disallowing unary expressions on the left side eliminates the ambiguity. In order to clearly specify intent, you need to include parentheses either around `-5` or around `5 ** 2`, as in this example:

```js
// ok
let result1 = -(5 ** 2);    // equal to -25

// also ok
let result2 = (-5) ** 2;    // equal to 25
```

The only unary operations allowed on the left side of an exponentiation operation are `++` and `--`. Both of these operators have clearly-defined behavior on their operands, where a prefix `++` or `--` changes the operand before any other operations take place and the postfix versions don't apply any changes until after the entire expression has been evaluated. In both cases, they are safe to use on the left side:

```js
let num1 = 2,
    num2 = 2;

console.log(++num1 ** 2);       // 9
console.log(num1);              // 3

console.log(num2-- ** 2);       // 4
console.log(num2);              // 1
```

In this example, `num1` is incremented before the exponentiation operator is applied, so `num1` becomes 3 and the result of the operation is 9. For `num2`, the value remains 2 for the exponentiation operation and then is decremented to 1.

## Array.prototype.includes

TODO
