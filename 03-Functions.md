# 関数

関数はプログラミング言語の重要な部分であり、ECMAScript6より前では、言語が生成されて以来、JavaScript関数はほとんど変更されていませんでした。これは、間違いを簡単にし、非常に基本的な振る舞いを達成するために多くのコードを必要とする問題や微妙な動作のバックログを残しました。

ECMAScript6の機能は、JavaScript開発者から何年もの苦情や要望を考慮に入れて、大きな飛躍を遂げています。その結果、ECMAScript5の機能の上で、JavaScriptのプログラミングをエラーを起こしにくく、より強力にしています。

## デフォルトのパラメータ値を持つ関数

JavaScriptの関数は、関数定義で宣言されたパラメータの数に関係なく、任意の数のパラメータを渡すことができる点で独特です。これにより、異なる数のパラメータを扱うことができる関数を定義することができます。パラメータが指定されていない場合には、デフォルト値を入力するだけです。このセクションでは、デフォルトパラメータがECMAScript6とその前の両方でどのように機能するか、`arguments`オブジェクトに関する重要な情報、式をパラメータとして使用する方法、および別のTDZについて説明します。

### ECMAScript5のデフォルトパラメータ値のシミュレーション

ECMAScript5以前では、次のパターンを使用して、デフォルトのパラメータ値を持つ関数を生成していました。

```js
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // the rest of the function

}
```

この例では、`timeout`と`callback`の両方が実際にはオプションです。なぜなら、パラメータが与えられていなければデフォルト値が与えられているからです。 論理OR演算子(`||`)は、最初のオペランドが偽であるとき、常に第2オペランドを返します。 明示的に提供されていない名前付き関数パラメータは`undefined`に設定されているので、論理OR演算子は欠けているパラメータのデフォルト値を提供するために頻繁に使用されます。 しかし、このアプローチには、 'timeout'の有効な値が実際には '0'であるという点で欠陥がありますが、これは '0'が偽であるため '2000'に置き換えられます。

その場合、より安全な選択肢は、この例のように`typeof`を使って引数の型をチェックすることです：

```js
function makeRequest(url, timeout, callback) {

    timeout = (typeof timeout !== "undefined") ? timeout : 2000;
    callback = (typeof callback !== "undefined") ? callback : function() {};

    // the rest of the function

}
```

このアプローチはより安全ですが、非常に基本的な操作にはさらに多くの余分なコードが必要です。 一般的なJavaScriptライブラリは、共通のパターンを表すため、類似のパターンで埋められています。

### ECMAScript6のデフォルトのパラメータ値

ECMAScript6では、パラメータが正式に渡されないときに使用される初期化を提供することにより、パラメータのデフォルト値を提供することが容易になります。 例えば：

```js
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // the rest of the function

}
```

この関数は、最初のパラメータが常に渡されることを期待しています。 他の2つのパラメータにはデフォルト値があり、欠損値をチェックするためのコードを追加する必要がないため、関数の本体が大幅に小さくなります。

`makeRequest()`が3つのパラメータすべてで呼び出されると、デフォルトは使用されません。 例えば：

```js
// uses default timeout and callback
makeRequest("/foo");

// uses default callback
makeRequest("/foo", 500);

// doesn't use defaults
makeRequest("/foo", 500, function(body) {
    doSomething(body);
});
```

ECMAScript6は`url`が必要であるとみなします。なぜなら、``/foo"`がmakeRequest()への3回の呼び出しのすべてに渡されるからです。 デフォルト値を持つ2つのパラメータはオプションと見なされます。

関数宣言でデフォルト値を持たない引数の前に出現するものを含め、任意の引数のデフォルト値を指定することは可能です。 たとえば、これは問題ありません。

```js
function makeRequest(url, timeout = 2000, callback) {

    // the rest of the function

}
```

この場合、`timeout`のデフォルト値は、2番目の引数が渡されていない場合、または2番目の引数が`undefined`として明示的に渡されている場合にのみ使用されます。

```js
// uses default timeout
makeRequest("/foo", undefined, function(body) {
    doSomething(body);
});

// uses default timeout
makeRequest("/foo");

// doesn't use default timeout
makeRequest("/foo", null, function(body) {
    doSomething(body);
});
```

デフォルトのパラメータ値の場合、`null`の値が有効であるとみなされます。つまり、`makeRequest()`の3回目の呼び出しでは、`timeout`のデフォルト値は使用されません。

### デフォルトのパラメータ値がargumentsオブジェクトに与える影響

デフォルトのパラメータ値が存在する場合、`arguments`オブジェクトの動作は異なります。ECMAScript5非厳密モードでは、`arguments`オブジェクトは、関数の名前付きパラメータの変更を反映します。これはどのように動作するかを示すコードです：

```js
function mixArgs(first, second) {
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d";
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b");
```

出力値は下記のようになります。

```
true
true
true
true
```

