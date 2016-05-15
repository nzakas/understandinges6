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

#### Declaring Objects with Const

`const` 선언은 binding 의 수정을 막고, 그 자체 값의 수정을 막지는 않는다. 객체를 위한 `const` 선언이 해당 객체의 수정을 막는 것은 아님을 의미한다. 예를 들면:

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

여기서 `person` binding 은 하나의 프로퍼티를 가진 객체의 초기값으로 생성되었다. `person.name` 을 바꾸는 것은 에러 발생 없이 가능하다. 왜냐하면 이것은 `person` 이 포함하는 값을 바꾸고 `person` 에 바인딩된(bound) 값을 바꾸지는 않기 때문이다. 이 코드가 `person` 에 값을 할당하려고 시도할 때 (binding 을 바꾸려고 시도하므로), 에러가 발생할 것이다. `const` 가 객체와 함께 어떻게 작동하는 지에 대한 이 미묘함은 오해하기 쉽다. 이것만 기억하도록 하자: `const` 는 bind 된 값(bound value)의 수정이 아닌, binding 의 수정을 막는다.

### The Temporal Dead Zone

`let` 이나 `const` 로 선언된 변수는 선언 이후까지는 접근될 수 없다. 심지어 이 예제에서 typeof 명령어(operation)와 같은, 일반적으로 안전한 명령어를 사용할때도 reference error 를 발생시킨다:  

```js
if (condition) {
    console.log(typeof value);  // ReferenceError!
    let value = "blue";
}
```

여기서, `value` 변수는 `let` 을 사용해서 정의되고 초기화 되지만, 이전 라인에서 에러를 던지기 때문에 수행문(statement)은 결코 실행되지 않는다. 이 문제는 `value` 가 JavaScript 커뮤니티에서 *temporal dead zone* (TDZ) 이라고 불러온 곳에 존재한다는 것을 의미한다. TDZ 는 ECMAScript 스펙에서 명시적으로 이름지어지지 않았지만, 종종 왜 `let` 과 `const` 선언이 선언 이전에 접근할 수 없는지 설명하기 위해서 사용된다. 이 섹션은 TDZ 가 야기하는 선언 배치의 미묘함을 설명한다, 비록 모든 예제에 `let` 을 사용했지만, 같은 정보가 `const` 에도 적용되는 것에 주의하도록 한다.

JavaScript 엔진은 다음 블록을 찾아보고 변수 선언을 발견하면, 그 선언을 (`var` 의 경우) 해당 함수나 global scope 또는 (`let` 과 `const` 의 경우) TDZ 내 선언부의 최상단으로 호이스팅(hoist) 한다. TDZ 안에서는 모든 변수에 접근하기 위한 시도시에 runtime error 를 발생시킨다. 그 변수는 오직 TDZ 에서만 제거되므로, 변수 선언 후에는 사용하기 안전하다.

이것은 `let` 이나 `const` 로 선언한 변수를 정의하기 전에 사용하려고 하면 언제나 적용된다. 이전 예제가 보았듯이, 심지어 일반적으로 안전한 `typeof` 명령어에도 적용된다. 비록 원하는 결과가 나오진 않겠지만, 변수가 선언된 블록의 바깥에서 변수에 `typeof` 를 사용할 수 있다. 아래 코드를 참고해보자:


```js
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

The variable `value` isn't in the TDZ when the `typeof` operation executes because it occurs outside of the block in which `value` is declared. That means there is no `value` binding, and `typeof` simply returns `"undefined"`.

The TDZ is just one unique aspect of block bindings. Another unique aspect has to do with their use inside of loops.

## Block Binding in Loops - 원필현

아마도 개발자의 대부분이 원하는 변수의 블록 레벨 유효 영역은 `for` 루프 안이고, 한번 쓰고 버리는(throwaway) 카운터 변수는 오직 루프 내에서 사용하기로 정해져 있다. 예를 들면, 자바스크립트에서 이와 같은 코드를 볼 일이 드물지 않다.

```js
for (var i = 0; i < 10; i++) {
    process(items[i]);
}

// i is still accessible here
console.log(i);                     // 10
```

다른 언어에서 블록 레벨 범위은 기본이고, 예제는 의도대로 동작할 것이다, 그리고 오직 `for` 루프만 `i` 변수에 접근할 수 있어야 한다. 그러나, 자바스크립트에서는 `var` 선언이 호이스팅 되기 떄문에 `i` 변수는 loop가 완료된 후에도 여전히 접근 가능하다. 다음 코드에서와 같이, `let`을 대신 사용하여 의도된 동작을 할 것이다.

```js
for (let i = 0; i < 10; i++) {
    process(items[i]);
}

