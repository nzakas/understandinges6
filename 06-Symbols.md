# シンボルとシンボルのプロパティ

シンボルはECMAScript6で導入されたプリミティブ型であり、文字列、数値、ブール値、ヌル、および未定義のプリミティブ型を結合します。シンボルは、プライベートオブジェクトメンバを作成する方法として始まりました.JavaScript開発者が長い間望んでいた機能です。記号の前には、文字列名を持つプロパティは名前の不明瞭さにかかわらず簡単にアクセスでき、`private name`機能は開発者が文字列以外のプロパティ名を作成できるようにするためのものでした。そうすれば、これらのプライベートな名前を検出するための通常の技術はうまくいかないでしょう。

プライベートネームの提案は、最終的にECMAScript6のシンボルに進化しました。この章では、シンボルを効果的に使用する方法について説明します。実装の詳細は同じままですが(つまり、プロパティ名に文字列以外の値を追加した)、プライバシーの目標は落とされました。代わりに、シンボルプロパティは他のオブジェクトプロパティとは別に分類されます。

## シンボルを作成する

シンボルは、ブール値の場合は`true`、数字の場合は`42`など、リテラル形式を持たないという点で、JavaScriptプリミティブ間でユニークです。次の例のように、グローバルな`Symbol`関数を使ってシンボルを作成することができます：

```js
let firstName = Symbol();
let person = {};

person[firstName] = "Nicholas";
console.log(person[firstName]);     // "Nicholas"
```

ここで、シンボル`firstName`が作成され、`person`オブジェクトに新しいプロパティを割り当てるために使用されます。 そのシンボルにアクセスするたびに、そのシンボルを使用する必要があります。 シンボル変数を適切に命名することは良い考えです。したがって、シンボルが何を表しているかを簡単に知ることができます。

W>シンボルはプリミティブな値なので、`new Symbol()`を呼び出すと、呼び出されたときにエラーがスローされます。`Symbol`のインスタンスを`new Object(yourSymbol) 'で作成することもできますが、この機能が役立つのは不明です。

`Symbol`関数は、シンボルの説明であるオプションの引数も受け入れます。 記述自体はプロパティにアクセスするために使用することはできませんが、デバッグの目的で使用されます。 例えば：

```js
let firstName = Symbol("first name");
let person = {};

person[firstName] = "Nicholas";

console.log("first name" in person);        // false
console.log(person[firstName]);             // "Nicholas"
console.log(firstName);                     // "Symbol(first name)"
```

シンボルの説明は`[[Description]]`プロパティに内部的に格納されます。 このプロパティは、シンボルの`toString()`メソッドが明示的にまたは暗黙的に呼び出されるたびに読み込まれます。 この例では、`firstName`シンボルの`toString()`メソッドは`console.log()`によって暗黙に呼び出され、その説明がログに出力されます。 そうでなければコードから直接[[Description]]`にアクセスすることはできません。 私はいつも読書とデバッグの両方の記号をより簡単にするための記述を提供することを勧めました。

A> ###記号を識別する
A>
A>シンボルはプリミティブな値なので、`typeof`演算子を使って変数にシンボルが含まれているかどうかを判断できます。 ECMAScript6は`typeof`を拡張してシンボルに使用するとき``symbol"`を返します。 例えば：
A>
A>```js
A>let symbol = Symbol("test symbol");
A>console.log(typeof symbol);         // "symbol"
A>```
A>
A>変数がシンボルかどうかを判断する他の間接的な方法がありますが、`typeof`演算子は最も正確で好ましい方法です。

## シンボルの使用

計算されたプロパティ名を使用する場所であればどこでもシンボルを使用できます。 この章のシンボルで使用されている括弧表記は既に見てきましたが、計算されたオブジェクトリテラルのプロパティ名や、Object.defineProperty()およびObject.defineProperties()

```js
let firstName = Symbol("first name");

// use a computed object literal property
let person = {
    [firstName]: "Nicholas"
};

// make the property read only
Object.defineProperty(person, firstName, { writable: false });

let lastName = Symbol("last name");

Object.defineProperties(person, {
    [lastName]: {
        value: "Zakas",
        writable: false
    }
});

console.log(person[firstName]);     // "Nicholas"
console.log(person[lastName]);      // "Zakas"
```

この例では、最初に計算されたオブジェクトリテラルプロパティを使用して`firstName`シンボルプロパティを作成します。次の行は、プロパティを読み取り専用に設定します。その後、`Object.defineProperties()`メソッドを使用して、読み取り専用`lastName`シンボルプロパティが作成されます。計算されたオブジェクト・リテラル・プロパティーがもう一度使用されますが、今回は`Object.defineProperties()`コールの2番目の引数の一部です。

