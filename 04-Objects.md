# 拡張されたオブジェクト機能

ECMAScript6はオブジェクトの有用性を向上させることに重点を置いています.JavaScriptのほとんどの値はある種のオブジェクトなので理にかなっています。さらに、JavaScriptアプリケーションの複雑さが増すにつれて、平均的なJavaScriptプログラムで使用されるオブジェクトの数が増え続けています。つまり、プログラムによって常に多くのオブジェクトが生成されています。より多くのオブジェクトを使用すると、それらをより効果的に使用する必要が生じます。

ECMAScript6は、単純なシンタックスの拡張から、操作や操作のためのオプションまで、さまざまな方法でオブジェクトを改善します。

## オブジェクトカテゴリ

JavaScriptではブラウザやNode.jsなどの実行環境で追加されたものとは対照的に、標準で見つかったオブジェクトを記述するために用語が混在しており、ECMAScript6仕様ではオブジェクトのカテゴリごとに明確な定義があります。言語全体をよく理解するには、この用語を理解することが重要です。オブジェクトのカテゴリは次のとおりです。

* *通常のオブジェクト* JavaScriptのオブジェクトのすべてのデフォルトの内部動作を持ちます。
* *エキゾチックなオブジェクト*何らかの形でデフォルトとは異なる内部的な振る舞いを持つ。
* *標準オブジェクト* ECMAScript6で定義されたもので、ArrayやDateなどです。標準的なオブジェクトは、通常のものでもエキゾチックなものでもよい
* *ビルトインオブジェクト*スクリプトの実行開始時にJavaScript実行環境に存在します。すべての標準オブジェクトはビルトインオブジェクトです。

ECMAScript6で定義されているさまざまなオブジェクトを説明するために、これらの用語を本で使用します。

## オブジェクトリテラルシンタックスの拡張

オブジェクトリテラルは、JavaScriptで最も一般的なパターンの1つです。 JSONはそのシンタックスに基づいて構築されており、インターネット上のほぼすべてのJavaScriptファイルにあります。オブジェクトリテラルは、それがそうでなければ数行のコードを取るオブジェクトを生成するための簡潔なシンタックスなので、人気があります。幸いなことに開発者にとって、ECMAScript6は、シンタックスをいくつかの方法で拡張することで、オブジェクトリテラルをより強力かつより簡潔にします。

### プロパティ初期化子略奪

ECMAScript 5以前では、オブジェクトリテラルは単に名前と値のペアの集合でした。つまり、プロパティ値が初期化されるときに重複が生じる可能性があります。例えば：

```js
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
}
```

`createPerson()`関数は、プロパティ名が関数のパラメータ名と同じであるオブジェクトを生成します。その結果は、一方がオブジェクトプロパティの名前で他方がそのプロパティの値を提供しても、`name`と`age`の重複であるように見えます。返されるオブジェクトのキー`name`には変数`name`に含まれる値が割り当てられ、返されたオブジェクトのキー`age`には変数`age`に含まれる値が割り当てられます。

ECMAScript6では、*プロパティ初期化子*の短縮形を使用して、プロパティ名とローカル変数の周りに存在する重複を排除できます。オブジェクトのプロパティ名がローカル変数名と同じ場合は、コロンと値のない名前を単純に含めることができます。たとえば、次のように、`createPerson()`をECMAScript6に書き換えることができます：

```js
function createPerson(name, age) {
    return {
        name,
        age
    };
}
```

オブジェクトリテラルのプロパティーに名前のみがある場合、JavaScriptエンジンは周囲のスコープ内で同じ名前の変数を探します。見つかった場合は、その変数の値がオブジェクトリテラルの同じ名前に割り当てられます。この例では、オブジェクトのリテラル・プロパティー`name`にローカル変数`name`の値が割り当てられています。

この拡張により、オブジェクトのリテラルの初期化がより簡潔になり、命名エラーを排除するのに役立ちます。ローカル変数と同じ名前のプロパティを割り当てることは、JavaScriptの非常に一般的なパターンであり、この拡張を歓迎します。

### 簡潔なメソッド

