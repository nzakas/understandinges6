# Block Bindings

전통적으로, 변수 선언 동작 방식은 자바스크립트 프로그래밍에서 까다로운 부분을 가지고 있다.
대부분의 C 기반은 언어들에서, 변수(또는 *bindings*)는 선언이 발생된 지점에서 생성된다.
그러나, 자바스크립트에서는 그렇지 않다.
변수가 실제로 생성되는 곳은 변수를 어떻게 선언했는지에 의존한다,
그리고 ECMAScript 6 은 유효 범위를 쉽게 제어할 수 있는 옵션을 제공한다.
이번 장에서는 전형적인 `var` 선언이 왜 혼란을 줄 수 있는지 실례를 들어가며 보여준다, ECMAScript 6 에서 block-level 바인딩을 소개한다,
그런 다음 그것들을 사용한 좋은 연습 방법을 제공한다.

## Var Declarations and Hoisting

`var` 를 사용한 변수 선언은 실제 선언이 발생한 곳에 상관없이 마치 함수의 맨 위(또는 만약 함수의 밖에 선언되어 있다면 global scope)에 있는 것처럼 처리되었다;
이것을 *hoisting*이라고 부른다.

호이스팅이 무엇인지에 대한 예제로, 다음 함수 정의를 고려했다.

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

만약 당신이 자바스크립트에 익숙하지 않다면, 당신은 변수 `value` 가 오직 if `condition` 이 true 일 경우에 만들어진다고 예상할 것이다.
실제로, 변수 `value` 는 상관없이 만들어진다.
이면에서, 자바스크립트 엔진은 `getValue` 함수를 아래처럼 변경한다.

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

초기화는 같은 지점에 남아있고, `value` 의 선언은 맨 위로 호이스팅 된다.
변수 `value` 는 실제로 `else` 절 안으로 부터 여전히 접근할 수 있다는 것을 의미한다.
접근된다면, 변수는 초기화되지 않았기 때문에, `undefined` 값을 가질 것이다.
종종 새로운 자바스크립트 개발자들은 호이스팅 선언에 익숙해 지는데 시간이 걸린다, 그리고 이 독특한 동작의 착오는 버그를 발생시킬 수 있다.
이러한 이유로, ECMAScript 6 은 더 강력하게 변수의 생명주기를 제어하기 위해 block level 유효 범위 옵션을 도입하였다.

## Block-Level Declarations

Block-level declarations 은 선언된 변수의 블록 범위 밖에서 접근할 수 없는 것이다.
블록 범위, 또는 어휘 범위라 불리며 다음 처럼 만들어 진다.

1. Inside of a function
2. Inside of a block (indicated by the `{` and `}` characters)

블록 범위는 많은 C 기반의 언어들에서 동작하는 방법이다, 그리고 ECMAScript 6 에 block-level declarations 의 도입은 자바스크립트에 유연성(그리고 통일성)을 가져오는 것이라고 생각한다.

### Let Declarations

`let` 선언 문법은 `var` 의 문법과 같다.
당신은 기본적으로 변수를 선언하기 위해 `var` 을 `let` 으로 대체할 수 있다, 그러나 변수의 범위는 오직 현재 코드 블록으로 제한된다.(또한, 좀 있다가 논의할 몇가지 다른 차이가 있다.)
`let` 선언은 선언된 블록의 맨 위로 호이스팅 되지 않기 때문에, 전체 블록에서 사용할 수 있도록 항상 `let` 을 블록의 처음에 위치하는 것이 나을 것이다.

여기에 예제가 있다.

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

이 버전의 `getValue` 함수는 다른 C 기반의 언어들에서 기대하는 방식과 훨씬 가깝게 동작한다.
변수 `value` 는 `var` 대신에 `let` 사용하여 선언되었기 때문에, 선언은 함수의 맨 위로 호이스팅되지 않는다, 그리고 변수 `value` 는 더이상 `if` 블록의 밖에 실행 흐름에서 접근할 수 없다.
만약 `condition` 의 값이 false 라면, 그때 `value` 는 결코 선언되거나 초기화되지 않는다.

### No Redeclaration

만약 하나의 식별자가 이미 어떤 범위에 선언되어 있다면, 그때 선언된 범위안에서 `let` 선언으로 식별자를 사용하는 것은 에러를 발생시킨다.(식별자 = 선언된 변수인 듯)

예를 들어

```js
var count = 30;

// Syntax error
let count = 40;
```