計算されたプロパティ名が許可されている場所でシンボルを使用することができますが、効果的に使用するには、これらのシンボルを異なるコード間で共有するシステムが必要です。

## シンボルを共有する

同じシンボルを使用するコードの異なる部分が必要な場合があります。たとえば、同じシンボルプロパティを使用して一意の識別子を表す2つの異なるオブジェクト型がアプリケーション内にあるとします。ファイルや大きなコードベースでシンボルを追跡するのは難しく、エラーを起こしやすくなります。そのため、ECMAScript6は、いつでもアクセスできるグローバルなシンボルレジストリを提供します。

共有するシンボルを作成する場合は、`Symbol()`メソッドを呼び出す代わりに`Symbol.for()`メソッドを使用してください。`Symbol.for()`メソッドは、作成したいシンボルの文字列識別子である単一のパラメータを受け入れます。このパラメータは、シンボルの説明としても使用されます。例えば：

```js
let uid = Symbol.for("uid");
let object = {};

object[uid] = "12345";

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"
```

`Symbol.for()`メソッドは、まずグローバルシンボルレジストリを検索して、キー "uid"を持つシンボルが存在するかどうかを調べます。 そうである場合、メソッドは既存のシンボルを返します。 そのようなシンボルが存在しない場合、新しいシンボルが作成され、指定されたキーを使用してグローバルシンボルレジストリに登録されます。 新しいシンボルが返されます。 つまり、同じキーを使って`Symbol.for()`を呼び出すと、次のように同じシンボルが返されます：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"

let uid2 = Symbol.for("uid");

console.log(uid === uid2);      // true
console.log(object[uid2]);      // "12345"
console.log(uid2);              // "Symbol(uid)"
```

この例では、`uid`と`uid2`には同じシンボルが含まれているので、それらは同じ意味で使用できます。 Symbol.for()への最初の呼び出しはシンボルを作成し、2番目の呼び出しはグローバルシンボルレジストリからシンボルを取得します。

共有シンボルのもう一つの特徴は、`Symbol.keyFor()`メソッドを呼び出すことによって、グローバルシンボルレジストリ内のシンボルに関連するキーを取得できることです。 例えば：

```js
let uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));    // "uid"

let uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));   // "uid"

let uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));   // undefined
```

`uid`と`uid2`の両方が``uid "`キーを返します。`uid3`シンボルはグローバルシンボルレジストリには存在しないため、関連するキーはなく、`Symbol.keyFor()`は`undefined`を返します。

W>グローバルシンボルレジストリは、グローバルスコープと同様、共有環境です。つまり、その環境に何が存在しているのか、存在していないのかを前提にすることはできません。シンボルキーの名前空間を使用すると、サードパーティのコンポーネントを使用するときに名前の衝突の可能性を減らすことができます。たとえば、jQueryコードでは、すべてのキーの前に``jquery。 ''などを使用したり、``jquery.element ''などのキーを使用したりすることがあります。

## シンボル強制

型強制はJavaScriptの重要な部分であり、あるデータ型を別の型に強制する能力には柔軟性があります。しかし、シンボルは、他のタイプがシンボルと論理的に等価でないため、強制的には柔軟性がありません。具体的には、シンボルを文字列または数字に強制変換することはできません。そうしないと、誤ってシンボルとして動作することが予想されるプロパティとして誤って使用されることはありません。

この章の例では`console.log()`を使ってシンボルの出力を示しています。これは`console.log()`が有用な出力を生成するためにシンボル上で`String()`を呼び出すためです。`String()`を直接使用して同じ結果を得ることができます。例えば：

```js
let uid = Symbol.for("uid"),
    desc = String(uid);

console.log(desc);              // "Symbol(uid)"
```

`String()`関数は`uid.toString()`を呼び出し、シンボルの文字列の説明が返されます。 ただし、シンボルを文字列と直接連結しようとすると、エラーがスローされます。

```js
let uid = Symbol.for("uid"),
    desc = uid + "";            // error!
```

`uid`を空文字列に連結するには、`uid`を最初に強制的に文字列に変換する必要があります。 強制が検出されるとエラーがスローされ、このように使用されなくなります。

同様に、記号を数字に強制することもできません。 すべての数学演算子は、記号に適用するとエラーを引き起こします。 例えば：

```js
let uid = Symbol.for("uid"),
    sum = uid / 1;            // error!
```

この例では、シンボルを1で除算しようとします。これはエラーの原因となります。 使用される数学演算子に関係なく、エラーがスローされます(論理演算子は、JavaScriptの空でない他の値と同様に、すべてのシンボルが`true`と同等と見なされるため、エラーをスローしません)。