ECMAScript6では、オブジェクトリテラルにメソッドを割り当てるためのシンタックスも改善されています。 ECMAScript 5以前では、オブジェクトにメソッドを追加するには、名前を指定してから完全な関数定義を指定する必要があります。

```js
var person = {
    name: "Nicholas",
    sayName: function() {
        console.log(this.name);
    }
};
```

ECMAScript6では、コロンと`function`キーワードを削除することで、シンタックスがより簡潔になります。つまり、前の例を次のように書き直すことができます。

```js
var person = {
    name: "Nicholas",
    sayName() {
        console.log(this.name);
    }
};
```

*簡潔なメソッド*シンタックスとも呼ばれるこの簡略シンタックスは、前の例のように`person`オブジェクトにメソッドを生成します。`sayName()`プロパティは無名関数に割り当てられ、ECMAScript 5の`sayName()`関数と同じ特性を持っています。 1つの違いは、簡潔なメソッドでは "super"を使うことができるということです(非標準メソッドでは簡単なPrototype Access with Super Referencesで説明します)。

I>簡略メソッドを使って生成されたメソッドの`name`プロパティは、かっこの前に使用される名前です。最後の例では、`person.sayName()`の`name`プロパティは``sayName"`です。

### 計算されたプロパティ名

ECMAScript 5以前では、プロパティがドット表記ではなく角かっこで設定されている場合、オブジェクトインスタンスのプロパティ名を計算できました。角括弧では、識別子で使用されるとシンタックスエラーの原因となる文字を含む変数や文字列リテラルを使用してプロパティ名を指定できます。ここに例があります：

```js
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

`lastName`に``lastname``の値が割り当てられているので、この例の両方のプロパティー名はスペースを使い、ドット表記法を使ってそれらを参照することはできません。しかし、括弧表記法は任意の文字列値をプロパティ名として使用できるので、``Nicholas '`と``last name"`を``Zakas'``に割り当てることができます。

さらに、文字列リテラルをオブジェクトリテラルのプロパティ名として直接使用することもできます。

```js
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas"
```

このパターンは、事前にわかっており、文字列リテラルで表現できるプロパティ名に対して機能します。しかし、プロパティ名`"ファーストネーム "が変数に含まれているか(前の例のように)計算されなければならない場合、ECMAScript 5のオブジェクトリテラルを使ってそのプロパティを定義する方法はありません。

ECMAScript6では、計算されたプロパティ名はオブジェクトリテラルシンタックスの一部であり、オブジェクトインスタンスの計算されたプロパティ名を参照するために使用されているのと同じ角括弧表記を使用します。例えば：

```js
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

オブジェクトリテラル内の角かっこは、プロパティ名が計算されていることを示しているため、その内容は文字列として評価されます。つまり、次のような式を含めることもできます。

```js
var suffix = " name";

var person = {
    ["first" + suffix]: "Nicholas",
    ["last" + suffix]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person["last name"]);       // "Zakas"
```

これらのプロパティは``first name '`と``last name'`と評価され、これらの文字列は後でプロパティを参照するために使用できます。オブジェクトインスタンスでブラケット記法を使用している間に角括弧の中に入れるものは、オブジェクトリテラル内の計算されたプロパティ名に対しても機能します。

## newメソッド

ECMAScriptの設計目標の1つは、ECMAScript 5で始まり、`Object.prototype`で新しいグローバル関数やメソッドを生成するのを避け、代わりに新しいメソッドを利用できるようなオブジェクトを見つけようとしました。結果として、`Object`グローバルは、他のオブジェクトがより適切でないときに、メソッドの数が増加しています。 ECMAScript6では、いくつかのタスクを容易にするために設計された`Object`グローバルにいくつかの新しいメソッドが導入されています。

### Object.is()メソッド

JavaScriptで2つの値を比較したい場合は、equals演算子(`==`)または同等に等しい演算子(`===`)のどちらかを使用することに多分慣れています。比較の際に型強制を避けるために、多くの開発者が後者を好んでいます。しかし、同等の演算子であっても、完全に正確ではありません。例えば、+0と-0の値は、JavaScriptエンジンで異なって表現されていても、`=== 'と等しいとみなされます。また、`NaN === NaN`は`false`を返します。これは`NaN`を正しく検出するために`isNaN()`を使う必要があります。