이 예제에서, `count` 는 `var` 와 `let` 으로 중복 선언되었다.
`let` 은 같은 범위 안에 이미 존재하는 식별자로 재정의 될 수 없기 때문에, `let` 선언은 에러를 발생할 것이다.
반면에, 다음 코드의 실제 예제처럼 만약 `let` 선언은 변수를 포함하는 범위에 같은 이름으로 새로운 변수를 만든다면 에러를 발생시키지 않는다.

```js
var count = 30;

// Does not throw an error
if (condition) {

    let count = 40;

    // more code
}
```

주변 블록에 `count` 를 만드는것 대신에, `if` 절 안에 새로운 변수 `count` 를 만들기 때문에 이 `let` 선언은 에러를 발생시키지 않는다.
`if` 블록 안에, 이 새로운 변수는 전역 `count` 를 가리고, 실행이 블록을 떠나기 전까지 전역 `count` 로 접근을 막는다.

### Constant Declarations - 서동현

You can also define variables in ECMAScript 6 with the `const` declaration syntax. Variables declared using `const` are considered *constants*, meaning their values cannot be changed once set. For this reason, every `const` variable must be initialized on declaration, as shown in this example:

```js
// Valid constant
const maxItems = 30;

// Syntax error: missing initialization
const name;
```

The `maxItems` variable is initialized, so its `const` declaration should work without a problem. The `name` variable, however, would cause a syntax error if you tried to run the program containing this code, because `name` is not initialized.


#### Constants vs Let Declarations

Constants, like `let` declarations, are block-level declarations. That means constants are no longer accessible once execution flows out of the block in which they were declared, and declarations are not hoisted, as demonstrated in this example:

```js
if (condition) {
    const maxItems = 5;

    // more code
}

// maxItems isn't accessible here
```

In this code, the constant `maxItems` is declared within an `if` statement. Once the statement finishes executing, `maxItems` is not accessible outside of that block.

In another similarity to `let`, a `const` declaration throws an error when made with an identifier for an already-defined variable in the same scope. It doesn't matter if that variable was declared using `var` (for global or function scope) or `let` (for block scope). For example, consider this code:

```js
var message = "Hello!";
let age = 25;

// Each of these would throw an error.
const message = "Goodbye!";
const age = 30;
```

The two `const` declarations would be valid alone, but given the previous `var` and `let` declarations in this case, neither will work as intended.

Despite those similarities, there is one big difference between `let` and `const` to remember. Attempting to assign a `const` to a previously defined constant will throw an error, in both strict and non-strict modes:

```js
const maxItems = 5;

maxItems = 6;      // throws error
```

Much like constants in other languages, the `maxItems` variable can't be assigned a new value later on. However, unlike constants in other language, the value a constant holds may be modified if it is an object.

#### Declaring Objects with Const - 김두형

A `const` declaration prevents modification of the binding and not of the value itself. That means `const` declarations for objects do not prevent modification of those objects. For example:

```js
const person = {
    name: "Nicholas"
};

// works
person.name = "Greg";

// throws an error
person = {
    name: "Greg"
};
```

Here, the binding `person` is created with an initial value of an object with one property. It's possible to change `person.name` without causing an error because this changes what `person` contains and doesn't change the value that `person` is bound to. When this code attempts to assign a value to `person` (thus attempting to change the binding), an error will be thrown. This subtlety in how `const` works with objects is easy to misunderstand. Just remember: `const` prevents modification of the binding, not modification of the bound value.

### The Temporal Dead Zone

A variable declared with either `let` or `const` cannot be accessed until after the declaration. Attempting to do so results in a reference error, even when using normally safe operations such as the `typeof` operation in this example:

```js
if (condition) {
    console.log(typeof value);  // ReferenceError!
    let value = "blue";
}
```

Here, the variable `value` is defined and initialized using `let`, but that statement is never executed because the previous line throws an error. The issue is that `value` exists in what the JavaScript community has dubbed the *temporal dead zone* (TDZ). The TDZ is never named explicitly in the ECMAScript specification, but the term is often used to describe why `let` and `const` declarations are not accessible before their declaration. This section covers some subtleties of declaration placement that the TDZ causes, and although the examples shown all use `let`, note that the same information applies to `const`.

When a JavaScript engine looks through an upcoming block and finds a variable declaration, it either hoists the declaration to the top of the function or global scope (for `var`) or places the declaration in the TDZ (for `let` and `const`). Any attempt to access a variable in the TDZ results in a runtime error. That variable is only removed from the TDZ, and therefore safe to use, once execution flows to the variable declaration.

This is true anytime you attempt to use a variable declared with `let` or `const`  before it's been defined. As the previous example demonstrated, this even applies to the normally safe `typeof` operator. You can, however, use `typeof` on a variable outside of the block where that variable is declared, though it may not give the results you're after. Consider this code:

```js
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

The variable `value` isn't in the TDZ when the `typeof` operation executes because it occurs outside of the block in which `value` is declared. That means there is no `value` binding, and `typeof` simply returns `"undefined"`.

The TDZ is just one unique aspect of block bindings. Another unique aspect has to do with their use inside of loops.

## Block Binding in Loops - 원필현

Perhaps one area where developers most want block level scoping of variables is within `for` loops, where the throwaway counter variable is meant to be used only inside the loop. For instance, it's not uncommon to see code like this in JavaScript:

```js
for (var i = 0; i < 10; i++) {
    process(items[i]);
}

// i is still accessible here
console.log(i);                     // 10
```

In other languages, where block level scoping is the default, this example should work as intended, and only the `for` loop should have access to the `i` variable. In JavaScript, however, the variable `i` is still accessible after the loop is completed because the `var` declaration gets hoisted. Using `let` instead, as in the following code, should give the intended behavior:

```js
for (let i = 0; i < 10; i++) {
    process(items[i]);
}

// i is not accessible here - throws an error
console.log(i);
```

In this example, the variable `i` only exists within the `for` loop. Once the loop is complete, the variable is no longer accessible elsewhere.

### Functions in Loops

The characteristics of `var` have long made creating functions inside of loops problematic, because the loop variables are accessible from outside the scope of the loop. Consider the following code:

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // outputs the number "10" ten times
});
```

You might ordinarily expect this code to print the numbers 0 to 9, but it outputs the number 10 ten times in a row. That's because `i` is shared across each iteration of the loop, meaning the functions created inside the loop all hold a reference to the same variable. The variable `i` has a value of `10` once the loop completes, and so when `console.log(i)` is called, that value prints each time.

To fix this problem, developers use immediately-invoked function expressions (IIFEs) inside of loops to force a new copy of the variable they want to iterate over to be created, as in this example:

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value);
        }
    }(i)));
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
});
```

This version uses an IIFE inside of the loop. The `i` variable is passed to the IIFE, which creates its own copy and stores it as `value`. This is the value used by the function for that iteration, so calling each function returns the expected value as the loop counts up from 0 to 9. Fortunately, block-level binding with `let` and `const` in ECMAScript 6 can simplify this loop for you.

### Let Declarations in Loops - 신일용

A `let` declaration simplifies loops by effectively mimicking what the IIFE does in the previous example. On each iteration, the loop creates a new variable and initializes it to the value of the variable with the same name from the previous iteration. That means you can omit the IIFE altogether and get the results you expect, like this:

```js
var funcs = [];

for (let i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
})
```

This loop works exactly like the loop that used `var` and an IIFE but is, arguably, cleaner. The `let` declaration creates a new variable `i` each time through the loop, so each function created inside the loop gets its own copy of `i`. Each copy of `i` has the value it was assigned at the beginning of the loop iteration in which it was created. The same is true for `for-in` and `for-of` loops, as shown here:

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

for (let key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

In this example, the `for-in` loop shows the same behavior as the `for` loop. Each time through the loop, a new `key` binding is created, and so each function has its own copy of the `key` variable. The result is that each function outputs a different value. If `var` were used to declare `key`, all functions would output `"c"`.

I> It's important to understand that the behavior of `let` declarations in loops is a specially-defined behavior in the specification and is not necessarily related to the non-hoisting characteristics of `let`. In fact, early implementations of `let` did not have this behavior, as it was added later on in the process.

### Constant Declarations in Loops - 김병규

The ECMAScript 6 specification doesn't explicitly disallow `const` declarations in loops; however, there are different behaviors based on the type of loop you're using. For a normal `for` loop, you can use `const` in the initializer, but the loop will throw a warning if you attempt to change the value. For example:

```js
var funcs = [];

// throws an error after one iteration
for (const i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}
```

In this code, the `i` variable is declared as a constant. The first iteration of the loop, where `i` is 0, executes successfully. An error is thrown when `i++` executes because it's attempting to modify a constant. As such, you can only use `const` to declare a variable in the loop initializer if you are not modifying that variable.

When used in a `for-in` or `for-of` loop, on the other hand, a `const` variable behaves the same as a `let` variable. So the following should not cause an error:

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

// doesn't cause an error
for (const key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

This code functions almost exactly the same as the second example in the "Let Declarations in Loops" section. The only difference is that the value of `key` cannot be changed inside the loop. The `for-in` and `for-of` loops work with `const` because the loop initializer creates a new binding on each iteration through the loop rather than attempting to modify the value of an existing binding (as was the case with the previous example using `for` instead of `for-in`).

## Global Block Bindings - 진유정

Another way in which `let` and `const` are different from `var` is in their global scope behavior. When `var` is used in the global scope, it creates a new global variable, which is a property on the global object (`window` in browsers). That means you can accidentally overwrite an existing global using `var`, such as:

```js
// in a browser
var RegExp = "Hello!";
console.log(window.RegExp);     // "Hello!"