## シンボルプロパティの取得

`Object.keys()`と`Object.getOwnPropertyNames()`メソッドは、オブジェクト内のすべてのプロパティ名を取り出すことができます。 前者のメソッドはすべての列挙可能なプロパティ名を返し、後者は列挙可能性に関係なくすべてのプロパティを返します。 ただし、どちらのメソッドもECMAScript5の機能を保持するためにシンボルプロパティを返しません。 代わりに、オブジェクトからプロパティシンボルを取得できるように、`Object.getOwnPropertySymbols()`メソッドがECMAScript6に追加されました。

`Object.getOwnPropertySymbols()`の戻り値は、独自のプロパティシンボルの配列です。 例えば：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

let symbols = Object.getOwnPropertySymbols(object);

console.log(symbols.length);        // 1
console.log(symbols[0]);            // "Symbol(uid)"
console.log(object[symbols[0]]);    // "12345"
```

このコードでは、`object`は`uid`という単一シンボルプロパティを持っています。`Object.getOwnPropertySymbols()`から返される配列は、そのシンボルだけを含む配列です。

すべてのオブジェクトはゼロの独自のシンボルプロパティで始まりますが、オブジェクトはプロトタイプからシンボルプロパティを継承できます。 ECMAScript6は、よく知られているシンボルを使って実装されたいくつかのそのようなプロパティを事前定義しています。

## よく知られているシンボルを使った内部操作の公開

ECMAScript5の主なテーマは、JavaScriptの「魔法」部分の一部を公開し、開発者がその時点でエミュレートすることができなかった部分を定義していたことでした。 ECMAScript6は、従来、特定のオブジェクトの基本的な動作を定義するためにシンボルプロトタイプのプロパティを使用することによって、従来の内部ロジックをさらに露出させることでその伝統を継承しています。

ECMAScript6には、以前は内部のみの操作とみなされていたJavaScriptの一般的な動作を表す*よく知られているシンボル*と呼ばれる事前定義されたシンボルがあります。それぞれのよく知られているシンボルは`Symbol`オブジェクトのプロパティで表されます。たとえば、`Symbol.create`です。

よく知られているシンボルは次のとおりです。

*`Symbol.hasInstance`- オブジェクトの継承を判断するために`instanceof`によって使用されるメソッドです。
*`Symbol.isConcatSpreadable`- コレクションが`Array.prototype.concat()`にパラメータとして渡された場合、`Array.prototype.concat()`がコレクションの要素を平坦化することを示すブール値です。
*`Symbol.iterator`- イテレータを返すメソッドです。 (イテレータは第7章で説明しています)
*`Symbol.match`-`String.prototype.match()`が文字列を比較するために使うメソッド。
*`Symbol.replace`-`String.prototype.replace()`が部分文字列を置き換えるために使用するメソッドです。
*`Symbol.search`-`String.prototype.search()`が部分文字列を見つけるために使うメソッドです。
*`Symbol.species`- 派生オブジェクトを作成するためのコンストラクタです。 (派生オブジェクトについては第8章で説明しています)
*`Symbol.split`-`String.prototype.split()`が文字列を分割するために使うメソッド。
*`Symbol.toPrimitive`- オブジェクトのプリミティブな値表現を返すメソッドです。
*`Symbol.toStringTag`- オブジェクト記述を作成するために`Object.prototype.toString()`によって使用される文字列。
*`Symbol.unscopables`- プロパティが`with`ステートメントに含まれてはならないオブジェクトプロパティの名前であるオブジェクトです。

一般的に使用されているいくつかのよく知られているシンボルについては、以降のセクションで説明しますが、他のセクションでは正しいコンテキストでそれらを保つために説明します。

I>よく知られているシンボルで定義されたメソッドを上書きすると、通常のオブジェクトがエキゾチックなオブジェクトに変更されます。これは内部のデフォルト動作を変更するためです。その結果、コードに実際的な影響はなく、仕様がオブジェクトを記述する方法が変わります。

### Symbol.hasInstanceプロパティ

すべての関数には、指定されたオブジェクトがその関数のインスタンスであるかどうかを判断する`Symbol.hasInstance`メソッドがあります。メソッドは`Function.prototype`で定義され、すべての関数が`instanceof`プロパティのデフォルトの振る舞いを継承し、メソッドは書き換え不可能で、設定不可能で非数え込みで誤って上書きされないようにします。

`Symbol.hasInstance`メソッドは1つの引数、すなわちチェックする値を受け取ります。渡された値が関数のインスタンスであれば真を返します。`Symbol.hasInstance`の仕組みを理解するには、次のコードを考えてみましょう：

```js
obj instanceof Array;
```

このコードは次のものと同等です。

```js
Array[Symbol.hasInstance](obj);
```

本質的に、ECMAScript6はこのメソッド呼び出しの略式構文として`instanceof`演算子を再定義しました。 メソッド呼び出しが含まれているので、`instanceof`の動作方法を実際に変更することができます。

たとえば、オブジェクトをインスタンスとして要求しない関数を定義するとします。`Symbol.hasInstance`の戻り値を`false`にハードコーディングすることによって、そうすることができます：

```js
function MyObject() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});