ECMAScript6では、Object.is()メソッドが導入され、同等に等しい演算子の残りのクォークを補完します。このメソッドは2つの引数を受け取り、値が等しい場合は`true`を返します。 2つの値は、同じ型で同じ値を持つ場合、等価と見なされます。ここではいくつかの例を示します。

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

多くの場合、`Object.is()`は`===`演算子と同じ働きをします。 唯一の違いは+0と-0が等価でないとみなされ、`NaN`が`NaN`と等価であるとみなされることです。 しかし、等価演算子の使用を完全に停止する必要はありません。 それらの特殊なケースがあなたのコードに与える影響に基づいて、`==`や`===`の代わりに`Object.is()`を使うかどうかを選択します。

### Object.assign()メソッド

*ミックスイン*は、JavaScriptのオブジェクト合成で最も一般的なパターンです。 ミックスインでは、あるオブジェクトは別のオブジェクトからプロパティとメソッドを受け取ります。 多くのJavaScriptライブラリには、次のようなmixinメソッドがあります。

```js
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });

    return receiver;
}
```

`mixin()`関数は`supplier`の独自のプロパティを繰り返し処理し、`receiver`(浅いコピー、プロパティ値がオブジェクトのときにオブジェクト参照が共有される)にコピーします。 これは、このコードのように、`receiver`が継承なしで新しいプロパティを得ることを可能にします：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
};

var myObject = {};
mixin(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

ここで、`myObject`は`EventTarget.prototype`オブジェクトから動作を受け取ります。これにより`myObject`はイベントを公開し、それぞれ`emit()`と`on()`メソッドを使ってイベントを購読することができます。

このパターンは、ECMAScript6が同じように動作する`Object.assign()`メソッドを追加し、受信者と任意の数のサプライヤを受け入れ、その後受信者を返すほど広く普及しました。`mixin()`から`assign()`への名前の変更は、実際の操作が反映されます。`mixin()`関数は代入演算子(`=`)を使うので、アクセサプロパティをアクセサープロパティとしてレシーバにコピーすることはできません。この区別を反映するために、`Object.assign()`という名前が選択されました。

I>さまざまなライブラリの類似のメソッドは、同じ基本機能の別の名前を持つことがあります。一般的な代替方法には`extend()`と`mix()`メソッドがあります。また、簡単に言うと、`Object.assign()`メソッドに加えて、ECMAScript6に`Object.mixin()`メソッドがありました。主な相違点は、`Object.mixin()`もアクセサプロパティ上にコピーされましたが、`super`の使用に対する懸念からメソッドが削除されたことです(この章の「簡単なプロトタイプアクセス」参照) 。

`mixin()`関数が使われていたなら、どこでも`Object.assign()`を使うことができます。ここに例があります：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
}

var myObject = {}
Object.assign(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

`Object.assign()`メソッドは任意の数のサプライヤを受け付け、サプライヤが指定された順序でプロパティを受け取ります。つまり、2番目のサプライヤが受信者の最初のサプライヤからの値を上書きする可能性があります。これはこのスニペットで行われます。

```js
var receiver = {};

Object.assign(receiver,
    {
        type: "js",
        name: "file.js"
    },
    {
        type: "css"
    }
);

console.log(receiver.type);     // "css"
console.log(receiver.name);     // "file.js"
```

The value of`receiver.type`is`"css"`because the second supplier overwrote the value of the first.

The`Object.assign()`method isn't a big addition to ECMAScript6, but it does formalize a common function found in many JavaScript libraries.

A> ### Working with Accessor Properties
A>
A> Keep in mind that`Object.assign()`doesn't create accessor properties on the receiver when a supplier has accessor properties. Since`Object.assign()`uses the assignment operator, an accessor property on a supplier will become a data property on the receiver. For example:
A>
A>```js
A> var receiver = {},
A>     supplier = {
A>         get name() {
A>             return "file.js"
A>         }
A>     };
A>
A> Object.assign(receiver, supplier);
A>
A> var descriptor = Object.getOwnPropertyDescriptor(receiver, "name");
A>
A> console.log(descriptor.value);      // "file.js"
A> console.log(descriptor.get);        // undefined
A>```
A>
A>このコードでは、`supplier`は`name`というアクセサプロパティを持っています。`Object.assign()`メソッドを使用した後、`receiver.name`は`file.js "の値を持つデータプロパティとして存在します。`supplier.name`は`File.js" .assign()`が呼び出されました。

## 重複するオブジェクトリテラルのプロパティ

ECMAScript 5 strictモードでは、重複したオブジェクトリテラルプロパティのチェックが導入されました。重複が見つかった場合はエラーを投げます。 たとえば、このコードは問題がありました。

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // syntax error in ES5 strict mode
};
```

ECMAScript 5 strictモードで実行すると、2番目の`name`プロパティはシンタックスエラーを引き起こします。しかし、ECMAScript6では、重複したプロパティチェックが削除されました。厳密モードと非厳密モードの両方のコードで重複したプロパティがチェックされなくなりました。代わりに、ここに示すように、指定された名前の最後のプロパティがプロパティの実際の値になります。

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // no error in ES6 strict mode
};

console.log(person.name);       // "Greg"
```

