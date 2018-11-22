# より簡単なデータアクセスのための破壊

オブジェクトリテラルと配列リテラルは、JavaScriptで最も頻繁に使用される表記法の2つで、一般的なJSONデータフォーマットのおかげで、言語の特に重要な部分となっています。 オブジェクトや配列を定義し、それらの構造から関連する情報を体系的に引き出すことは、非常に一般的です。 ECMAScriptは、* destructuring *を追加することでこの作業を簡素化します。これは、データ構造をより小さな部分に分割するプロセスです。 この章では、オブジェクトと配列の両方の非構造化を利用する方法を示します。

## なぜ破壊は有用ですか？

ECMAScript5以前では、オブジェクトや配列から情報を取得する必要があるため、特定のデータをローカル変数に取得するだけで、同じように見えるコードが大量に発生する可能性がありました。 例えば：

```js
let options = {
        repeat: true,
        save: false
    };

// extract data from the object
let repeat = options.repeat,
    save = options.save;
```

このコードは、`options`オブジェクトから`repeat`と`save`の値を抽出し、そのデータを同じ名前のローカル変数に格納します。このコードは単純に見えますが、割り当てる変数が多数ある場合は想像してください。それらをすべて1つずつ割り当てる必要があります。そして、代わりに情報を見つけるために横断するネストされたデータ構造があった場合は、ただ一つのデータを見つけるために構造全体を調べなければならないかもしれません。

ECMAScriptでは、オブジェクトと配列の両方に破壊が加えられています。データ構造を小さな部分に分割すると、必要な情報をより簡単に得ることができます。多くの言語は、プロセスをより簡単に使用できるように最小限の構文で非構造化を実装しています。 ECMAScriptの実装では、あなたがすでによく知っている構文を使用します：オブジェクトリテラルと配列リテラルの構文です。

## オブジェクトの破壊

オブジェクトの構造化構文では、代入操作の左側にオブジェクトリテラルが使用されます。例えば：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

このコードでは、`node.type`の値は`type`という変数に格納され、`node.name`の値は`name`という変数に格納されます。 この構文は第4章で紹介したオブジェクトリテラルプロパティの初期化子の略語と同じです。識別子`type`と`name`はローカル変数の宣言と`node`オブジェクトの値を読み込むプロパティです。

A> ####イニシャライザを忘れないでください
A>
A>`var`、`let`、または`const`を使って変数を宣言するために非構造化を使うときは、イニシャライザ(等号の後の値)を与えなければなりません。 次のコード行は、初期化子がないために構文エラーが発生します。
A>
A>```js
A>// syntax error!
A>var { type, name };
A>
A>// syntax error!
A>let { type, name };
A>
A>// syntax error!
A>const { type, name };
A>```
A>
A> constは常に初期化子を必要としますが、構造化されていない変数を使用していても、`var`と`let`は構造化を使うときに初期化子しか必要としません。

#### 割り当てを破棄する

これまでのオブジェクトの構造化の例では、変数の宣言が使用されています。 ただし、代入での構造解除を使用することもできます。 たとえば、変数が定義された後、次のように変数の値を変更することができます。

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