let obj = new MyObject();

console.log(obj instanceof MyObject);       // false
```

書き込み不可能なプロパティを上書きするにはObject.defineProperty()を使用する必要があります。この例では、このメソッドを使用して、`Symbol.hasInstance`メソッドを新しい関数で上書きします。 新しい関数は常に`false`を返すので、`obj`が実際には`MyObject`クラスのインスタンスであっても、`instanceof`演算子は`Object.defineProperty()`の後に`false`を返します。

もちろん、値を調べて、任意の条件に基づいて値をインスタンスとみなす必要があるかどうかを判断することもできます。 たとえば、1〜100の値を持つ数値は、特殊な数値型のインスタンスと見なされることがあります。 そのような振る舞いを実現するには、次のようなコードを記述します。

```js
function SpecialNumber() {
    // empty
}

Object.defineProperty(SpecialNumber, Symbol.hasInstance, {
    value: function(v) {
        return (v instanceof Number) && (v >=1 && v <= 100);
    }
});

let two = new Number(2),
    zero = new Number(0);

console.log(two instanceof SpecialNumber);    // true
console.log(zero instanceof SpecialNumber);   // false
```

このコードは、値が`Number`のインスタンスであり、値が1から100までの場合は`true`を返す`Symbol.hasInstance`メソッドを定義しています。したがって、`SpecialNumber`は``SpecialNumber`関数と`two`変数の間に直接定義された関係はありません。`instanceof`への左オペランドは、非オブジェクトが`false`を単に返すようにするため、`Symbol.hasInstance`呼び出しをトリガーするオブジェクトでなければならないことに注意してください。

W>また、`Date`関数や`Error`関数のようなすべての組み込み関数の既定の`Symbol.hasInstance`プロパティを上書きすることもできます。 ただし、コードへの影響が予想外に混乱する可能性があるため、これはお勧めしません。`Symbol.hasInstance`はあなた自身の関数で、必要な時にだけ上書きすることをお勧めします。

### The Symbol.isConcatSpreadable Symbol

JavaScript配列には、2つの配列を連結するための`concat()`メソッドがあります。 この方法を使用する方法は次のとおりです。

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ]);

console.log(colors2.length);    // 4
console.log(colors2);           // ["red","green","blue","black"]
```

このコードは、新しい配列を`colors1`の終わりに連結し、`colors2`を作成します。これは、両方の配列のすべての項目を持つ単一の配列です。 しかし、`concat()`メソッドは、非配列引数を受け入れることもできます。その場合、それらの引数は単純に配列の最後に追加されます。 例えば：

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ], "brown");

console.log(colors2.length);    // 5
console.log(colors2);           // ["red","green","blue","black","brown"]
```

ここで余分な引数``brown "は`concat()`に渡され、`colors2`配列の5番目の項目になります。 なぜ配列引数は文字列引数とは違って扱われますか？ JavaScriptの仕様では、配列は自動的に個々のアイテムに分割され、他のすべてのタイプは自動的に分割されません。 ECMAScript6以前は、この動作を調整する方法がありませんでした。

`Symbol.isConcatSpreadable`プロパティは、オブジェクトが`length`プロパティと数値キーを持ち、数値プロパティ値が`concat()`呼び出しの結果に個別に追加されるべきであることを示すブール値です。 他のよく知られているシンボルとは異なり、このシンボルプロパティはデフォルトでは標準オブジェクトには表示されません。 代わりに、このシンボルは、`concat()`が特定のタイプのオブジェクトでどのように動作して、デフォルト動作を効果的に短絡するかを補う方法として利用できます。 次のように`concat()`コールで配列のように振る舞うような型を定義することができます：

```js
let collection = {
    0: "Hello",
    1: "world",
    length: 2,
    [Symbol.isConcatSpreadable]: true
};

let messages = [ "Hi" ].concat(collection);

console.log(messages.length);    // 3
console.log(messages);           // ["Hi","Hello","world"]
```