この例では、`person.name`の値は`Greg "です。なぜなら、それがプロパティに割り当てられた最後の値だからです。

## プロパティの列挙順序

ECMAScript 5はオブジェクトプロパティの列挙順序を定義していませんでした.JavaScriptエンジンのベンダーに任せていました。ただし、ECMAScript6では、列挙されたときに独自のプロパティを返す順序を厳密に定義しています。これは、`Object.getOwnPropertyNames()`と`Reflect.ownKeys`(第12章で扱う)を使ってプロパティを返す方法に影響します。`Object.assign()`によってプロパティが処理される順序にも影響します。

独自のプロパティ列挙の基本的な順序は次のとおりです。

1.昇順のすべての数字キー
2.オブジェクトに追加された順序でのすべての文字列キー
3.オブジェクトに追加された順にすべてのシンボルキー(第6章で説明)

ここに例があります：

```js
var obj = {
    a: 1,
    0: 1,
    c: 1,
    2: 1,
    b: 1,
    1: 1
};

obj.d = 1;

console.log(Object.getOwnPropertyNames(obj).join(""));     // "012acbd"
```

`Object.getOwnPropertyNames()`メソッドは`obj`のプロパティを`0`、`1`、`2`、`a`、`c`、`b`、d`の順に返します。テンキーは、オブジェクトリテラルで順不同で表示されていても、グループ化されソートされていることに注意してください。文字列キーは数字キーの後に来て、`obj`に追加された順に表示されます。オブジェクトリテラルのキー自体が最初に来て、その後に追加された動的キー(この場合は`d`)が続きます。

W>`for-in`ループは、すべてのJavaScriptエンジンが同じように実装しているわけではないので、未定義の列挙命令を持っています。`Object.keys()`メソッドと`JSON.stringify()`はどちらも`for-in`と同じ(指定されていない)列挙型を使うように指定されています。

列挙の順序はJavaScriptの仕組みに微妙な変更がありますが、特定の列挙命令に依存するプログラムが正しく動作することは珍しくありません。列挙命令を定義することにより、ECMAScript6は列挙に頼っているJavaScriptコードがどこで実行されているかにかかわらず正しく動作するようにします。

## より強力なプロトタイプ

プロトタイプはJavaScriptの継承の基礎であり、ECMAScript6はプロトタイプをより強力にし続けています。 JavaScriptの初期のバージョンでは、プロトタイプでできることを厳しく制限していました。しかし、言語が成熟し、開発者がプロ​​トタイプの仕組みに慣れてきたので、開発者はプロトタイプをよりコントロールし、より簡単な方法で作業できるようになりました。その結果、ECMAScript6ではプロトタイプにいくつかの改良が加えられました。

### オブジェクトのプロトタイプを変更する

通常、オブジェクトのプロトタイプは、オブジェクトの生成時にコンストラクタまたは`Object.create()`メソッドを介して指定されます。インスタンス化後にオブジェクトのプロトタイプが変更されないという考えは、ECMAScript 5を使ったJavaScriptプログラミングの最大の想定の1つでした。ECMAScript 5では、オブジェクトのプロトタイプを取得するために`Object.getPrototypeOf()`メソッドを追加しました。インスタンス化後にオブジェクトのプロトタイプを変更するための標準的な方法です。