// assign different values using destructuring
({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

この例では、`type`と`name`は宣言時に値で初期化され、同じ名前の2つの変数は異なる値で初期化されます。 次の行は、`node`オブジェクトから読み取ることによってそれらの値を変更するために、構造解除割り当てを使用します。 消滅代入文の前後にかっこを入れる必要があることに注意してください。 これは、開始中括弧がブロックステートメントであると予想され、ブロックステートメントが割り当ての左側に表示されないためです。 括弧は、次の中括弧がブロックステートメントではないことを示し、割り当てを完了できるように式として解釈する必要があります。

destructuring代入式は、式の右側(`=`の後)で評価されます。 つまり、値が期待される場所であればどこでも、構造化代入式を使用できます。 たとえば、ある関数に値を渡す：

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

function outputInfo(value) {
    console.log(value === node);        // true
}

outputInfo({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

`outputInfo()`関数はdestructuring代入式で呼び出されます。 式は、式の右辺の値であるため、`node`と評価されます。`type`と`name`への代入はどちらも正常に動作し、`node`は`outputInfo()`に渡されます。

W>構造化代入式の右側(`=`の後の式)が`null`または`undefined`に評価されると、エラーがスローされます。 これは、`null`または`undefined`のプロパティーを読み取ろうとするとランタイムエラーが発生するためです。

#### デフォルト値

構造化代入文を使用するとき、オブジェクトに存在しないプロパティ名を持つローカル変数を指定すると、そのローカル変数には`undefined`の値が割り当てられます。 例えば：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // undefined
```

このコードは`value`という追加のローカル変数を定義し、それに値を割り当てようとします。 しかし、`node`オブジェクトには対応する`value`プロパティがないので、変数は`undefined`の値を期待通りに割り当てられます。

指定したプロパティが存在しない場合に使用するデフォルト値をオプションで定義できます。 これを行うには、プロパティ名の後に等号(`=`)を挿入し、次のようにデフォルト値を指定します。

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value = true } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // true
```

この例では、変数`value`はデフォルト値として`true`が与えられています。 デフォルト値は、プロパティが`node`にない場合、または`undefined`の値を持つ場合にのみ使用されます。`node.value`プロパティはないので、変数`value`はデフォルト値を使います。 これは、第3章で説明したように、関数のデフォルトのパラメータ値と同様に機能します。

#### 異なるローカル変数名への割り当て

ここまでは、オブジェクトの名前をローカル変数名として使用していました。 たとえば、`node.type`の値は`type`変数に格納されていました。 それはあなたが同じ名前を使用したいときはうまくいくが、そうでない場合はどうなるだろうか？ ECMAScriptには拡張された構文があり、ローカル変数に別の名前を付けることができ、その構文はオブジェクトのリテラルの非公式プロパティイニシャライザ構文のように見えます。 ここに例があります：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type: localType, name: localName } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "foo"
```

このコードでは、`nodeType`と`node.name`プロパティからの値をそれぞれ含む`localType`と`localName`変数を宣言するために、非構造化割り当てを使います。`type：localType`という構文は、`type`という名前のプロパティを読み込み、その値を`localType`変数に格納することを示します。 この構文は、名前がコロンの左側にあり、値が右側にある従来のオブジェクトリテラル構文とは事実上反対です。 この場合、名前はコロンの右側にあり、読み込む値の位置は左側にあります。

異なる変数名を使用する場合は、デフォルト値を追加することもできます。 等号とデフォルト値は、引き続きローカル変数名の後に置かれます。 例えば：

```js
let node = {
        type: "Identifier"
    };

let { type: localType, name: localName = "bar" } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "bar"
```

ここで、`localName`変数のデフォルト値は``bar '`です。`node.name`プロパティがないので、変数にはデフォルト値が割り当てられます。

これまでは、プロパティがプリミティブな値であるオブジェクトの非構造化に対処する方法を見てきました。 オブジェクトの構造化を使用して、ネストされたオブジェクト構造の値を取得することもできます。

#### ネストされたオブジェクトの破棄

オブジェクトリテラルに似た構文を使用すると、ネストされたオブジェクト構造にナビゲートして、必要な情報だけを取得できます。 ここに例があります：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

let { loc: { start }} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
```

この例の構造解除パターンは、中括弧を使用して、パターンが`node`の`loc`という名前のプロパティに降りて、`start`プロパティを探していることを示しています。 最後のセクションから、コロンが破壊パターンにあるときはいつでも、コロンが検査する位置を与える前の識別子を意味し、右側は値を割り当てます。 コロンの後に中括弧がある場合、それは目的地がオブジェクトの別のレベルにネストされていることを示します。

一歩進んで、ローカル変数にも別の名前を使うことができます：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

// extract node.loc.start
let { loc: { start: localStart }} = node;

console.log(localStart.line);   // 1
console.log(localStart.column); // 1
```

このバージョンのコードでは、`node.loc.start`は`localStart`と呼ばれる新しいローカル変数に格納されます。 破壊パターンは任意のレベルの深度にネストすることができ、各レベルですべての機能を使用できます。

オブジェクトの構造化は非常に強力で多くの選択肢がありますが、配列の構造化は、配列から情報を抽出できる独自の機能を提供します。

A> ####構文Gotcha
A>
A>効果がない文を誤って作成する可能性があるため、ネストされた非構造化を使用する場合は注意してください。 空の中括弧はオブジェクトの構造化には合法ですが、何もしません。 例えば：
A>
A>```js
A>// no variables declared!
A>let { loc: {} } = node;
A>```
A>
A>このステートメントに宣言されているバインディングはありません。 右側の中括弧のため、`loc`は作成するバインディングではなく検査する場所として使用されます。 そのような場合は、`：`を使わずにデフォルト値を定義して位置を定義するという意図があったようです。 この構文が将来不法になる可能性はありますが、今のところ、これは目を引くためのものです。

## Array Destructuring

配列の非構造化構文は、オブジェクトの構造化と非常によく似ています。 オブジェクトリテラル構文の代わりに配列リテラル構文を使用するだけです。 構造解除は、オブジェクト内で使用可能な名前付きプロパティではなく、配列内の位置で機能します。 例えば：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

ここでは、配列の破壊は、``red``と``green``を`colors`配列から引き出し、`firstColor`と`secondColor`変数に格納します。 これらの値は配列内の位置のために選択されます。 実際の変数名は何でもかまいません。 構造解除パターンに明示的に記載されていない項目はすべて無視されます。 配列自体は決して変更されないことに注意してください。

また、消滅パターン内の項目を省略して、関心のある項目の変数名のみを提供することもできます。たとえば、配列の3番目の値を必要とする場合は、最初の変数名を指定する必要はありません 2番目の項目。 それはどのように動作するのです：

```js
let colors = [ "red", "green", "blue" ];

let [ , , thirdColor ] = colors;

console.log(thirdColor);        // "blue"
```

このコードは、`colors`で3番目の項目を取り出すために、非構造化割り当てを使います。 パターン内の`thirdColor`の前のコンマは、その前に来る配列項目のプレースホルダです。 この方法を使用すると、配列の途中にある任意の数のスロットから変数名を指定する必要なく、値を簡単に選択できます。

W>オブジェクトの破壊と同様に、`var`、`let`、`const`を使って配列を破壊するときは、常に初期化子を指定する必要があります。

#### 割り当てを破棄する

代入のコンテキストで配列の非構造化を使用することはできますが、オブジェクトの構造化とは異なり、式をかっこで囲む必要はありません。 例えば：

```js
let colors = [ "red", "green", "blue" ],
    firstColor = "black",
    secondColor = "purple";

[ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

このコードの構造化されていない代入は、最後の配列の非構造化の例と同じように機能します。 唯一の違いは、`firstColor`と`secondColor`がすでに定義されていることです。 ほとんどの場合、おそらくアレイの構造化の割り当てについて知っておく必要があるでしょうが、おそらく有用であると思われるものがもう少しあります。

配列の構造化の割り当てには、2つの変数の値を簡単に交換できる非常にユニークな使用例があります。 バリュー・スワッピングはソート・アルゴリズムの一般的な操作であり、ECMAScript5の変数のスワップ方法では、この例のように3番目の一時変数が使用されます。

```js
// Swapping variables in ECMAScript5
let a = 1,
    b = 2,
    tmp;

tmp = a;
a = b;
b = tmp;

console.log(a);     // 2
console.log(b);     // 1
```

一時変数`tmp 'は`a`と`b`の値を入れ替えるために必要です。 ただし、配列の構造化代入を使用すると、余分な変数は必要ありません。 ECMAScriptで変数をスワップする方法は次のとおりです。

```js
// Swapping variables in ECMAScript
let a = 1,
    b = 2;

[ a, b ] = [ b, a ];

console.log(a);     // 2
console.log(b);     // 1
```

この例の配列の構造解除代入は、鏡像のように見えます。 代入の左辺(等号の前)は、他の配列の非構造化の例と同様の非構造化パターンです。 右側は、スワップ用に一時的に作成される配列リテラルです。 破壊は一時的な配列上で起こります。一時的な配列は、`b`と`a`の値が1番目と2番目の位置にコピーされています。 その効果は、変数が値を入れ替えたことです。

W>オブジェクトの非構造化の割り当てと同様に、配列の非構造化代入式が`null`または`undefined`と評価されると、エラーがスローされます。

#### デフォルト値

配列の構造解除の割り当てでは、配列内の任意の位置のデフォルト値を指定することもできます。 デフォルト値は、指定された位置のプロパティが存在しないか値が未定義の場合に使用されます。 例えば：

```js
let colors = [ "red" ];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

このコードでは`colors`配列にはアイテムが1つしかないので、`secondColor`が一致するものはありません。 デフォルト値があるので、`secondColor`は`undefined`ではなく``green``に設定されます。

#### 入れ子構造の破壊

ネストされたオブジェクトを破壊するのと同様の方法で、ネストされた配列を解体することができます。 別の配列パターンを全体のパターンに挿入することによって、破壊は次のようにネストされた配列に降下します：

```js
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// later

let [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

ここで、`secondColor`変数は、``colors``配列の``green"`の値を参照します。 その項目は2番目の配列に含まれているので、構造化パターンの`secondColor`の周りに余分な角括弧が必要です。 オブジェクトと同様に、配列を任意に深く入れ子にすることができます。

#### レストアイテム

第3章では関数の残りのパラメータを紹介し、配列の破壊は* rest items *と同じ概念を持っています。 残りの項目は`... '構文を使用して配列の残りの項目を特定の変数に割り当てます。 ここに例があります：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, ...restColors ] = colors;

console.log(firstColor);        // "red"
console.log(restColors.length); // 2
console.log(restColors[0]);     // "green"
console.log(restColors[1]);     // "blue"
```

`colors`の最初の項目は`firstColor`に割り当てられ、残りは新しい`restColors`配列に割り当てられます。 したがって、`restColors`配列には``green``と``blue"`という2つの項目があります。 残りの項目は配列から特定の項目を抽出し、残りの部分を利用可能にするのに便利ですが、便利な使い方があります。

JavaScript配列からの目立たない欠点は、簡単にクローンを作成できることです。 ECMAScript5では、開発者は配列を簡単に複製するために`concat()`メソッドをよく使用していました。 例えば：

```js
// cloning an array in ECMAScript5
var colors = [ "red", "green", "blue" ];
var clonedColors = colors.concat();

console.log(clonedColors);      //"[red,green,blue]"
```

`concat()`メソッドは2つの配列を連結するためのものですが、引数を指定しないで呼び出すと配列のクローンが返されます。 ECMAScriptでは、restアイテムを使用して、そのように機能するように意図された構文で同じことを達成できます。 それはこのように動作します：

```js
// cloning an array in ECMAScript
let colors = [ "red", "green", "blue" ];
let [ ...clonedColors ] = colors;

console.log(clonedColors);      //"[red,green,blue]"
```

この例では、`colors`配列の値を`clonedColors`配列にコピーするためにrest項目が使用されています。 この手法が`concat()`メソッドを使用するよりも開発者の意図をより明確にするかどうかについての認識の問題ですが、これは知っておくと便利な機能です。

W>レストアイテムは、非構造化配列内の最後のエントリでなければならず、コンマが続くことはできません。 残りの項目の後にカンマを含めると構文エラーです。

## 混合破壊

より複雑な式を作成するために、オブジェクトと配列の非構造化を一緒に使用できます。 そうすることで、オブジェクトと配列の混在から必要な情報のみを抽出することができます。 例えば：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        },
        range: [0, 3]
    };

let {
    loc: { start },
    range: [ startIndex ]
} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
console.log(startIndex);        // 0
```

このコードは`node.loc.start`と`node.range [0]`を`start`と`startIndex`にそれぞれ抽出します。 destructuredパターンの`loc：`と`range：`は`node`オブジェクトのプロパティに対応する場所に過ぎないことに注意してください。 オブジェクトと配列の非構造化を混在させて使用するときに、非構造化を使って抽出できない`node`の部分はありません。 このアプローチは、構造全体をナビゲートせずにJSON構成構造から値を引き出す場合に特に便利です。

## 破壊されたパラメータ

Destructuringには、特に有用なユースケースが1つあります。これは、関数の引数を渡すときです。 JavaScript関数が多数のオプションパラメータを取る場合、一般的なパターンの1つは`options`オブジェクトを作成し、そのプロパティは次のように追加のパラメータを指定します：

```js
// properties on options represent additional parameters
function setCookie(name, value, options) {

    options = options || {};

    let secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;

    // code to set the cookie
}

// third argument maps to options
setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```
多くのJavaScriptライブラリには、このような`setCookie()`関数が含まれています。 この関数では、`name`と`value`引数は必須ですが、`secure`、`path`、`domain`、`expires`はありません。 また、他のデータの優先順位はないので、追加の名前付きパラメータをリストするのではなく、名前付きプロパティを持つ`options`オブジェクトを持つだけで効率的です。 このアプローチは機能しますが、関数定義を見るだけでどのような入力が期待しているのかを知ることはできません。 関数本体を読む必要があります。

破壊されたパラメータは、関数がどのような引数を期待しているかを明確にする代替手段を提供します。 非構造化パラメータは、名前付きパラメータの代わりにオブジェクトまたは配列の非構造化パターンを使用します。 これを実際に見るには、最後の例の`setCookie()`関数のこの書き直し版を見てください：

```js
function setCookie(name, value, { secure, path, domain, expires }) {

    // code to set the cookie
}

setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

この関数は前の例と同様に動作しますが、3番目の引数は構造化を使用して必要なデータを引き出します。破壊されたパラメータの外側のパラメータは明らかに期待されていますが、同時に`setCookie()`を使っている人にとっては余分な引数でどのようなオプションが利用できるのかは明らかです。もちろん、第3引数が必要な場合は、その値に明瞭な値を入れる必要があります。破壊されたパラメータは、渡されないと`undefined`に設定されるという点で、通常のパラメータのようにも動作します。

A>破壊されたパラメータには、これまでのところこの章で学んだ破壊のすべての機能があります。デフォルト値を使用したり、オブジェクトと配列のパターンを混在させたり、読み込み中のプロパティとは異なる変数名を使用することができます。

### 破棄されたパラメータは必須です

構造化されていないパラメータを使用することの1つは、デフォルトでは、関数呼び出しで提供されていないときにエラーがスローされるということです。たとえば、前の例の`setCookie()`関数を呼び出すと、エラーがスローされます。

```js
// Error!
setCookie("type", "js");
```

3番目の引数が欠けているので、期待どおりに`undefined`と評価されます。 これは、構造化されていないパラメータは実際には構造化宣言の省略形であるため、エラーが発生します。`setCookie()`関数が呼ばれると、JavaScriptエンジンは実際にこれを行います：

```js
function setCookie(name, value, options) {

    let { secure, path, domain, expires } = options;

    // code to set the cookie
}
```

右辺の式が`null`または`undefined`と評価されたときに、破壊がエラーをスローするので、第3引数が`setCookie()`関数に渡されないときも同じことが言えます。

destructuredパラメータを必要とする場合は、この動作がすべて問題になるわけではありません。 しかし、destructuredパラメータをオプションにしたい場合は、destructuredパラメータのデフォルト値を次のように指定することで回避できます。

```js
function setCookie(name, value, { secure, path, domain, expires } = {}) {

    // ...
}
```

この例では、新しいオブジェクトを3番目のパラメータのデフォルト値として提供しています。 destructuredパラメータのデフォルト値を指定すると、`secure`、`path`、`domain`、`expires`は`setCookie()`の3番目の引数が与えられていなければ`undefined`になります。 エラーがスローされます。

### 破棄されたパラメータのデフォルト値

破壊された代入の場合と同じように、非構造化パラメータの非構造化デフォルト値を指定することができます。 パラメータの後に等号を追加し、デフォルト値を指定するだけです。 例えば：

```js
function setCookie(name, value,
    {
        secure = false,
        path = "/",
        domain = "example.com",
        expires = new Date(Date.now() + 360000000)
    } = {}
) {

    // ...
}
```

destructuredパラメータの各プロパティはこのコードでデフォルト値を持つため、正しい値を使用するために特定のプロパティが含まれているかどうかを確認する必要はありません。また、destructuredパラメータ全体が空のオブジェクトのデフォルト値を持ち、パラメータをオプションにします。これにより、関数宣言は通常より少し複雑に見えますが、それは各引数が使用可能な値を持つことを確保するために支払う小さな代償です。

## まとめ

破壊は、JavaScriptのオブジェクトや配列を簡単に処理します。おなじみのオブジェクトリテラルと配列リテラル構文を使用すると、興味のある情報だけを取得するためにデータ構造を選ぶことができます。オブジェクトパターンではオブジェクトからデータを抽出でき、配列パターンでは配列からデータを抽出できます。

オブジェクトと配列の両方のdestructuringは、`undefined`であるプロパティや項目のデフォルト値を指定することができ、割り当ての右側が`null`または`undefined`と評価されたときにエラーをスローします。また、オブジェクトおよび配列の破壊を任意の深度まで降りて深く入れ子になったデータ構造をナビゲートすることもできます。

破壊宣言は、変数を作成するために`var`、`let`、または`const`を使います。常に初期化子を持たなければなりません。分割割当は、他の割当の代わりに使用され、オブジェクトプロパティおよび既存の変数に分解することができます。

破壊されたパラメータは、関数パラメータとして使用されたときに`option`オブジェクトをより透明にするために、構造解除構文を使用します。あなたが興味を持っている実際のデータは、他の名前付きパラメータと一緒にリストアップすることができます。破壊されたパラメータは、配列パターン、オブジェクトパターン、または混合物にすることができ、破壊のすべての機能を使用することができます。