この例の`collection`オブジェクトは配列のように設定されています：`length`プロパティと2つの数値キーを持っています。`Symbol.isConcatSpreadable`プロパティは`true`に設定され、プロパティ値を個々の項目として配列に追加する必要があることを示します。`collection`が`concat()`メソッドに渡されると、結果の配列は``"Hi"`要素の後に別々の項目として``"Hello"`と``"world"`を持ちます。

I>配列のサブクラスで`Symbol.isConcatSpreadable`を`false`に設定して、`concat()`呼び出しで項目が区切られないようにすることもできます。サブクラス化については、第8章で説明します。

### Symbol.match、Symbol.replace、Symbol.search、Symbol.splitの各シンボル

JavaScriptでは文字列と正規表現は常に密接な関係にあります。特に、文字列型には、正規表現を引数として受け入れるいくつかのメソッドがあります。

*`match(regex)`- 与えられた文字列が正規表現と一致するかどうかを判定します
*`replace(regex、replacement)`- 正規表現のマッチを`replacement`に置き換えます。
*`search(regex)`- 文字列の中で正規表現のマッチを探します。
*`split(regex)`- 文字列を正規表現のマッチで配列に分割する

ECMAScript6より前では、これらのメソッドが正規表現とやりとりする方法は開発者から隠されていたため、開発者定義のオブジェクトを使って正規表現を模倣する方法はありませんでした。 ECMAScript6では、これらの4つのメソッドに対応する4つのシンボルが定義され、ネイティブビヘイビアを`RegExp`組み込みオブジェクトにアウトソースします。

`Symbol.match`、`Symbol.replace`、`Symbol.search`、そして`Symbol.split`の記号は正規表現の引数のメソッドを表します。これは`match()`メソッドの最初の引数で呼び出されます。`replace()`メソッド、`search()`メソッド、`split()`メソッドをそれぞれ呼びます。 4つのシンボルプロパティは、文字列メソッドが使用すべきデフォルトの実装として`RegExp.prototype`で定義されています。

これを知ることで、文字列メソッドで使用するオブジェクトを正規表現に似た方法で作成できます。これを行うには、コードで次のシンボル関数を使用できます。

*`Symbol.match`- 文字列の引数を受け取り、一致の配列を返す関数。一致するものが見つからない場合は`null`を返します。
*`Symbol.replace`- 文字列引数と置換文字列を受け取り、文字列を返す関数です。
*`Symbol.search`- 文字列の引数を受け取り、一致の数値インデックスを返す関数。一致するものが見つからなければ-1を返します。
*`Symbol.split`- 文字列の引数を受け取り、その文字列を含む配列を返します。

オブジェクトにこれらのプロパティを定義することで、正規表現を使用しないパターンマッチングを実装し、正規表現を必要とするメソッドで使用するオブジェクトを作成できます。実際にこれらのシンボルが表示されている例を次に示します。

```js
// effectively equivalent to /^.{10}$/
let hasLengthOf10 = {
    [Symbol.match]: function(value) {
        return value.length === 10 ? [value] : null;
    },
    [Symbol.replace]: function(value, replacement) {
        return value.length === 10 ? replacement : value;
    },
    [Symbol.search]: function(value) {
        return value.length === 10 ? 0 : -1;
    },
    [Symbol.split]: function(value) {
        return value.length === 10 ? ["", ""] : [value];
    }
};

let message1 = "Hello world",   // 11 characters
    message2 = "Hello John";    // 10 characters


let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10);

console.log(match1);            // null
console.log(match2);            // ["Hello John"]

let replace1 = message1.replace(hasLengthOf10, "Howdy!"),
    replace2 = message2.replace(hasLengthOf10, "Howdy!");

console.log(replace1);          // "Hello world"
console.log(replace2);          // "Howdy!"

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10);

console.log(search1);           // -1
console.log(search2);           // 0

let split1 = message1.split(hasLengthOf10),
    split2 = message2.split(hasLengthOf10);