// i is not accessible here - throws an error
console.log(i);
```

이 예제에서, `i` 변수는 오로지 `for` 루프 내에서만 존재한다. 한번 루프가 완료되면, 변수는 더이상 다른곳에서 접근할 수 없다.

### Functions in Loops

`var`의 특징은 루프 안에서 함수를 만드는 것에 대해 오랫동안 문제를 가지고 있었다, 루프 변수는 루프의 범위 밖에서 접근할 수 있기 때문이다. 다음 코드를 살펴보자.

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // outputs the number "10" ten times
});
```

당신은 이 코드를 0에서 9까지 출력할 것으로 예상할 것이다. 그러나 이것은 숫자 10을 10번 출력한다.
왜냐하면 `i`는 루프의 각 반복 사이에서 공유되기 때문이다. 루프 내에서 생성된 함수의 의미는 모두 같은 변수를 참조한다는 것이다. 한번 루프가 완료되면 변수 `i`는 `10`의 값을 가지고, 그래서 `console.log(i)`가 호출될 때, 그때마다 그 값을 출력한다.

이 문제를 고치기 위해서, 개발자들은 반복해서 생성되기 원하는 변수의 새 복사본을 강제로 만드려고 루프 내부에서 즉시 실행 함수(IIFE)를 사용한다, 이 예제와 같이:

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

이 버전은 루프 내부에서 즉시 실행 함수(IIFE)를 사용한다.`i` 변수는 즉시실행함수(IIFE)에 전달된다, 자신의 복사본을 생성하고 `value`로 저장한다. 이 값은 함수에 의해 반복을 위해서 사용된다, 그래서 각 함수 호출은 0부터 9까지 루프에서 증가된 기대값을 반환한다. 다행스럽게도, ECMAScript6의 `let`과 `const`를 이용한 블록 바인딩은 이 루프를 단순화 할 수 있다.

### Let Declarations in Loops

`let` 선언은 이전 예제에서 IIFE 가 하는 것을 모방해서 효과적으로 반복문(loop)을 단순화한다.각 이터레이션에서, 반복문은 새 변수를 만들고 그것을 이전 이터레이션에서와 같은 이름의 변수 값에 초기화한다.아래코드처럼, 그것은 IIFE 를 생략하고도 기대하던 결과를 얻을 수 있다는 의미이다:

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

이 반복문은 `var` 와 IIFE 를 사용한 반복문과 똑같이, 그러나 더 깔끔하게 동작한다.`let` 선언은 반복할때마다 매번 새 변수 `i` 를 만들어서, 반복문 안에서 만들어진 각 함수는 각 반복(loop)시에 `i` 의 사본을 얻을 수 있다.각 `i` 의 사본은 그것이 만들어지는 반복(loop iteration)의 시작시에 할당된 값을 가진다. 아래에서 볼 수 있듯이 `for-in` 과 `for-of` 반복문에서도 마찬가지이다:

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

위 예제에서, `for-in` 반복문은 `for` 반복문과 같은 동작을 보여준다. 반복시마다, 새로운 `key` binding 이 생성되어서, 각 함수는 각 반복시의 변수 `key` 의 복사본을 가진다. 각 함수가 다른 값을 출력하는 결과를 볼 수 있다. 만약 `var` 이 `key` 선언에 사용되었다면, 모든 함수는 `"c"` 를 출력했을 것이다.

I> 반복문에서의 `let` 선언의 동작이 명세에 특별하게 정의된 동작이라는 것과 `let` 의 non-hoisting 특성에 반드시 연관된 것은 아니라는 것을 이해하는 것이 중요하다. 사실, 진행과정에서 나중에 추가된 것으로, `let` 의 초기 구현은 이러한 동작을 가지고 있지 않았다.

### Constant Declarations in Loops - 김병규
반복문에서의 상수 선언

The ECMAScript 6 specification doesn't explicitly disallow `const` declarations in loops; however, there are different behaviors based on the type of loop you're using. For a normal `for` loop, you can use `const` in the initializer, but the loop will throw a warning if you attempt to change the value. For example:

ECMAScript 6 정의에서는 반복분에서 `const` 선언을 사용하는것을 명시적으로 막진 않는다. 하지만 사용하는 반복문의 유형에 따라 다르게 작동한다. 일반적인 'for'반복문의 경우  `const`를 사용하여 초기화 할 수 있지만, 그 값을 바꾸려고할때 반복문은 warning을 뱉을것이다. 예를들어: 

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

이 코드에서, i 변수는 상수로 선언되어 있다. 반복분의 첫번째 반복에서, 'i'는 0이고 정상적으로 작동한다. 에러는 'i++'이 실행될 때 상수를 수정하려 하기 때문에 발생한다. 이처럼 반복문에서 'const' 사용하여 변수를 선언하는 것은 반복문에서 초기화한 변수를 수정하려 하지 않을때만 사용할 수 있다.

When used in a `for-in` or `for-of` loop, on the other hand, a `const` variable behaves the same as a `let` variable. So the following should not cause an error:

반면에 'for-in' 이나 'for-of' 반복문을 사용할때는, 'const'변수는 'let'으로 선언한 변수와 똑같이 돌아간다. 그래서 아래 코드는 에러를 뱉지 않는다.

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

이 코드의 함수들은 두번째 예제인 "Let Declarations in Loops"와 거의 흡사하다. 단 하나의 차이점은 반복분 안에서 `key`가 변할수 없다는 것이다. 'for-in' 와 'for-of' 반복분은 'const'와 잘 작동하는데 이는 반복문의 초기화가 기존에 바인딩 되었던 값을 변경하려하지않고, 각각의 반복에서 새롭게 생성되어 바인딩되기 때문이다. (이전 'for' 반본문 대신 'for-in'을 사용하면)

## Global Block Bindings

전역 범위에서 `let`과 `const`는 `var`와 다르게 동작한다. 전역 범위에서 `var`를 사용할 때, 새로운 전역 변수를 생성한다. 전역 변수는 전역 객체의 프로퍼티이다(브라우저에서는 `window` 객체). 이 것은 `var`를 사용해서 뜻하지 않게 존재하는 전역을 덮어쓸 수도 있다는 걸 의미한다. 다음 예제처럼 : 

```js
// in a browser
var RegExp = "Hello!";
console.log(window.RegExp);     // "Hello!"

var ncz = "Hi!";
console.log(window.ncz);        // "Hi!"
```

`window`에 `RegExp`이 정의되어 있더라도, 이것은 `var` 선언으로 인해 덮어써질 수 있어 안전하지 않다. 이 예제는 원본을 덮어쓰는 새로운 전역 변수 `RegExp`를 선언하는 예제이다. 비슷하게, `ncz`도 전역 변수로 선언되자마자 `window`의 프로퍼티로 정의된다. 자바스크립트는 늘 이렇게 동작한다.

만약 전역 범위에서 `let`이나 `const`를 대신 사용한다면, 새로운 바인딩이 전역 범위에 생성되지만 전역 객체의 프로퍼티로 추가되지는 않는다. 이것은 `let`이나 `const`를 사용하면 전역 변수를 덮어쓸 수 없다는 것이고, 단지 그림자처럼 사용할 수 있다. 여기 예제가 있다 :

```js
// in a browser
let RegExp = "Hello!";
console.log(RegExp);                    // "Hello!"
console.log(window.RegExp === RegExp);  // false

const ncz = "Hi!";
console.log(ncz);                       // "Hi!"
console.log("ncz" in window);           // false
```

`RegExp`에 대한 새로운 `let` 선언은 전역 `RegExp`의 그림자 바인딩을 생성한다. 이것은 `window.RegExp`와 `RegExp`는 같지 않으며, 전역 범위에 지장을 주지 않는다는 걸 의미한다. 또한, `ncz`에 대한 `const` 선언은 바인딩을 생성하지만, 전역 객체의 프로퍼티를 생성하지 않는다. 전역 객체의 프로퍼티를 생성하고 싶지 않을 때, `let`과 `const`를 이용한 이 기능은 전역 범위를 훨씬 안전하게 사용할 수 있게 만든다.

I> 만약 당신이 전역 객체를 사용할 수 있어야 하는 코드가 있다면, 여전히 전역 범위의 `var`를 사용할 수 있다. 이것이 프레임이나 창을 넘어 코드를 접근하기 원할 때 브라우저 안의 가장 일반적인 방법이다.

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