`arguments`オブジェクトは、指定されたパラメータの変更を反映するために、非厳密モードで常に更新されます。 したがって、`first`と`second`に新しい値が代入されると`arguments 'と`arguments[1]`はそれに応じて更新され、`===`のすべての比較が`true`に解決されます。

しかし、ECMAScript5の厳密なモードは、`arguments`オブジェクトのこの混乱する側面を排除します。 strictモードでは、`arguments`オブジェクトは指定されたパラメータへの変更を反映しません。 以下は`mixArgs()`関数ですが、strictモードです：

```js
function mixArgs(first, second) {
    "use strict";

    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b");
```

`mixArgs()`関数の出力値は下記のようになります。

```
true
true
false
false
```

今回は、`first`と`second`を変更しても`arguments`には影響しないので、通常は期待通りに出力が動作します。

ただし、ECMAScript6のデフォルトのパラメータ値を使用する関数の`arguments`オブジェクトは、関数が厳密なモードで明示的に実行されているかどうかにかかわらず、常にECMAScript5 strictモードと同じように動作します。 デフォルトのパラメータ値が存在すると、`arguments`オブジェクトは、指定されたパラメータから切り離されたままになります。 これは、`arguments`オブジェクトがどのように使われるかという理由で、微妙だが重要な詳細です。 次の点を考慮してください。

```js
// not in strict mode
function mixArgs(first, second = "b") {
    console.log(arguments.length);
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a");
```

出力値は下記のようになります。

```
1
true
false
false
false
```

この例では、`arguments.length`は1で、`mixArgs()`に1つの引数しか渡されていないためです。 これは、`arguments [1]`が`undefined`であることを意味します。これは、関数に1つの引数しか渡されないときに期待される動作です。 つまり、`first`は`arguments [0]`と同じです。`first`と`second`を変更しても`arguments`には何の効果もありません。 この動作は、厳密でないモードと厳密なモードの両方で発生するため、常に`arguments`を利用して初期呼び出し状態を反映させることができます。

###デフォルトのパラメータ式

おそらく、デフォルトのパラメータ値の最も興味深い機能は、デフォルト値がプリミティブ値である必要はないということです。 たとえば、次のようにデフォルトのパラメータ値を取得する関数を実行できます。

```js
function getValue() {
    return 5;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
```

ここで、最後の引数が与えられていない場合は、関数`getValue()`が呼び出されて正しいデフォルト値が取得されます。`getValue()`は、関数宣言が最初にパースされたときではなく、`add()`が第2引数なしで呼び出されたときにのみ呼び出されることに注意してください。 つまり、`getValue()`が異なって記述された場合、潜在的に異なる値を返す可能性があります。 例えば：

```js
let value = 5;

function getValue() {
    return value++;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
console.log(add(1));        // 7
```

この例では、`value`は5で始まり、`getValue()`が呼び出されるたびにインクリメントします。`add(1)`への最初の呼び出しは6を返し、`add(1)`への2回目の呼び出しは`value`がインクリメントされて7を返します。`second`のデフォルト値は、関数の呼び出し時にのみ評価されるため、その値への変更はいつでも行うことができます。

W>関数呼び出しをデフォルトのパラメータ値として使用する場合は注意してください。 最後の例で`second = getValue`のようなかっこを忘れた場合は、関数呼び出しの結果ではなく関数への参照を渡しています。

この動作により、もう1つ興味深い機能が導入されます。 前のパラメータは、後のパラメータのデフォルトとして使用できます。 ここに例があります：

```js
function add(first, second = first) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

このコードでは、パラメータ`second`にはデフォルト値`first`が与えられています。つまり、1つの引数だけを渡すと、両方の引数が同じ値になります。 したがって、`add(1)`は`add(1)`が2を返すのと同じように2を返します。これをさらに進めると、`first`を関数に渡して`second`の値を次のように得ることができます：

```js
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

この例では`second`を`getValue(first)`によって返された値と等しく設定していますので、`add(1,1)`はまだ2を返しますが、`add(1)`は7(1 + 6)を返します。

既定のパラメータ割り当てからパラメータを参照する機能は、以前の引数に対してのみ機能するため、以前の引数は後の引数にアクセスできません。 例えば：

```js
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // throws error
```

`second 'は`first`の後に定義されているので、`add(undefined, 1)`の呼び出しはエラーを投げるので、デフォルト値として利用できません。 そのようなことが起こる理由を理解するには、一時的な不感地帯を再訪することが重要です。

###デフォルトのパラメータ値Temporal Dead Zone

第1章では、letとconstに関連する一時的な不感帯(TDZ)について紹介しました。デフォルトのパラメータ値には、パラメータにアクセスできないTDZもあります。`let`宣言と同様に、各パラメータは、エラーを投げずに初期化の前に参照することができない新しい識別子バインディングを生成します。 パラメータの初期化は、関数の呼び出し時にパラメータの値を渡すか、デフォルトのパラメータ値を使用して行われます。

デフォルトのパラメータ値TDZを調べるには、この例を「デフォルトのパラメータ式」から再度検討してください。

```js
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

`add(1,1)`と`add(1)`の呼び出しは`first`と`second`パラメータ値を作るのに以下のコードを効果的に実行します：

```js
// JavaScript representation of call to add(1, 1)
let first = 1;
let second = 1;

// JavaScript representation of call to add(1)
let first = 1;
let second = getValue(first);
```

関数`add()`が最初に実行されると、バインディング`first`と`second`がパラメータ固有のTDZに追加されます(`let`の動作と同様)。 したがって、`second`は`first`の値で初期化することができますが、`first`は常にその時に初期化されるため、その逆は真ではありません。 今、この書き直された`add()`関数を考えてみましょう：

```js
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // throws error
```

この例の`add(1,1)`と`add(undefined、1)`への呼び出しは、このコードの裏にマップされるようになりました：

```js
// JavaScript representation of call to add(1, 1)
let first = 1;
let second = 1;

// JavaScript representation of call to add(undefined, 1)
let first = second;
let second = 1;
```

この例では、`first`が初期化されたときに`second`がまだ初期化されていないので、`add(undefined、1)`の呼び出しはエラーを投げます。この時点では、`second`はTDZ内にあり、したがって、`second`への参照はエラーを投げます。これは、第1章で説明した`let`バインディングの動作を反映しています。

I>関数のパラメータには、独自のスコープと、関数本体のスコープとは別の独自のTDZがあります。つまり、パラメータのデフォルト値は、関数本体の中で宣言された変数にはアクセスできません。

## 無名パラメータの操作

これまでのところ、この章の例では、関数定義で指定されたパラメータのみを扱っていました。しかし、JavaScript関数は、定義された名前付きパラメーターの数に渡すことができるパラメーターの数を制限しません。正式に指定された数よりも少なくても多くのパラメーターを渡すことができます。デフォルトのパラメータ値は、関数がより少ないパラメータを受け入れることができるときを明確にし、ECMAScript6は、よりよく定義されたよりも多くのパラメータを渡す問題を解決しようとしました。

### ECMAScript5の無名パラメータ

初期段階では、JavaScriptは`arguments`オブジェクトを、必ずしも各パラメータを個別に定義することなく渡されるすべての関数パラメータを検査する方法として提供しました。ほとんどの場合、`引数`を検査することはうまく動作しますが、このオブジェクトはやや面倒です。例えば、`arguments`オブジェクトを調べるこのコードを調べます：

```js
function pick(object) {
    let result = Object.create(null);

    // start at the second parameter
    for (let i = 1, len = arguments.length; i < len; i++) {
        result[arguments[i]] = object[arguments[i]];
    }

    return result;
}

let book = {
    title: "Understanding ECMAScript6",
    author: "Nicholas C. Zakas",
    year: 2015
};

let bookData = pick(book, "author", "year");

console.log(bookData.author);   // "Nicholas C. Zakas"
console.log(bookData.year);     // 2015
```

この関数は、*Underscore.js*ライブラリの`pick()`メソッドを模倣しています。これは、指定されたオブジェクトのコピーを元のオブジェクトのプロパティの指定されたサブセットで返します。この例では、1つの引数のみを定義し、最初の引数はプロパティをコピーするオブジェクトであると想定しています。渡される他のすべての引数は、結果にコピーする必要があるプロパティの名前です。

この`pick()`関数には注意すべきことがいくつかあります。まず、関数が複数のパラメータを処理できることはまったく明らかではありません。さらにいくつかのパラメータを定義することができますが、この関数が任意の数のパラメータを取ることができることを示すには不十分です。第2に、最初のパラメータの名前が直接使用されるので、コピーするプロパティを探すときは、インデックス0ではなくインデックス1の`arguments`オブジェクトから開始する必要があります。`arguments`で適切なインデックスを使用することを覚えていませんisn必然的に難しいですが、それを追いかけるのはもう一つのことです。

ECMAScript6では、これらの問題を解決するための残りのパラメータが導入されています。

### 残りのパラメータ

A *restパラメータ*は、指定されたパラメータの前に3つのドット(`...`)で示されます。その名前付きパラメータは、関数に渡される残りのパラメータを含む「配列」になります。これは、`restパラメータ`の起源です。例えば、`pick()`は次のようにrestパラメータを使って書き換えることができます：

```js
function pick(object, ...keys) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

このバージョンの関数では、`keys`は`object`の後に渡されるすべてのパラメータを含むrestパラメータです(`arguments`とは異なり、最初のパラメータを含むすべてのパラメータを含みます)。 つまり、キーを最初から最後まで、心配することなく繰り返すことができます。 ボーナスとして、任意の数のパラメータを処理できることを知ることができます。

I> Restパラメータは、関数の名前付きパラメータの数を示す関数の長さプロパティには影響しません。 この例の`pick()`の`length`の値は`object`だけがこの値に向かっているので1です。

#### Restパラメータの制限事項

残りのパラメータには2つの制限があります。 最初の制約は、残りのパラメータが1つだけで、残りのパラメータは最後でなければならないということです。 たとえば、次のコードは機能しません。

```js
// Syntax error: Can't have a named parameter after rest parameters
function pick(object, ...keys, last) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

ここで、パラメータlastは残りのパラメータkeysに続き、シンタックスエラーが発生します。

2番目の制約は、オブジェクトのリテラルセッターでは、残りのパラメーターを使用できないということです。つまり、このコードでもシンタックスエラーが発生します。

```js
let object = {

    // Syntax error: Can't use rest param in setter
    set name(...value) {
        // do something
    }
};
```

この制限は、オブジェクトリテラルセッターが単一の引数に制限されているために存在します。残りのパラメータは定義上、無限の数の引数なので、この文脈では許されません。

#### Restパラメータがargumentsオブジェクトに与える影響

残りのパラメータは、ECMAScriptの`arguments`を置き換えるように設計されています。もともと、ECMAScript 4は`arguments`を削除し、残りの引数を追加して無制限の引数を関数に渡すことができました。 ECMAScript 4は決して登場しませんでしたが、この考え方はECMAScript6に残されていました。

`arguments`オブジェクトは、このプログラムに示されているように、呼び出されたときに関数に渡された引数を反映することによって、restパラメータと一緒に働きます：

```js
function checkArgs(...args) {
    console.log(args.length);
    console.log(arguments.length);
    console.log(args[0], arguments[0]);
    console.log(args[1], arguments[1]);
}

checkArgs("a", "b");
```

`checkArgs()`の呼び出しは以下を出力します：

```
2
2
a a
b b
```

`arguments`オブジェクトは、restパラメータの使用法にかかわらず関数に渡されたパラメータを常に正確に反映します。

それは、あなたが本当にそれらを使用して開始するために残りのパラメータについて知っておく必要があります。

## 関数コンストラクターの機能強化

`Function`コンストラクタは、あなたが動的に新しい関数を生成することを可能にするJavaScriptのまれな部分です。コンストラクタへの引数は、関数と関数本体のパラメータであり、すべてが文字列です。ここに例があります：

```js
var add = new Function("first", "second", "return first + second");

console.log(add(1, 1));     // 2
```

ECMAScript6は、デフォルトのパラメータと残りのパラメータを許可するために、`Function`コンストラクタの機能を補強します。次のように、パラメータ名に等号と値を追加するだけで済みます。

```js
var add = new Function("first", "second = first",
        "return first + second");

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

この例では、1つのパラメータだけが渡された場合、パラメータ`second`には`first`の値が割り当てられます。シンタックスは`Function`を使用しない関数宣言と同じです。

残りのパラメータの場合は、最後のパラメータの前に`...`を追加するだけです。

```js
var pickFirst = new Function("...args", "return args[0]");

console.log(pickFirst(1, 2));   // 1
```

このコードは、単一のrestパラメータのみを使用し、渡された最初の引数を返す関数を生成します。

defaultとrestパラメータを追加すると、`Function`は宣言型の関数を生成するのと同じ機能をすべて持つことができます。

## スプレッド演算子

残りのパラメータと密接に関連するのは、スプレッド演算子です。 restパラメータでは、複数の独立した引数を配列に結合するように指定できますが、spread演算子を使用すると、分割する配列を指定し、その項目を関数の別の引数として渡すことができます。`Math.max()`メソッドを考えてみましょう。これは任意の数の引数を受け取り、最も高い値を持つものを返します。このメソッドの簡単な使用例を次に示します。

```js
let value1 = 25,
    value2 = 50;

console.log(Math.max(value1, value2));      // 50
```

この例のように2つの値だけを扱う場合、`Math.max()`は非常に使いやすいです。 2つの値が渡され、高い値が返されます。しかし、配列内の値をトラッキングしていて、今最も高い値を探したい場合はどうすればよいでしょうか？`Math.max()`メソッドは配列を渡すことを許可していません。したがって、ECMAScript5以前では、配列を自分で検索するか、次のように`apply()`を使用します。

```js
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100
```

この解決策は動作しますが、このように`apply()`を使うのはちょっと混乱します。実際には、追加のシンタックスでコードの真の意味をわかりにくくするようです。

ECMAScript6スプレッド演算子は、このケースを非常に簡単にします。`apply()`を呼び出す代わりに、配列を`Math.max()`に直接渡して、残りのパラメータで使われる同じ`...`パターンで接頭辞を付けることができます。 JavaScriptエンジンは配列を個々の引数に分割し、次のように渡します。

```js
let values = [25, 50, 75, 100]

// equivalent to
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100
```

`Math.max()`の呼び出しはもう少し従来のものになります。前の例で`Math.max.apply()`の最初の引数)を指定する複雑さを回避します数学的操作。

スプレッド演算子を他の引数と組み合わせて組み合わせることもできます。`Math.max()`から返される最小の数値が0になるようにしたいとしましょう(負の数が配列に入った場合)。その引数を別々に渡しても、次のように他の引数にspread演算子を使用することができます。

```js
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0
```

この例では、`Math.max()`に渡された最後の引数は`0`です。これは、他の引数がスプレッド演算子を使って渡された後です。

引数を渡すための普遍的な演算子は、関数引数のための配列の使用をはるかに簡単にします。たいていの場合、`apply()`メソッドの適切な置き換えである可能性が高いでしょう。

ECMAScript6では、これまでのデフォルトおよび残りのパラメータでの使用に加えて、両方のパラメータ型をJavaScriptの`Function`コンストラクタに適用することもできます。

## ECMAScript6の名前プロパティ

関数を特定するには、さまざまな方法でJavaScriptを使用する必要があります。さらに、匿名関数式の普及により、デバッグが少し難しくなり、読み取りや解読が困難なスタックトレースが発生することがよくあります。これらの理由から、ECMAScript6はすべての関数に`name`プロパティを追加します。

### 適切な名前の選択

ECMAScript6プログラムのすべての関数は、`name`プロパティに適切な値を持ちます。この動作を確認するには、関数と関数式を示す次の例を見て、両方の`name`プロパティを出力します：

```js
function doSomething() {
    // ...
}

var doAnotherThing = function() {
    // ...
};

console.log(doSomething.name);          // "doSomething"
console.log(doAnotherThing.name);       // "doAnotherThing"
```

このコードでは、`doSomething()`は`"doSomething"`と同じ`name`プロパティを持っています。これは関数宣言であるからです。無名関数式`doAnotherThing()`は、`"doAnotherThing"`の`name`を持っています。なぜなら、それが割り当てられている変数の名前だからです。

### nameプロパティの特別なケース

関数宣言と関数式の適切な名前は簡単に見つかりますが、ECMAScript6では*すべての*関数に適切な名前が付いていることが確認されています。これを説明するには、次のプログラムを検討してください。

```js
var doSomething = function doSomethingElse() {
    // ...
};

var person = {
    get firstName() {
        return "Nicholas"
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);      // "doSomethingElse"
console.log(person.sayName.name);   // "sayName"

var descriptor = Object.getOwnPropertyDescriptor(person, "firstName");
console.log(descriptor.get.name); // "get firstName"
```

この例では、`doSomething.name`は`"doSomethingElse"`です。なぜなら、関数式自体に名前があり、その名前が関数が割り当てられた変数よりも優先されるからです。 値がオブジェクトリテラルから解釈されたので、`person.sayName()`の`name`プロパティは`"sayName"`です。 同様に、`person.firstName`は実際にゲッター関数なので、その名前は`"get firstName"`です。setter関数の先頭には`"set"`も付いています。 (getterとsetterの両方の関数は、`Object.getOwnPropertyDescriptor()`を使って検索しなければなりません)。

関数名には他にも特別なケースが2つあります。`bind()`で生成された関数の名前は`"bound"`で始まり、`"Function"`のコンストラクタで生成された関数の名前は`"anonymous"`です。

```js
var doSomething = function() {
    // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous"
```

バインドされた関数の`name`は、バインドされている関数の`name`に常に文字列`'bound'`が付いているので、`doSomething()`のバインドされたバージョンは`"bound doSomething"`です。

関数の`name`の値は必ずしも同じ名前の変数を参照するとは限りません。`name`プロパティはデバッグを助けるために有益なものであるため、`name`の値を使って関数への参照を得る方法はありません。

## 関数の2つの目的を明確にする

ECMAScript5以前では、関数は`new`の有無にかかわらず呼び出し可能であるという二重の目的を果たします。`new`と共に使用すると、関数内の`this`値は新しいオブジェクトであり、この例に示すように新しいオブジェクトが返されます：

```js
function Person(name) {
    this.name = name;
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");

console.log(person);        // "[Object object]"
console.log(notAPerson);    // "undefined"
```

`notAPerson`を生成するときに、`new`なしで`Person()`を呼び出すと`undefined`となります(非厳密モードではグローバルオブジェクトに`name`プロパティを設定します)。`Person`の大文字は、JavaScriptプログラムでよく見られるように、関数が`new`を使って呼び出されることを意味する唯一の本当の指標です。関数の二重の役割に対するこの混乱は、ECMAScript6のいくつかの変更をもたらしました。

JavaScriptには関数用の2つの異なる内部専用メソッドがあります：`[[Call]]`と`[[Construct]]`。関数が`new`なしで呼び出されると、`[[Call]]`メソッドが実行され、コード内に現れる関数の本体が実行されます。関数が`new`で呼び出されると、`[[Construct]]`メソッドが呼び出されます。`[[Construct]]`メソッドは* new target *と呼ばれる新しいオブジェクトを生成し、`this`を新しいターゲットに設定して関数本体を実行します。`[[Construct]]`メソッドを持つ関数は、* constructors *と呼ばれます。

I>すべての関数が`[[Construct]]`を持っているわけではないので、すべての関数を`new`で呼び出すことはできません。 Arrow関数は、[[Construct]]`メソッドを持っていません。

### ECMAScript5で関数がどのように呼び出されたかを調べる

ECMAScript5で関数が`new`(したがって、コンストラクタとともに)で呼び出されたかどうかを判定する最も一般的な方法は、`instanceof`を使用することです。

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");  // throws error
```

ここでは、`this`値がコンストラクタのインスタンスであるかどうかを調べ、そうであれば、通常通り実行を続けます。`this`が`Person`のインスタンスでなければ、エラーがスローされます。 これは、`[[Construct]]`メソッドが`Person`の新しいインスタンスを生成し、それを` this`に割り当てるために機能します。 残念ながら、このアプローチは完全に信頼できるものではありません。この例では、`this`は`new`を使用せずに`Person`のインスタンスになる可能性があるからです。

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // works!
```

Person.call()の呼び出しは`person`変数を第1引数として渡します。これは`this`が`Person`関数の中の`person`に設定されていることを意味します。 この関数には、これを`new`で呼び出すことと区別する方法はありません。

### new.target MetaProperty

この問題を解決するために、ECMAScript6は`new.target` *メタプロパティ*を導入しました。 メタ属性は、非オブジェクトのプロパティで、ターゲットに関連する追加情報(`new`など)を提供します。 関数の[[Construct]]メソッドが呼び出されると、`new.target`は`new`演算子のターゲットで埋められます。 そのターゲットは、通常、新しく生成されたオブジェクトインスタンスのコンストラクタであり、これは関数本体の中で`this`になります。`[[Call]]`が実行された場合、`new.target`は`undefined`です。

この新しいメタ属性を使うと、`new.target`が次のように定義されているかどうかをチェックすることで、関数が`new`で呼び出されたかどうかを安全に検出できます：

```js
function Person(name) {
    if (typeof new.target !== "undefined") {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // error!
```

`this instanceof Person`の代わりに`new.target`を使うことによって、`Person`コンストラクタは`new`なしで使われると正しくエラーを投げます。

`new.target`が特定のコンストラクタで呼び出されたことを確認することもできます。 たとえば、次の例を見てください。

```js
function Person(name) {
    if (new.target === Person) {
        this.name = name;   // using new
    } else {
        throw new Error("You must use new with Person.")
    }
}

function AnotherPerson(name) {
    Person.call(this, name);
}

var person = new Person("Nicholas");
var anotherPerson = new AnotherPerson("Nicholas");  // error!
```

このコードでは、正しく動作するためには`new.target`が`Person`でなければなりません。`new AnotherPerson("Nicholas")`が呼び出されると、`Person.call(this、name)`の後続の呼び出しは`new`ターゲットが`Person`コンストラクタの中で`undefined`, `new`なしで呼ばれた)。

W>警告：関数の外で`new.target`を使うのはシンタックスエラーです。

`new.target`を追加することで、ECMAScript6は関数呼び出しのあいまいさを明確にするのに役立ちました。このテーマに続いて、ECMAScript6は、ブロック内で関数を宣言するという、以前のあいまいであった言語の別の部分も解決します。

## ブロックレベル関数

ECMAScript 3以前では、ブロック内で発生する関数宣言(*ブロックレベル関数*)は技術的にはシンタックスエラーでしたが、すべてのブラウザでサポートされていました。残念ながら、シンタックスを許可した各ブラウザは若干異なる方法で動作します。したがって、ブロック内での関数宣言を避けることがベストプラクティスです(関数式を使用することをお勧めします)。

この互換性のない動作を抑止しようとすると、ECMAScript5 strictモードでは、このようにしてブロック内で関数宣言が使用されるたびにエラーが発生しました。

```js
"use strict";

if (true) {

    // Throws a syntax error in ES5, not so in ES6
    function doSomething() {
        // ...
    }
}
```

ECMAScript5では、このコードはシンタックスエラーを投げます。 ECMAScript6では、`doSomething()`関数はブロックレベルの宣言とみなされ、定義された同じブロック内でアクセスして呼び出すことができます。 例えば：

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "undefined"
```

ブロックレベルの関数は定義されているブロックの先頭に持ち込まれるので、`typeof doSomething`はコードの関数宣言の前に現れますが、関数"を返します。`if`ブロックの実行が終了すると、`doSomething()`はもう存在しません。

### ブロックレベル関数をいつ使うべきかを決める

ブロックレベル関数は、定義されたブロックから実行が流れ出ると関数定義が削除されるという点で、関数式のlet関数に似ています。 重要な違いは、ブロックレベルの関数が包含ブロックの上部に吊り上げられていることです。 この例のように、`let`を使用する関数式は吊り上げられません：

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // throws error

    let doSomething = function () {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);
```

`let`文がまだ実行されておらず、`doSomething()`がTDZに残っているため、`typeof doSomething`が実行されるとコード実行が停止します。 この違いを知っていると、ブロックレベル関数を使用するか、巻上げ動作を行うかどうかに基づいて`let`式を使用するかどうかを選択できます。

### 非厳密モードのブロックレベル関数

ECMAScript6では、ブロックレベルの機能を非厳密モードで使用することもできますが、動作は少し異なります。 これらの宣言をブロックの上部に持ち上げるのではなく、それらを含む機能または地球環境に吊り上げます。 例えば：

```js
// ECMAScript6 behavior
if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "function"
```

この例では、`doSomething()`はグローバルスコープに持ち込まれ、`if`ブロックの外に存在します。 ECMAScript6はこの動作を標準化して、以前存在していた互換性のないブラウザの動作を削除しました。そのため、すべてのECMAScript6ランタイムは同じように動作するはずです。

ブロックレベルの関数を許可すると、JavaScriptで関数を宣言する能力が向上しますが、ECMAScript6では、関数を宣言する全く新しい方法も導入されました。

## 矢印関数

ECMAScript6の最も興味深い新しい部分の1つは、*arrow関数*です。矢印関数は、名前が示すように、「矢印」(`=>`)を使用する新しいシンタックスで定義された関数です。しかし、矢印関数は従来のJavaScript関数とはいくつかの重要な点で異なって動作します。

* **`this`, ` super`, `arguments`, `new.target`バインディング** -`this`, `super`, `arguments`, `new.target`の値は、関数は、最も近い非小関数を含むものである。 (`super`は第4章でカバーしています)
* **`new`では呼び出すことができません** - 矢印関数は`[[Construct]]`メソッドを持たず、したがってコンストラクタとして使うことはできません。矢印関数は`new`と一緒に使うとエラーになります。
* **プロトタイプはありません** - 矢印関数に`new`を使うことはできないので、プロトタイプは必要ありません。矢印関数の`prototype`プロパティは存在しません。
* **`this`を変更することはできません** - 関数内の`this`の値は変更できません。関数のライフサイクル全体にわたって同じままです。
* **いいえ`arguments`オブジェクト** - 矢印関数には`arguments`バインディングがないので、関数引数にアクセスするには、名前付き引数と残りの引数に依存する必要があります。
* **重複した名前のパラメータはありません。** - 矢印関数は、strictモードでのみ名前付きパラメータを重複させることができない非小規模な関数ではなく、strictまたはnonstrictモードで名前付きパラメータを重複させることはできません。

これらの違いにはいくつかの理由があります。まず第一に、このバインディングはJavaScriptの一般的なエラーの原因です。関数内の`this`値の追跡を失うのは非常に簡単です。意図しないプログラムの振る舞いにつながり、矢印関数がこの混乱を排除します。第2に、矢印関数を単に`this`値でコードを実行するように制限することで、JavaScriptエンジンは、コンストラクタとして使用されるか、または変更される通常の関数とは異なり、これらの操作をより簡単に最適化できます。

相違の残りの部分は、矢印関数内のエラーとあいまいさを減らすことにも焦点を当てています。これにより、JavaScriptエンジンは矢印関数の実行を最適化することができます。

I> 注意：矢印関数には、他の関数と同じ規則に従う`name`プロパティもあります。

### 矢印関数のシンタックス

arrow関数のシンタックスは、あなたが達成しようとしているものに応じて多くの種類があります。すべてのバリエーションは、関数の引数で始まり、その後に矢印が続き、関数の本体が続きます。引数と本文は、用途に応じて異なる形式をとることができます。たとえば、次の矢印関数は単一の引数をとり、単純にそれを返します。

```js
var reflect = value => value;

// effectively equivalent to:

var reflect = function(value) {
    return value;
};
```

矢印関数の引数が1つしかない場合、その1つの引数はそれ以上のシンタックスなしで直接使用できます。 矢印が次に来て、矢印の右側の式が評価され、返されます。 明示的な`return`文がなくても、この矢印関数は渡された最初の引数を返します。

複数の引数を渡す場合は、次のようにそれらの引数のまわりにカッコを入れる必要があります。

```js
var sum = (num1, num2) => num1 + num2;

// effectively equivalent to:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

`sum()`関数は2つの引数を単に加算して結果を返します。 この矢印関数と`reflect()`関数の唯一の違いは、引数がカッコで区切られていることです(伝統的な関数のように)。

関数への引数がない場合は、次のように、宣言に空の括弧セットを含める必要があります。

```js
var getName = () => "Nicholas";

// effectively equivalent to:

var getName = function() {
    return "Nicholas";
};
```

おそらく複数の式からなる、より伝統的な関数本体を提供したいときは、このバージョンの`sum()`のように関数本体を中括弧で囲み、明示的に戻り値を定義する必要があります：

```js
var sum = (num1, num2) => {
    return num1 + num2;
};

// effectively equivalent to:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

中括弧の内側は、従来の関数と同じように扱うことができますが、`arguments`は利用できません。

何もしない関数を生成する場合は、次のように中括弧を組み込む必要があります。

```js
var doNothing = () => {};

// effectively equivalent to:

var doNothing = function() {};
```

中括弧は関数の本体を示すために使用されています。 しかし、関数本体の外側にオブジェクトリテラルを返すようにする矢印関数は、リテラルをカッコで囲む必要があります。 例えば：

```js
var getTempItem = id => ({ id: id, name: "Temp" });

// effectively equivalent to:

var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
};
```

括弧内のオブジェクトリテラルをラップすると、中括弧は関数本体ではなくオブジェクトリテラルです。

### 直ちに呼び出される関数式を生成する

JavaScriptでよく使われる関数の1つは、直ちに呼び出される関数式(IIFE)の生成です。 IIFEを使用すると、無名関数を定義し、参照を保存せずにすぐに呼び出すことができます。 このパターンは、プログラムの残りの部分から遮蔽されたスコープを生成したいときに便利です。 例えば：

```js
let person = function(name) {

    return {
        getName: function() {
            return name;
        }
    };

}("Nicholas");

console.log(person.getName());      // "Nicholas"
```

このコードでは、IIFEを使用して`getName()`メソッドを持つオブジェクトを生成します。 このメソッドは`name`引数を戻り値として使用し、`name`を実際に返されるオブジェクトのprivateメンバーにします。

矢印関数を括弧で囲んでいる限り、同じことを矢印関数を使って行うことができます：

```js
let person = ((name) => {

    return {
        getName: function() {
            return name;
        }
    };

})("Nicholas");

console.log(person.getName());      // "Nicholas"
```

括弧は`("Nicholas")`の周りではなく、矢の関数定義の周りにのみあることに注意してください。 これは正式な関数とは異なり、渡されたパラメータの外に、関数定義のまわりにかっこを置くことができます。

### このバインディングはありません

JavaScriptの最も一般的なエラー領域の1つは、関数の中に`this`をバインドすることです。`this`の値は、関数が呼び出されるコンテキストに応じて単一の関数の内部で変更できるので、別のオブジェクトに影響を与えることを意図したときにあるオブジェクトに誤って影響する可能性があります。 次の例を考えてみましょう。

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // error
        }, false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

このコードでは、オブジェクト`PageHandler`は、ページ上の対話を処理するように設計されています。`init()`メソッドは相互作用を設定するために呼び出され、そのメソッドは`this.doSomething()`を呼び出すイベントハンドラを割り当てます。 しかし、このコードは意図したとおりに正確に動作しません。

`this.doSomething()`の呼び出しは、`this`が` PageHandler`にバインドされるのではなく、イベントのターゲットであったオブジェクト(この場合は`document`)への参照であるため、壊れています。 このコードを実行しようとすると、`this.doSomething()`がターゲット`document`オブジェクトに存在しないため、イベントハンドラが起動するとエラーが発生します。

関数の`bind()`メソッドを明示的に`PageHandler`にバインドすることで、これを修正することができます。

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // no error
        }).bind(this), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

コードは期待通りに機能しますが、少し奇妙に見えるかもしれません。`bind(this)`を呼び出すことによって、`this`が現在の`this`にバインドされた新しい関数を実際に生成しています。これは`PageHandler`です。余分な関数を生成しないようにするには、このコードを修正するより良い方法は、矢印関数を使用することです。

矢印関数には`this`バインディングがありません。つまり、矢印関数内の`this`の値は、スコープチェーンを調べることによってのみ決定できます。矢印関数が非小関数内に含まれている場合、`this`は包含関数と同じになります。それ以外の場合、`this`はグローバルスコープの`this`の値と等価です。矢印関数を使用してこのコードを書く方法の1つは次のとおりです。

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

この例のイベントハンドラは`this.doSomething()`を呼び出す矢印関数です。`this`の値は`init()`の中と同じですので、このバージョンのコードは`bind(this)`を使った場合と同様に動作します。`doSomething()`メソッドは値を返しませんが、関数本体で実行される唯一のステートメントなので、中カッコを入れる必要はありません。

矢印関数は「使い捨て」関数として設計されているため、新しい型を定義するために使用することはできません。これは通常の関数には存在しない`prototype`プロパティがないことから明らかです。 arrow関数で`new`演算子を使用しようとすると、この例のようにエラーが発生します：

```js
var MyType = () => {},
    object = new MyType();  // error - you can't use arrow functions with 'new'
```

このコードでは、`MyType`は矢印関数であるため、`[[Construct]]`の動作を持たないため、`new MyType()`の呼び出しは失敗します。 矢印関数を`new`と一緒に使うことができないことが分かっていれば、JavaScriptエンジンはその動作をさらに最適化できます。

また、`this`の値はarrow関数が定義されている包含関数によって決まるため、`call()`, `apply()`, `bind()`を使って`this`の値を変更することはできません。

### 矢印の関数と配列

矢印関数の簡潔なシンタックスは、配列処理での使用にも理想的です。 たとえば、カスタムコンパレータを使用して配列をソートする場合、通常は次のように記述します。

```js
var result = values.sort(function(a, b) {
    return a - b;
});
```

それは非常に簡単な手順のための多くのシンタックスです。 それをより簡潔な矢印関数のバージョンと比較してください：

```js
var result = values.sort((a, b) => a - b);
```

`sort()`, `map()`, `reduce()`のようなコールバック関数を受け入れる配列メソッドは、一見複雑なプロセスをよりシンプルなコードに変更するより簡単な矢印関数のシンタックスの恩恵を受けることができます。

### 引数なし

矢印関数は独自の`arguments`オブジェクトを持っていませんが、それを含む関数から`arguments`オブジェクトにアクセスすることは可能です。 後で矢印関数がどこで実行されても、その`arguments`オブジェクトは利用可能です。 例えば：

```js
function createArrowFunctionReturningFirstArg() {
    return () => arguments[0];
}

var arrowFunction = createArrowFunctionReturningFirstArg(5);

console.log(arrowFunction());       // 5
```

`createArrowFunctionReturningFirstArg()`の内部では、生成された矢印関数によって`arguments[0]`要素が参照されます。 この参照には、`createArrowFunctionReturningFirstArg()`関数に渡される最初の引数が含まれています。 矢印関数が後で実行されると、`createArrowFunctionReturningFirstArg()`に渡された最初の引数だった`5`を返します。 矢印関数はもはやそれを生成した関数のスコープには入っていませんが、`arguments`は`arguments`識別子のスコープチェーン解決のためにアクセス可能なままです。

### 矢印関数を識別する

異なるシンタックスにもかかわらず、矢印関数は依然として関数であり、そのように識別されます。 次のコードを考えてみましょう：

```js
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true
```

`console.log()`出力は、`typeof`と`instanceof`の両方が、他の関数と同じように矢印関数と同じように動作することを示しています。

他の関数と同様に、関数の`this`バインディングには影響しませんが、矢印関数に`call()`, `apply()`, `bind()`を使用することはできます。 ここではいくつかの例を示します。

```js
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3

var boundSum = sum.bind(null, 1, 2);

console.log(boundSum());                // 3
```

`sum()`関数は、関数と同様に`call()`と`apply()`を使って引数を渡します。`bind()`メソッドは`boundsum()`を生成するために使われ、2つの引数が`1`と` 2`に束縛されているので、直接渡す必要はありません。

Arrow関数は、コールバックなど、現在無名関数式を使用している場所であればどこでも使用できます。 次のセクションではECMAScript6の主要な開発について説明しますが、これはすべて内部的なものであり、新しいシンタックスはありません。

## テールコールの最適化

おそらく、ECMAScript6の機能の最も興味深い変更は、テールコールシステムを変更するエンジンの最適化です。 A * tail呼び出し*は、ある関数が別の関数の最後の文として呼び出されたときです。

```js
function doSomething() {
    return doSomethingElse();   // tail call
}
```

ECMAScript5エンジンで実装されているテールコールは、ほかの関数呼び出しと同様に処理されます。新しいスタックフレームが生成され、関数呼び出しを表すために呼び出しスタックにプッシュされます。これは、以前のすべてのスタックフレームがメモリに保持されていることを意味します。これは、コールスタックが大きくなりすぎると問題になります。

### 何が違うの？

ECMAScript6では、厳密なモードで特定のテールコールのコールスタックのサイズを縮小しようとしています(非厳密モードのテールコールはそのままです)。この最適化では、テールコール用の新しいスタックフレームを生成する代わりに、次の条件が満たされている限り、現在のスタックフレームがクリアされ再利用されます。

1.テールコールは現在のスタックフレーム内の変数へのアクセスを必要としません(つまり、関数はクロージャではありません)
テールコールを行う関数は、テールコールが返った後にそれ以上の作業を行う必要はありません。
1.テールコールの結果が関数値として返されます

たとえば、このコードは3つの条件すべてに適合するため、簡単に最適化できます。

```js
"use strict";

function doSomething() {
    // optimized
    return doSomethingElse();
}
```

この関数は`doSomethingElse()`への末尾呼び出しを行い、すぐに結果を返し、ローカルスコープ内の変数にはアクセスしません。 結果を返さない小さな変化の1つは、最適化されていない関数になります。

```js
"use strict";

function doSomething() {
    // not optimized - no return
    doSomethingElse();
}
```

同様に、テールコールから戻った後に操作を実行する関数がある場合、その関数を最適化することはできません：

```js
"use strict";

function doSomething() {
    // not optimized - must add after returning
    return 1 + doSomethingElse();
}
```

この例では、値を返す前に`doSomethingElse()`の結果を1で加算して、最適化を無効にします。

誤って最適化をオフにする別の一般的な方法は、関数呼び出しの結果を変数に格納し、次にそのような結果を返すことです。

```js
"use strict";

function doSomething() {
    // not optimized - call isn't in tail position
    var result = doSomethingElse();
    return result;
}
```

`doSomethingElse()`の値がすぐに返されないので、この例は最適化できません。

おそらく、避けるのが最も難しい状況はクロージャを使用することです。 クロージャは、包含スコープ内の変数にアクセスできるため、テールコールの最適化がオフになっている可能性があります。 例えば：

```js
"use strict";

function doSomething() {
    var num = 1,
        func = () => num;

    // not optimized - function is a closure
    return func();
}
```

クロージャ`func()`はこの例ではローカル変数`num`にアクセスできます。`func() 'を呼び出すとすぐに結果が返されますが、変数numを参照することで最適化はできません。

### テールコール最適化を活用する方法

実際には、テールコールの最適化はバックグラウンドで行われるため、関数を最適化しない限り、テールコールの最適化について考える必要はありません。 テールコール最適化の主な使用例は、最適化が最も効果的であるような再帰関数です。 階乗を計算するこの関数を考えてみましょう：

```js
function factorial(n) {

    if (n <= 1) {
        return 1;
    } else {

        // not optimized - must multiply after returning
        return n * factorial(n - 1);
    }
}
```

関数のこのバージョンは、`factorial()`への再帰呼び出しの後に乗算が行われる必要があるため、最適化することはできません。`n`が非常に大きい場合、呼び出しスタックのサイズが大きくなり、スタックオーバーフローを引き起こす可能性があります。

関数を最適化するには、最後の関数呼び出しの後に乗算が行われないようにする必要があります。 これを行うには、デフォルトパラメータを使用して、`return`ステートメントの外で乗算演算を動かすことができます。 結果の関数は、一時的な結果に沿って次の繰り返しに移り、ECMAScript6エンジンで同じように動作し、最適化できる関数を生成します。 新しいコードは次のとおりです：

```js
function factorial(n, p = 1) {

    if (n <= 1) {
        return 1 * p;
    } else {
        let result = n * p;

        // optimized
        return factorial(n - 1, result);
    }
}
```

この再書式化された`factorial()`では、第2引数`p`がデフォルト値1のパラメータとして追加されます。`p`パラメータは前の乗算結果を保持し、次の結果が別の関数コール。`n`が1より大きい場合、乗算は最初に行われ、次にfactorial()への第2引数として渡されます。これにより、ECMAScript6エンジンは再帰呼び出しを最適化できます。

テールコールの最適化は、特に計算量の多い関数に適用した場合に、大幅なパフォーマンスの向上をもたらすことができるため、再帰関数を記述するたびに考えるべきことです。

## まとめ

関数はECMAScript6では巨大な変更を受けていませんでしたが、作業を容易にする一連の段階的な変更がありました。

デフォルトの関数パラメータを使用すると、特定の引数が渡されないときに使用する値を簡単に指定できます。 ECMAScript6より前のバージョンでは、引数の有無を確認して別の値を割り当てるために、関数内に余分なコードが必要でした。

残りのパラメータを使用すると、残りのすべてのパラメータを配置する配列を指定できます。実際の配列を使用して、どのパラメータをインクルードするかを指定させると、restパラメータは`arguments`よりはるかに柔軟な解決策になります。

スプレッド演算子は、関数を呼び出すときに、配列を別々のパラメータに分解することを可能にする、残りのパラメータの仲間です。 ECMAScript6以前では、配列に含まれる個々のパラメータを渡す方法は2つしかありませんでした。手動で各パラメータを指定するか、`apply()`を使用します。スプレッド演算子を使うと、関数の`this`束縛を心配することなく、配列を任意の関数に簡単に渡すことができます。

`name`プロパティを追加すると、デバッグや評価のための関数をより簡単に識別するのに役立ちます。さらに、ECMAScript6ではブロックレベルの関数の動作が正式に定義されているため、厳密なモードではシンタックスエラーにはなりません。

ECMAScript6では、関数が`new[]`で呼び出されたとき、`[[Call]]`、通常の関数実行、`[[Construct]]`で定義されます。`new.target`メタ属性は`new`を使って関数が呼び出されたかどうかを判定することもできます。

ECMAScript6の機能の最大の変更は、矢印機能の追加でした。矢印関数は、無名関数式の代わりに使用するように設計されています。 Arrow関数はより簡潔なシンタックス、字句`this`バインディング、`arguments`オブジェクトはありません。さらに、矢印関数は`this`バインディングを変更することができないので、コンストラクタとして使用することはできません。

テールコールの最適化により、小さなコールスタックを保持し、メモリを少なくし、スタックオーバーフローエラーを防ぐために、いくつかの関数呼び出しを最適化することができます。この最適化はエンジンが自動的に適用されますが、安全な場合には自動的に適用されますが、この最適化を利用するには再帰関数を書き換えることがあります。