ECMAScript6は、`Object.setPrototypeOf()`メソッドを追加することでその前提を変更します。これにより、任意のオブジェクトのプロトタイプを変更することができます。`Object.setPrototypeOf()`メソッドは、プロトタイプが変更されるべきオブジェクトと最初の引数のプロトタイプになるオブジェクトの2つの引数を受け取ります。例えば：

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
let friend = Object.create(person);
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true

// set prototype to dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

このコードは`person`と`dog`の2つの基本オブジェクトを定義しています。両方のオブジェクトには文字列を返す`getGreeting()`メソッドがあります。オブジェクト`friend`は`person`オブジェクトから継承します。つまり`getGreeting()`は``Hello "`を出力します。プロトタイプが`dog`オブジェクトになると、`person.getGreeting()`は`person`への元の関係が壊れているので``Woof``を出力します。

オブジェクトのプロトタイプの実際の値は`[[Prototype]]`と呼ばれる内部専用プロパティに格納されます。`Object.getPrototypeOf()`メソッドは`[[Prototype]]`に格納された値を返し、`Object.setPrototypeOf()`は`[[Prototype]]`に格納された値を変更します。しかし、これらは`[[Prototype]]`の値を扱う唯一の方法ではありません。

### スーパーリファレンスによる簡単なプロトタイプアクセス

前述のように、プロトタイプはJavaScriptにとって非常に重要であり、ECMAScript6での使用を容易にするための多くの作業が行われました。もう1つの改善点は、オブジェクトのプロトタイプの機能へのアクセスを容易にする`super`参照の導入です。たとえば、同じ名前のプロトタイプメソッドも呼び出すように、オブジェクトインスタンスのメソッドをオーバーライドするには、次のようにします。

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


let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};

// set prototype to person
Object.setPrototypeOf(friend, person);
console.log(friend.getGreeting());                      // "Hello, hi!"
console.log(Object.getPrototypeOf(friend) === person);  // true

// set prototype to dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof, hi!"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

この例では、`friend`の`getGreeting()`は同じ名前のプロトタイプメソッドを呼び出します。`Object.getPrototypeOf()`メソッドは正しいプロトタイプが呼び出されたことを保証し、その後に追加の文字列が出力に追加されます。追加の`.call(this)`は、プロトタイプメソッド内の`this`値が正しく設定されていることを保証します。

プロトタイプのメソッドを呼び出すために`Object.getPrototypeOf()`と`.call(this) 'を使うことは少し関わっているので、ECMAScript6は`super`を導入しました。最も単純な場合、`super`は現在のオブジェクトのプロトタイプへのポインタであり、実質的に`Object.getPrototypeOf(this)`の値です。それを知っているなら、`getGreeting()`メソッドを次のように単純化することができます：

```js
let friend = {
    getGreeting() {
        // in the previous example, this is the same as:
        // Object.getPrototypeOf(this).getGreeting.call(this)
        return super.getGreeting() + ", hi!";
    }
};
```

`super.getGreeting()`への呼び出しは、この文脈では`Object.getPrototypeOf(this).getGreeting.call(this)`と同じです。同様に、簡潔なメソッドの中にある限り、オブジェクトのプロトタイプのどのメソッドも`super`参照を使って呼び出すことができます。簡潔なメソッドの外で`super`を使用しようとすると、この例のようにシンタックスエラーが発生します。

```js
let friend = {
    getGreeting: function() {
        // syntax error
        return super.getGreeting() + ", hi!";
    }
};
```

この例では、関数で名前付きプロパティを使用しています。このコンテキストで`super`が無効であるため、`super.getGreeting()`を呼び出すとシンタックスエラーが発生します。

`super`リファレンスは、あなたが複数のレベルの継承を持つときに本当に強力です。その場合、`Object.getPrototypeOf()`はもはやすべての状況で機能しないからです。例えば：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// prototype is person
let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// prototype is friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // error!
```