var ncz = "Hi!";
console.log(window.ncz);        // "Hi!"
```

Even though the `RegExp` global is defined on `window`, it is not safe from being overwritten by a `var` declaration. This example declares a new global variable `RegExp` that overwrites the original. Similarly, `ncz` is defined as a global variable and immediately defined as a property on `window`. This is the way JavaScript has always worked.

If you instead use `let` or `const` in the global scope, a new binding is created in the global scope but no property is added to the global object. That also means you cannot overwrite a global variable using `let` or `const`, you can only shadow it. Here's an example:

```js
// in a browser
let RegExp = "Hello!";
console.log(RegExp);                    // "Hello!"
console.log(window.RegExp === RegExp);  // false

const ncz = "Hi!";
console.log(ncz);                       // "Hi!"
console.log("ncz" in window);           // false
```

Here, a new `let` declaration for `RegExp` creates a binding that shadows the global `RegExp`. That means `window.RegExp` and `RegExp` are not the same, so there is no disruption to the global scope. Also, the `const` declaration for `ncz` creates a binding but does not create a property on the global object. This capability makes `let` and `const` a lot safer to use in the global scope when you don't want to create properties on the global object.

I> You may still want to use `var` in the global scope if you have a code that should be available from the global object. This is most common in a browser when you want to access code across frames or windows.

## Emerging Best Practices for Block Bindings

ECMAScript 6 으로 개발하는 경우, 변수 선언을 위해 `var` 를 사용하는 대신 `let` 를 사용해야 하는 것이 기본적으로 널리 퍼진 통념이다.
많은 자바스크립트 개발자들을 위해, `let` 은 자바스크립트 개발자들이 `var` 가 동작해야 한다고 생각하는 방법으로 정확하게 동작한다,
그래서 직접적인 교체는 논리적으로 이해가 된다.
이 경우, 수정되는 것을 막을 필요가 있는 변수를 위해 'const' 를 사용할 것이다.

그러나, 많은 개발자들은 ECMAScript 6 로 이주한것 처럼, 선택가능한 접근법은 인기를 얻었다: 기본적으로 `const` 를 사용하고 변수의 값이 변경될 필요가 있을 때만 `let` 을 사용한다.
그 이유는 예기치 않은 변수의 변경은 버그를 가진 소스이기 때문에 대부분 변수들은 초기화 한 후에 변경되지 않아야 한다는 것이다.
이러한 아이디어는 ECMASCript 6 를 채택함으로 충분하고 당신의 코드를 살펴볼 가치가 있다.

## Summary

`let` 과 `const` 블록 바인딩을 자바스크립트 어휘를 살펴보는 것으로 소개하였다.
이 선언들은 호이스팅되지 않고 오직 선언된 곳의 블록 안에 존재한다.
이제 변수가 꼭 필요한 곳에 선언될 수 있으므로, 이것은 좀 더 다른 언어들처럼 동작하고 뜻하지 않은 에러 발생을 더 적게 한다.
부작용으로, 예를 들어 `typeof` 와 같은 안전한 명령어들도, 선언되기 전에 변수에 접근할 수 없다.
바인딩은 temporal dead zone(TDZ) 에 있기 때문에, 선언되기 전 블록 바인딩에 접근하려고 시도하는 것은 에러를 야기한다.


많은 경우들에서, `let` 과 `const` 는 `var` 와 비슷한 방식으로 동작한다. 그러나, for 루프에서는 사실이 아니다.
`for-in` 과 `for-of` 루프에서 `let` 과 `const` 둘 다, 루프의 각 반복마다 새로운 바인딩을 만든다.
그것은 루프의 마지막 반복후에 있는것 보다는(`var` 의 동작), 루프 안에서 만들어진 함수는 현재 반복 동안 루프에 바인딩된 변수에 접근할 수 있다는 것을 의미한다.
`for` 루프 안에 `let` 선언도 마찬가지다, `for` 루프 안에서 `const` 선언을 사용하려는 경우 에러를 야기할 수 있다.

현재 block 바인딩의 좋은 연습은 기본적으로 `const` 를 사용하고 오직 변수의 값이 변할 필요가 있을 때만 `let` 을 사용하는 것이다.
이것은 확실한 타입 에러를 예방할 수 있는 코드에 불변성의 기본적인 수준을 보장한다.