console.log(split1);            // ["Hello world"]
console.log(split2);            // ["", ""]
```

`hasLengthOf10`オブジェクトは、文字列の長さが正確に10になるたびに一致する正規表現のように動作します。`hasLengthOf10`の4つのメソッドのそれぞれは、適切なシンボルを使用して実装され、2つの文字列の対応するメソッドが呼び出されます。最初の文字列`message1`は11文字であるため、一致しません。 2番目の文字列`message2`は10文字であるため、一致します。正規表現ではないにもかかわらず、`hasLengthOf10`は各文字列メソッドに渡され、追加のメソッドのために正しく使用されます。

これは簡単な例ですが、現在のところ正規表現で可能なより複雑なマッチを実行する能力は、カスタムパターンマッチャーの可能性を広げています。

### Symbol.toPrimitiveメソッド

JavaScriptは、特定の操作が適用されると、オブジェクトをプリミティブ値に暗黙的に変換しようとします。たとえば、文字列をdouble equals(`==`)演算子を使用してオブジェクトと比較すると、オブジェクトは比較前にプリミティブ値に変換されます。正確にどのプリミティブな値を使用すべきかは以前は内部操作でしたが、ECMAScript6は`Symbol.toPrimitive`メソッドを通してその値を公開しました(変更可能にしました)。

`Symbol.toPrimitive`メソッドは、各標準型のプロトタイプ上で定義され、オブジェクトがプリミティブに変換されるときに何が起こるべきかを規定します。プリミティブ変換が必要なときは、`Symbol.toPrimitive`が単一の引数で呼び出されます。これは、仕様では`hint`と呼ばれます。`hint`引数は、3つの文字列値のうちの1つです。`hint`が``number"`ならば、`Symbol.toPrimitive`は数値を返さなければなりません。`hint`が``string '`ならば、文字列が返されなければならず、``default'`ならば、操作はその型に関して優先度を持たない。

ほとんどの標準オブジェクトでは、数値モードには次のような動作が優先度順に適用されます。

1.`valueOf()`メソッドを呼び出し、結果がプリミティブな値であればそれを返します。
1.それ以外の場合は、`toString()`メソッドを呼び出し、結果がプリミティブな値であればそれを返します。
1.それ以外の場合は、エラーを投げます。

同様に、ほとんどの標準オブジェクトでは、文字列モードの動作には次の優先順位があります。

1.`toString()`メソッドを呼び出し、結果がプリミティブな値であればそれを返します。
1.それ以外の場合は、`valueOf()`メソッドを呼び出し、結果がプリミティブ値であればそれを返します。
1.それ以外の場合は、エラーを投げます。

多くの場合、標準オブジェクトはデフォルトモードを数値モードと同じように扱います(デフォルトモードを文字列モードと同じものとして扱う`Date`を除く)。`Symbol.toPrimitive`メソッドを定義することで、これらのデフォルトの強制動作を上書きすることができます。

I>デフォルトモードは、`==`演算子、`+`演算子、および`Date`コンストラクタに単一の引数を渡す場合にのみ使用されます。ほとんどの操作では、文字列モードまたは数値モードが使用されます。

デフォルトの変換動作をオーバーライドするには、`Symbol.toPrimitive`を使用し、その値として関数を割り当てます。例えば：

```js
function Temperature(degrees) {
    this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {

    switch (hint) {
        case "string":
            return this.degrees + "\u00b0"; // degrees symbol

        case "number":
            return this.degrees;

        case "default":
            return this.degrees + " degrees";
    }
};

let freezing = new Temperature(32);

console.log(freezing + "!");            // "32 degrees!"
console.log(freezing / 2);              // 16
console.log(String(freezing));          // "32°"
```

このスクリプトは`Temperature`コンストラクタを定義し、プロトタイプのデフォルトの`Symbol.toPrimitive`メソッドをオーバーライドします。`hint`引数が文字列、数字、またはデフォルトモード(JavaScriptエンジンによって`hint`引数が埋められる)を示すかどうかによって、異なる値が返されます。文字列モードでは、`Symbol.toPrimitive`メソッドはUnicode度記号で温度を返します。数値モードでは、数値だけを返し、デフォルトモードでは数値の後ろに`degrees`という語を追加します。

それぞれのログステートメントは、異なる`hint`引数値をトリガーします。`+`演算子は`hint`を``default``に設定することでデフォルトモードを起動し、`/`演算子は``hint``を``"number"`に設定して数値モードを起動し、`String()`関数は文字列`hint`を``string '`に設定することで、モードを有効にします。 3つのモードすべてに対して異なる値を返すことが可能です。デフォルトモードを文字列モードまたは数字モードと同じにする方がはるかに一般的です。

### Symbol.toStringTagシンボル

JavaScriptの最も興味深い問題の1つは、複数のグローバル実行環境が利用できることです。これは、ページにiframeが含まれている場合、ページとiframeにそれぞれ独自の実行環境があるため、Webブラウザで発生します。ほとんどの場合、これは問題ではありません。データが懸念する必要がほとんどない環境間を行き来できるためです。この問題は、オブジェクトが異なるオブジェクト間を渡された後に扱うオブジェクトのタイプを識別しようとするときに発生します。

この問題の標準的な例は、iframeから配列を含むページに配列を渡すことです。 ECMAScript6の用語では、iframeとそのページを含むページはそれぞれ、JavaScriptの実行環境である別の* realm *を表しています。各レルムには、グローバルオブジェクトのコピーを持つ独自のグローバルスコープがあります。配列が作成されるどの領域でも、配列は間違いありません。しかし、別の領域に渡されたとき、配列は異なる領域のコンストラクタで作成され、`Array`は現在の領域のコンストラクタを表すため、`instanceof Array`呼び出しは`false`を返します。

#### 識別問題の回避策

この問題に直面して、開発者はすぐに配列を識別する良い方法を見つけました。彼らは、オブジェクトの標準の`toString()`メソッドを呼び出すことによって、予測可能な文字列が常に返されることを発見しました。したがって、多くのJavaScriptライブラリは、次のような関数を含むようになりました。

```js
function isArray(value) {
    return Object.prototype.toString.call(value) === "[object Array]";
}

console.log(isArray([]));   // true
```

これは少し迂回して見えるかもしれませんが、すべてのブラウザで配列を識別するのには非常にうまく機能しました。配列の`toString()`メソッドは、オブジェクトに含まれる項目の文字列表現を返すため、オブジェクトを識別するのには役に立ちません。しかし、`Object.prototype`の`toString()`メソッドには奇妙なものがありました。返された結果に`[[Class]]`という内部定義の名前が含まれていました。開発者はオブジェクトに対してこのメ​​ソッドを使用して、JavaScript環境がオブジェクトのデータ型であると考える内容を取得できます。

開発者は、この動作を変更する方法がないため、ネイティブオブジェクトと開発者によって作成されたオブジェクトを区別するために同じアプローチを使用することが可能であることをすぐに認識しました。これの最も重要なケースは、ECMAScript5の`JSON`オブジェクトでした。

ECMAScript5以前では、多くの開発者がDouglas Crockfordの* json2.js *を使って、グローバルな`JSON`オブジェクトを作成していました。ブラウザーが`JSON`グローバルオブジェクトを実装し始めたので、グローバルな`JSON`がJavaScript環境自体によって提供されたのか、それとも他のライブラリーから提供されたのかがわかりました。`isArray()`関数で示したのと同じテクニックを使って、多くの開発者が次のような関数を作成しました：

```js
function supportsNativeJSON() {
    return typeof JSON !== "undefined" &&
        Object.prototype.toString.call(JSON) === "[object JSON]";
}
```

開発者がiframe境界を越えて配列を識別できるようにする`Object.prototype`の同じ特性は、`JSON`がネイティブな`JSON`オブジェクトであるかどうかを判断する方法も提供しました。 非ネイティブの`JSON`オブジェクトはネイティブバージョンが`[object JSON]`を返す間に`[object Object]`を返します。 このアプローチは、ネイティブオブジェクトを識別するための事実上の標準になりました。

#### ECMAScript6 Answer

ECMAScript6は、この動作を`Symbol.toStringTag`シンボルを通して再定義します。 このシンボルは、`Object.prototype.toString.call()`が呼び出されたときに生成される値を定義する各オブジェクトのプロパティを表します。 配列の場合、関数が返す値は``Array "`を`Symbol.toStringTag`プロパティに格納することで説明されます。

同様に、あなた自身のオブジェクトの`Symbol.toStringTag`値を定義することができます：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

let me = new Person("Nicholas");

console.log(me.toString());                         // "[object Person]"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

この例では、`Person.prototype`に`Symbol.toStringTag`プロパティが定義され、文字列表現を作成するためのデフォルトの動作を提供します。`Person.prototype`は`Object.prototype.toString()`メソッドを継承しているので、`Symbol.toStringTag`から返された値は`me.toString()`メソッドを呼び出すときにも使われます。 しかし、`Object.prototype.toString.call()`メソッドの使用に影響を与えずに、異なる動作を提供する独自の`toString()`メソッドを定義することはできます。 それがどのように見えるかは次のとおりです。

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

このコードは、`Person.prototype.toString()`を定義して`name`プロパティの値を返します。`Person`インスタンスは`Object.prototype.toString()`メソッドを継承しないので、`me.toString()`を呼び出すことで別の振る舞いをします。

I>他の指定がない限り、すべてのオブジェクトは`Object.prototype`から`Symbol.toStringTag`を継承します。 文字列``Object '`はデフォルトのプロパティ値です。

開発者が定義したオブジェクトの`Symbol.toStringTag`にどの値を使用できるかは制限されていません。 例えば、``Array``を`Symbol.toStringTag`プロパティの値として使用することを妨げるものはありません：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Array";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Array]"
```

`Object.prototype.toString()`の呼び出しの結果は、このコードでは``[object Array]``です。これは実際の配列から得られる結果と同じです。 これは、`Object.prototype.toString()`がオブジェクトの型を識別する完全に信頼できる方法ではなくなったという事実を強調しています。

ネイティブオブジェクトの文字列タグを変更することも可能です。 オブジェクトのプロトタイプの`Symbol.toStringTag`に以下のように代入するだけです：

```js
Array.prototype[Symbol.toStringTag] = "Magic";

let values = [];

console.log(Object.prototype.toString.call(values));    // "[object Magic]"
```

この例では`Symbol.toStringTag`を配列に上書きしていますが、`Object.prototype.toString()`を呼び出すと``[object Magic]」という結果になります。私はこのように組み込みのオブジェクトを変更しないことをお勧めしましたが、そうするのを禁じる言語は何もありません。

### Symbol.unscopablesシンボル

`with`ステートメントは、JavaScriptの最も議論の余地がある部分の1つです。もともと繰り返しタイピングを避けるように設計された`with`ステートメントは、後でコードを理解しにくくしたり、パフォーマンスに悪影響を及ぼしたり、エラーを起こしやすいようにするために批判的になりました。

その結果、strictモードでは`with`ステートメントは使用できません。その制限は、クラスとモジュールにも影響します。クラスとモジュールは、デフォルトでは厳密モードであり、オプトアウトはありません。

将来のコードは間違いなく`with`ステートメントを使用しませんが、ECMAScript6は下位互換性のために非厳密モードで`with`をサポートしていますので、`with`を使用するコードが適切に動作し続けるようにする方法を見つけなければなりません。

このタスクの複雑さを理解するには、次のコードを検討してください。

```js
let values = [1, 2, 3],
    colors = ["red", "green", "blue"],
    color = "black";

with(colors) {
    push(color);
    push(...values);
}

console.log(colors);    // ["red", "green", "blue", "black", 1, 2, 3]
```

この例では、`with`ステートメント内の`push()`への2回の呼び出しは、`with`ステートメントがローカルバインディングとして`push`を追加したので`colors.push()`と同等です。`color`リファレンスは、`values`リファレンスと同様に、`with`ステートメントの外で作成された変数を参照します。

しかし、ECMAScript6は配列に`values`メソッドを追加しました。 (`values`メソッドについては、第7章「反復子とジェネレータ」で詳しく説明しています)。ECMAScript6環境では、`with`ステートメント内の`values`参照はローカル変数`values`を呼び出すだけではなく、配列の`values`メソッドに渡されます。これが`Symbol.unscopables`シンボルが存在する理由です。

`Symbol.unscopables`シンボルは`Array.prototype`で`with`ステートメントの中でどのプロパティがバインディングを作成すべきでないかを示すために使われます。存在する場合、`Symbol.unscopables`は`with`ステートメントバインディングを省略するための識別子とブロックを強制するための値が`true`のキーを持つオブジェクトです。配列のデフォルトの`Symbol.unscopables`プロパティは次のとおりです：

```js
// built into ECMAScript6 by default
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null), {
    copyWithin: true,
    entries: true,
    fill: true,
    find: true,
    findIndex: true,
    keys: true,
    values: true
});
```

`Symbol.unscopables`オブジェクトには、`Object.create(null)`呼び出しで作成された`null`プロトタイプがあり、ECMAScript6の新しい配列メソッドがすべて含まれています(これらのメソッドについては、 7、 "Iterators and Generators"、第9章、 "配列")これらのメソッドのバインディングは`with`ステートメントの内部では作成されないので、古いコードを問題なくそのまま使用できます。

一般に、`with`ステートメントを使用し、コードベース内の既存のオブジェクトに変更を加えない限り、オブジェクトに`Symbol.unscopables`を定義する必要はありません。

## まとめ

シンボルは、JavaScriptの新しいタイプのプリミティブ値であり、シンボルを参照せずにアクセスできないプロパティを作成するために使用されます。

これらのプロパティは、プライベートではありませんが、誤って変更や上書きすることは難しく、開発者からの保護レベルが必要な機能に適しています。

シンボル値の識別を容易にするシンボルの説明を提供できます。同じ記述を使用して、コードの異なる部分で共有シンボルを使用できるようにするグローバルシンボルレジストリがあります。このように、複数の場所で同じ理由で同じシンボルを使用することができます。

`Object.keys()`や`Object.getOwnPropertyNames()`のようなメソッドはシンボルを返さないので、ECMAScript6では`Object.getOwnPropertySymbols()`という新しいメソッドが追加され、シンボルのプロパティを取得できます。`Object.defineProperty()`と`Object.defineProperties()`メソッドを呼び出すことで、シンボルのプロパティを変更することができます。

よく知られているシンボルは、以前の標準オブジェクトの内部専用機能を定義し、`Symbol.hasInstance`プロパティのようなグローバルに利用可能なシンボル定数を使用します。これらのシンボルは、仕様では`Symbol.`という接頭辞を使用し、開発者はさまざまな方法で標準オブジェクトの動作を変更できます。