`relative.getGreeting()`が呼ばれると、`Object.getPrototypeOf()`の呼び出しはエラーになります。これは`this`が`relative`で、`relative`のプロトタイプが`friend`オブジェクトですからです。`friend.getGreeting()。call()`が`relative`を`this`として呼び出されると、プロセスはやり直され、スタックオーバーフローエラーが発生するまで再帰的に呼び出しを続けます。

この問題はECMAScript 5では解決するのが難しいですが、ECMAScript6と`super`では簡単です：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// prototype is person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// prototype is friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // "Hello, hi!"
```

`super`参照は動的ではないので、常に正しいオブジェクトを参照します。この場合、`super.getGreeting()`は、メソッドを継承する他のオブジェクトの数にかかわらず、常に`person.getGreeting()`を参照します。

## 形式的なメソッド定義

ECMAScript6以前は、`method`の概念は形式的に定義されていませんでした。メソッドは、データの代わりに関数を含むオブジェクトプロパティでした。 ECMAScript6はメソッドを正式に、メソッドが属するオブジェクトを含む内部の[[HomeObject]]プロパティを持つ関数として定義します。次の点を考慮してください。

```js
let person = {

    // method
    getGreeting() {
        return "Hello";
    }
};

// not a method
function shareGreeting() {
    return "Hi!";
}
```

この例は`getGreeting()`と呼ばれる単一のメソッドで`person`を定義しています。`getGreeting()`の`[[HomeObject]]`は、関数をオブジェクトに直接割り当てることによって`person`です。一方、`shareGreeting()`関数はオブジェクトの生成時にオブジェクトに割り当てられていないため、[[HomeObject]]は指定されていません。ほとんどの場合、この違いは重要ではありませんが、`super`参照を使用する場合は非常に重要になります。

`super`への参照は`[[HomeObject]]`を使って何をすべきかを決定します。最初のステップは`[[HomeObject]]`で`Object.getPrototypeOf()`を呼び出してプロトタイプへの参照を取得することです。次に、プロトタイプで同じ名前の関数が検索されます。最後に、`this`バインディングが設定され、メソッドが呼び出されます。ここに例があります：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// prototype is person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);

console.log(friend.getGreeting());  // "Hello, hi!"
```

`friend.getGreeting()`を呼び出すと、`person.getGreeting()`の値と``、hi！ "`の値を組み合わせた文字列が返されます。`friend.getGreeting()`の`[[HomeObject]]`は`friend`であり、`friend`のプロトタイプは`person`なので、`super.getGreeting()`は`person.getGreeting.call (これ)`。

## まとめ

オブジェクトはJavaScriptのプログラミングの中心であり、ECMAScript6では、オブジェクトを扱いやすく、より強力なものにする便利な変更が加えられました。

ECMAScript6はオブジェクトリテラルをいくつか変更します。省略名のプロパティ定義では、スコープ内変数と同じ名前のプロパティを簡単に割り当てることができます。計算されたプロパティ名を使用すると、リテラル以外の値をプロパティ名として指定できます。これは、すでに言語の他の領域で実行できました。簡略法では、コロンや`function`キーワードを完全に省略することで、オブジェクトリテラルのメソッドを定義するために、より少ない文字数を入力できます。 ECMAScript6は厳密なモードチェックを緩め、オブジェクトリテラルの重複したプロパティ名もチェックします。つまり、単一のオブジェクトリテラルに同じ名前の2つのプロパティを持つことができます。

`Object.assign()`メソッドは、一度に一つのオブジェクトに対して複数のプロパティを変更することを容易にします。 mixinパターンを使用すると非常に便利です。`Object.is()`メソッドは、任意の値に対して厳密な等価性を実行し、特殊なJavaScript値を扱うときには安全に`===`のバージョンになります。

ECMAScript6では、独自のプロパティの列挙順序が明確になりました。プロパティを列挙するときは、数字キーが常に最初に昇順になり、その後に文字列キーが挿入順、記号キーが挿入順に続きます。

ECMAScript6の`Object.setPrototypeOf()`メソッドのおかげで、オブジェクトのプロトタイプがすでに生成された後に修正することが可能になりました。

最後に、`super`キーワードを使ってオブジェクトのプロトタイプのメソッドを呼び出すことができます。`super`を使って呼び出されたメソッド内の`this`バインディングは、`this`の現在の値で自動的に動作するように設定されています。
