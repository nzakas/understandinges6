# 文字列と正規表現

Stringはプログラミングにおいて最も重要なデータ型の1つです。ほぼすべての高級言語に含まれており、効果的にそれらを使用することは、開発者にとって有用なプログラムを生成するための基本です。正規表現は、開発者が文字列を操作するための強力なツールです。これらの事実を念頭に置いて、ECMAScript6の開発者は、機能拡張および待望されていた仕様を追加して、文字列と正規表現を改善しました。この章では、両タイプの変更について説明します。

## より良いUnicodeサポート

ECMAScript6以前では、JavaScript文字列は16ビット文字エンコーディング(UTF-16)を中心に進化しました。各16ビットの連続した値は文字を表す*符号単位*です。`length`プロパティや`charAt() `メソッドのような、すべての文字列のプロパティとメソッドは、これらの16ビットの符号単位に基づいていました。もちろん、16ビットは任意の文字を含むのに十分でした。それは、Unicodeによって導入された拡張文字セットのおかげに他なりません。

### UTF-16コードポイント

Unicodeが世界のすべての文字にグローバルにユニークな識別子を提供するという目標のために、文字の長さを16ビットに制限することはできませんでした。*コードポイント*と呼ばれるこれらのグローバルに一意な識別子は、0から始まる単純な数字です。コードポイントは、文字コードと考えることができます。数字は文字を表します。文字エンコードでは、コードポイントを内部的に一貫した符号単位にエンコードする必要があります。 UTF-16の場合、コードポイントは多くの符号単位で構成できます。

UTF-16の最初の2 ^ 16 ^コードポイントは、単一の16ビット符号単位として表されます。この範囲は*基本多言語面*(BMP)と呼ばれます。それを超えるものはすべて、コードポイントをもはや16ビットで表現することができない*追加面*の1つとみなされます。 UTF-16は、1つのコードポイントが2つの16ビット符号単位で表される*サロゲートペアー*を導入することで、この問題を解決します。つまり、文字列内の任意の1文字は、合計16ビットを与えるBMP文字用の1つの符号単位、または追加面文字用の2単位の合計32ビットを与えることができます。

ECMAScript5では、すべての文字列演算が16ビット符号単位で動作します。つまり、この例のように、サロゲートペアーを含むUTF-16でエンコードされた文字列から予期しない結果を得ることができます。

```js
var text = "";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(text.charAt(0));        // ""
console.log(text.charAt(1));        // ""
console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
```

単一のUnicode文字 `""` はサロゲートペアを使用して表されているため、上記のJavaScript文字列操作では、文字列を2つの16ビット文字として扱います。つまり、

* `text`の`length`は1でなければならないときは2です。
* 1文字と一致する正規表現は2文字と考えるので失敗します。
* `charAt()`メソッドは有効な文字列を返すことができません。なぜなら、16ビットのいずれのセットも印刷可能な文字に対応しないからです。

`charCodeAt()`メソッドは、文字を正しく識別することもできません。各符号単位に適切な16ビットの数値を返しますが、これはECMAScript5の`text`の実際の値に最も近い数値です。

一方、ECMAScript6では、このような問題に対処するためにUTF-16文字列エンコーディングが強制されます。この文字列エンコーディングに基づいて文字列の操作を標準化することにより、JavaScriptがサロゲートペアによって特別に機能するように設計された機能をサポートできるようになります。このセクションの残りの部分では、その機能の重要な例をいくつか説明します。

### codePointAt()メソッド

UTF-16を完全にサポートするために追加されたECMAScript6の1つのメソッドは`codePointAt()`メソッドです。このメソッドは、文字列内の指定された位置にマッピングされるUnicodeコードポイントを取得します。このメソッドは文字位置ではなく符号単位の位置を受け取り、下記のサンプルの`console.log()`が示すように、整数値を返します：

```js
var text = "a";

console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
console.log(text.charCodeAt(2));    // 97

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 57271
console.log(text.codePointAt(2));   // 97
```

`codePointAt()`メソッドは、非BMP文字で動作しない限り、`charCodeAt()`メソッドと同じ値を返します。`text`の最初の文字はBMPではないので、`length`プロパティが2ではなく3であることを意味する2つの符号単位で構成されます。`charCodeAt()`メソッドは、位置0の最初の符号単位のみを返します。`codePointAt()`は、コードポイントが複数の符号単位にまたがっていても、完全なコードポイントを返します。どちらのメソッドも、位置1(最初の文字の2番目の符号単位)と2(`"a"`)に同じ値を返します。

文字に対して`codePointAt()`メソッドを呼び出すことは、その文字が1つまたは2つのコードポイントで表されるかどうかを判定する最も簡単な方法です。下記のような関数で確認することができます。

```js
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit(""));         // true
console.log(is32Bit("a"));          // false
```

16ビット文字の上限は16進数で`FFFF`と表されているため、その数を超えるコードポイントは2つの符号単位で表されなければならず、合計32ビットです。

### String.fromCodePoint()メソッド

ECMAScriptが何かを行う方法を提供する場合、ECMAScriptはその逆を行う方法も提供する傾向があります。`codePointAt()`を使うと、文字列中の文字のコードポイントを取得することができ、`String.fromCodePoint()`は与えられたコードポイントから単一文字の文字列を生成することができます。例えば、

```js
console.log(String.fromCodePoint(134071));  // ""
```

`String.fromCharCode()`メソッドのより完全なバージョンとして`String.fromCodePoint()`を考えてみましょう。両方とも、BMPのすべての文字に対して同じ結果が得られます。BMP以外の文字のコードポイントを渡すときにのみ違いがあります。

### normalize()メソッド

Unicodeのもう1つの面白い点は、ソートやその他の比較演算の目的で、異なる文字列が同等と見なされることです。これらの関係を定義するには2つの方法があります。まず、*正準等価*は、コードポイントの2つの連続した値がすべての点で交換可能であると考えられることを意味します。例えば、2つの文字の組み合わせは、標準的に1つの文字に等しくすることができます。2番目の関係は*互換等価*です。コードポイントの2つの互換性のある連続した値は異なるように見えますが、特定の状況では互換的に使用できます。

これらの関係のため、基本的に同じテキストを表す2つの文字列は、異なるコードポイント連続した値を含むことができます。たとえば、文字"æ"と2文字の文字列"ae"は互換的に使用できますが、何らかの方法で正規化しない限り、厳密には同等ではありません。

ECMAScript6は文字列に`normalize()`メソッドを与えることでUnicode正規化形式をサポートしています。このメソッドは、次のUnicode正規化形式のいずれかを示す単一の文字列パラメータをオプションとして受け入れます。

* 正規化形式C(NFC, デフォルト)
* 正規化形式D(NFD
* 正規化形式KC(NFKC)
* 正規化形式KD(NFKD)

これらの4つのフォームの違いを説明することは、この本の範囲を超えています。文字列を比較するときは、両方の文字列を同じ形式に正規化する必要があります。例えば、

```js
var normalized = values.map(function(text) {
    return text.normalize();
});

normalized.sort(function(first, second) {
    if (first < second) {
        return -1;
    } else if (first === second) {
        return 0;
    } else {
        return 1;
    }
});
```

このコードは `values`配列の文字列を正規化された形式に変換して、配列を適切にソートすることができます。次のように、コンパレータの一部として`normalize()`を呼び出すことによって元の配列をソートすることもできます：

```js
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
```

繰り返しますが、このコードに最も重要なことは、`first`と`second`の両方が同じように正規化されるということです。これらの例ではデフォルトのNFCを使用していますが、次のように簡単に指定することもできます。

```js
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
```

前にUnicodeの正規化について心配していなかったなら、おそらくこのメソッドにあまり使われていないでしょう。しかし、国際化されたアプリケーションで作業する場合、`normalize()`メソッドが役立つことは間違いありません。

ただし、これらのメソッドだけが、ECMAScript6がUnicode文字列を扱うために提供する改善点というわけではありません。標準仕様には、2つの有用なシンタックスも追加されています。

### 正規表現uフラグ

正規表現を使用して、多くの一般的な文字列操作を実行できます。しかし、覚えておいてください。正規表現は16ビットの符号単位を想定しています。それぞれの符号単位は1文字です。この問題に対処するために、ECMAScript6はUnicodeを表す正規表現のための`u`フラグを定義します。

#### uフラグ

正規表現に`u`フラグがセットされていると、符号単位ではなく文字を扱うモードに切り替わります。これは、正規表現がもはや文字列のサロゲートペアについて混乱しなくなり、期待どおりに動作することを意味します。たとえば、次のコードを考えてみましょう。

```js
var text = "";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```

正規表現 `/^.$/`は、すべての入力文字列を1文字でマッチします。`u`フラグなしで使用すると、この正規表現は符号単位でマッチするため、日本語の文字(2つの符号単位で表されます)は正規表現とマッチしません。`u`フラグとともに使用すると、正規表現は符号単位の代わりに文字を比較するので、日本語文字はマッチします。

#### コードポイントを数える

残念ながら、ECMAScript6では、文字列に含まれるコードポイントの数を判定するメソッドは追加されませんが、`u`フラグを使用すると、正規表現を使用して次のように調べることができます。

```js
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("bc"));   // 3
```

この例では、Unicodeを有効にしてグローバルに適用される正規表現を使用して、`[\s\S]`を使ってパターンが改行に一致することを確認する`match()`を呼び出して、少なくとも1つでもマッチしたとき、`result`はマッチした配列を含んでいます。そのため配列の長さは文字列のコードポイントの数です。Unicodeでは、文字列`"abc"`と`"bc"`の両方に3つの文字列があるので、配列の長さは3です。

W> この方法は有効ですが、特に長い文字列に適用すると、それほど高速ではありません。文字列イテレータ(第8章で説明)も使用できます。可能であれば、コードポイントのカウントを最小限に抑えるようにしてください。

#### uフラグのサポートを策定

`u`フラグはシンタックスの変更であるため、ECMAScript6と互換性のないJavaScriptエンジンでそれを使用しようとするとシンタックスエラーが発生します。下記のような関数で、`u`フラグがサポートされているかどうかを安全に判定することができます。

```js
function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        return true;
    } catch (ex) {
        return false;
    }
}
```

この関数は、`RegExp`コンストラクタを使用して`u`フラグを引数として渡します。このシンタックスは古いJavaScriptエンジンでも有効ですが、`u`がサポートされていない場合、コンストラクタはエラーを投げます。

I> あなたのコードが古いJavaScriptエンジンで動作する必要がある場合は、`u`フラグを使用するときは常に`RegExp`コンストラクタを使用してください。これにより、シンタックスエラーを防ぎ、実行を中断することなく`u`フラグを検出して使用することができます。

## その他の文字列の変更

JavaScriptの文字列は、他の言語の同様の機能と比較して遅れています。たとえば、ECMAScript5では文字列が最終的に`trim()`メソッドを取得し、ECMAScript6はJavaScriptの機能を拡張して新しい機能で文字列をパースし続けました。

### 部分文字列を識別するメソッド

開発者は、JavaScriptが最初に導入されて以来、`indexOf()`メソッドを使用して他の文字列内の文字列を識別してきました。 ECMAScript6には、次の3つの方法があります。

* `includes()`メソッドは、指定されたテキストが文字列内のどこにあっても真を返します。そうでない場合はfalseを返します。
* `startsWith()`メソッドは、指定されたテキストが文字列の先頭にある場合にtrueを返します。そうでない場合はfalseを返します。
* `endsWith()`メソッドは、指定されたテキストが文字列の最後にある場合にtrueを返します。そうでない場合はfalseを返します。

各メソッドは、検索するテキストとオプションのインデックスの2つの引数を受け取ります。2番目の引数が与えられたとき、`includes()`と`startsWith()`はそのインデックスからのマッチを開始します。endsWith()は2番目の引数からマッチを開始します。2番目の引数が省略された場合、`includes()`と`startsWith()`は文字列の先頭から検索され、`endsWith()`は最後から検索されます。実際には、2番目の引数は検索される文字列の量を最小限に抑えます。これらの3つの方法を実際に使用している例をいくつか示します。

```js
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.includes("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.includes("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.includes("o", 8));          // false
```

最初の6つの呼び出しには2番目のパラメータが含まれていないため、必要に応じて文字列全体を検索します。最後の3回の呼び出しは、文字列の一部のみをチェックします。`msg.startsWith("o", 4)`を呼び出すと、`msg`文字列のindex=4の("Hello"の"o")を見ることで一致を開始します。`msg.endsWith("o", 8)`の呼び出しは、index=0からの検索を開始し、index=7まで検索します。これは "world"の"o"です。`msg.includes("o", 8)`の呼び出しは、"world"の"r"であるindex=8からのマッチを開始します。

これらの3つのメソッドでは、部分文字列の存在をより簡単に識別できますが、それぞれが真偽値を返すだけです。ある文字列の実際の位置を別の文字列内で見つける必要がある場合は、`indexOf()`または`lastIndexOf()`メソッドを使用してください。

W> `startsWith()`, `endsWith()`, `includes()`メソッドは文字列の代わりに正規表現を渡すとエラーを投げます。これは`indexOf()`や`lastIndexOf()`とは対照的です。正規表現の引数を文字列に変換し、その文字列を検索します。

### repeat()メソッド

また、ECMAScript6は文字列に`repeat()`メソッドを追加します。このメソッドは、文字列を引数として繰り返す回数を受け付けます。指定された回数繰り返した元の文字列を含む新しい文字列を返します。例えば：

```js
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
```

このメソッドは、とりわけ有用な機能であり、とりわけテキストを操作するときに役立ちます。インデントの階層を生成する必要のあるフォーマットでは、特に役立ちます。

```js
// indent using a specified number of spaces
var indent = " ".repeat(4),
    indentLevel = 0;

// whenever you increase the indent
var newIndent = indent.repeat(++indentLevel);
```

最初の`repeat()`呼び出しは4つのスペースの文字列を生成し、`indentLevel`変数はインデントの階層を追跡します。次に、インクリメントされた`indentLevel`を使って`repeat()`を呼び出して、スペースの数を変更することができます。

ECMAScript6は、特定のカテゴリに適合しない正規表現機能にいくつかの有用な変更を加えます。次のセクションでは、いくつかを強調しています。

## その他の正規表現の変更

正規表現は、JavaScriptで文字列を扱う際に重要な機能であり、言語の多くの部分と同様、最近のバージョンではあまり変更されていません。しかし、ECMAScript6では、文字列の更新に合わせて正規表現をいくつか改善しています。

### 正規表現yフラグ

ECMAScript6は、正規表現の独自の拡張機能としてFirefoxで実装された後、`y`フラグを標準化しました。`y`フラグは、正規表現検索の`sticky`プロパティに影響を与え、正規表現のlastIndexプロパティで指定された位置にある文字列内で一致する文字を検索するように指示します。その場所に一致するものがなければ、正規表現は一致を停止します。これがどのように動作するかは下記のコードで確認できます。

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

pattern.lastIndex = 1;
globalPattern.lastIndex = 1;
stickyPattern.lastIndex = 1;

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // Error! stickyResult is null
```

この例には3つの正規表現があります。`pattern`の式はフラグを持たず、`globalPattern`の式は`g`フラグを使い、`stickyPattern`の式は`y`フラグを使います。`console.log()`呼び出しの最初の3つは、3つの正規表現はすべて、`"hello1 "`を最後にスペースを入れて返すべきです。

その後、3つのパターンすべてでlastIndexプロパティが1に変更されます。つまり、正規表現はすべての文字の2番目の文字からのマッチングを開始する必要があります。フラグを持たない正規表現は`lastIndex`への変更を完全に無視し、`"hello1 "`と事実上一致します。`g`フラグが付いた正規表現は、文字列(`"e"`)の2番目の文字から前方を検索しているので、`"hello2 "`にマッチします。sticky正規表現は2番目の文字から始まるものと一致しないので、`stickyResult`は`null`です。

stickeyフラグは、操作が実行されるたびにlastIndexで最後に一致した次の文字のインデックスを保存します。演算の結果が一致しない場合、lastIndexは0に設定されます。グローバルフラグは、次のように同様に動作します。

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 7
console.log(stickyPattern.lastIndex);   // 7

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // "hello2 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 14
console.log(stickyPattern.lastIndex);   // 14
```

`lastIndex`の値は、`stickyPattern`変数と`globalPattern`変数の両方に対して、`exec()`の最初の呼び出しの後で7に、2番目の呼び出しの後で14に変わります。

覚えておきたいのは、stickeyフラグについてさらに2つのわかりにくい仕様があります。

1. `lastIndex`プロパティは、`exec()`や`test()`メソッドのように、正規表現オブジェクトに存在するメソッドを呼び出すときにのみ有効です。正規表現を`match()`のような文字列メソッドに渡しても、スティッキーな動作は起こりません。
1. 文字列の先頭に一致する`^`文字を使用する場合、スティッキー正規表現は文字列の先頭(または複数行モードの行の先頭)からのみ一致します。`lastIndex`は0ですが、`^`はスティッキーな正規表現を、スティッキーでないものと変わらないものにします。`lastIndex`がシングルラインモードの文字列の先頭や複数行モードの行頭に対応しない場合、スティッキー正規表現は決して一致しません。

他の正規表現フラグと同様に、プロパティを使って`y`の存在を検出することができます。この場合、次のように`sticky`プロパティをチェックします：

```js
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```

`sticky`プロパティは、stickeyフラグが存在する場合はtrueに設定され、stickeyフラグが存在しない場合はfalseに設定されます。`sticky`プロパティは、フラグの存在に基づいて読み取り専用であり、コード内で変更することはできません。

`u`フラグと同様に、`y`フラグはシンタックスの変更であるため、古いJavaScriptエンジンではシンタックスエラーが発生します。サポートしているかどうかは下記のように確認できます。

```js
function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        return true;
    } catch (ex) {
        return false;
    }
}
```

`u`チェックと同様に、`y`フラグを持つ正規表現を生成できない場合はfalseを返します。`u`との最後の類似点として、古いJavaScriptエンジンで実行されるコードに`y`を使う必要がある場合、正規表現を定義する際に`RegExp`コンストラクタを使ってシンタックスエラーを避けるようにしてください。

### 正規表現を複製する

ECMAScriptでは、正規表現を`RegExp`コンストラクタに次のように渡してコピーすることができます：

```js
var re1 = /ab/i,
    re2 = new RegExp(re1);
```

`re2`変数は単に`re1`変数のコピーです。しかし、正規表現のフラグを指定する`RegExp`コンストラクタに2番目の引数を指定した場合、この例のようにコードは機能しません：

```js
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");
```

ECMAScript5環境でこのコードを実行すると、最初の引数が正規表現の場合に2番目の引数を使用できないというエラーが発生します。 ECMAScript6では、この動作が変更され、2番目の引数が許可され、最初の引数にあるフラグがすべて上書きされます。例えば、

```js
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");


console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false

```

このコードでは、`re1`は大文字小文字を区別しない`i`フラグを持ち、`re2`は大域的`g`フラグのみを持っています。`RegExp`コンストラクタはパターンを`re1`から複製し、`i`フラグのために`g`フラグを置き換えました。第2引数がなければ、`re2`は`re1`と同じフラグを持ちます。

### `flags`プロパティ

新しいフラグの追加とフラグの操作方法の変更に加えて、ECMAScript6はそれらに関連付けられたプロパティを追加しました。ECMAScript5では、`source`プロパティを使って正規表現のテキストを得ることができますが、フラグ文字列を取得するには、以下のように`toString()`メソッドの出力をパースする必要があります：

```js
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() is "/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g"
```

正規表現を文字列に変換し、最後の`/`の後にある文字を返します。これらの文字はフラグです。

ECMAScript6は、`source`プロパティに対して`flags`プロパティを追加することで、フラグの取得を容易にします。両方のプロパティは、ゲッターのみが割り当てられたプロトタイプアクセサプロパティで、ReadOnlyになっています。`flags`プロパティはデバッグと継承の目的のために正規表現を簡単に検査します。

ECMAScript6に後で追加された`flags`プロパティは、正規表現に適用されたフラグの文字列表現を返します。例えば、

```js
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
```

これは、`re`のすべてのフラグを取り出し、`toString() `よりもはるかに少ないコード行でコンソールに出力します。`source`と`flags`を一緒に使うことで、正規表現文字列を直接パースすることなく必要な正規表現を抽出することができます。

この章で扱ってきた文字列や正規表現の変更は確かに強力ですが、ECMAScript6は文字列よりもはるかに強大な力を手にしました。それは、文字列をより柔軟にするリテラルです。

## テンプレートリテラル

JavaScriptの文字列は、他の言語の文字列と比較して機能が制限されていました。たとえば、ECMAScript6までは、この章で説明しているメソッドが文字列に存在しないため、文字列の連結は可能な限り簡単です。開発者がより複雑な問題を解決できるように、ECMAScript6の*テンプレートリテラル*は、ECMAScript5以前のソリューションより安全な方法で文字列を扱うためのドメイン固有言語(DSL)のシンタックスを提供します。 (DSLは、JavaScriptなどの汎用言語とは対照的に、特定の狭い目的のために設計されたプログラミング言語です)。ECMAScript wikiでは、下記のように記載されています。 [template literal strawman](http://wiki.ecmascript.org/doku.php?id=harmony:quasis)

> このスキームは、XSSやSQL Injectionなどの攻撃に対して耐性を持つ他の言語のコンテンツを容易に生成、参照、操作するDSLをライブラリに提供できるように、シンタックスシュガーでECMAScriptシンタックスを拡張します。

実際には、テンプレートリテラルは、JavaScriptがECMAScript5では対応していないと言われてきた次の機能に対するECMAScript6の答えです。

* **複数行の文字列**複数行の文字列の正式な概念です。
* **基本的な文字列フォーマット**文字列の一部を変数に含まれる値に置き換えることができます。
* **HTMLエスケープ** HTMLに挿入するのが安全なように文字列を変換する機能。

テンプレートリテラルは、JavaScriptの既存の文字列にさらに多くの機能を追加しようとするのではなく、これらの問題を解決する全く新しいアプローチです。

### 基本シンタックス

最も単純なテンプレートリテラルは、ダブルクォートやシングルクォートの代わりにバッククォート(`` ` ``)で区切られた通常の文字列のように動作します。たとえば、次の点を考慮してください。

```js
let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12
```

このコードは、変数`message`に通常のJavaScript文字列が含まれていることを示しています。テンプレートリテラルシンタックスは文字列値を生成するために使用され、`message`変数に割り当てられます。

文字列中でバッククォートを使いたいのであれば、下記の`message`変数のようにバックスラッシュ(`\`)でエスケープしてください。

```js
let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14
```

テンプレートリテラル内で二重引用符または一重引用符のいずれかをエスケープする必要はありません。

### 複数行の文字列

JavaScriptの開発者は、言語の最初のバージョン以降、複数行の文字列を生成する方法を求めていました。しかし、二重引用符または一重引用符を使用する場合、文字列は完全に単一の行に含まれなければなりません。

#### Pre-ECMAScript6の回避策

長年にわたるシンタックスのバグのおかげで、JavaScriptには回避策があります。改行の前にバックスラッシュ(`\`)がある場合は、複数行の文字列を生成できます。下記がサンプルです。

```js
var message = "Multiline \
string";

console.log(message);       // "Multiline string"
```

バックスラッシュは改行ではなく文字列の継続として扱われるため、`message`文字列にはコンソールに出力する際に改行がありません。出力に改行を表示するには、手動で含める必要があります。

```js
var message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string"
```

主要なJavaScriptエンジンでは。2つの別々の行に`Multiline String`を表示しなければなりませんが、その振る舞いはバグとして定義され、避けるべきだと言われてきました。

他のpre-ECMAScript6では、以下のような配列や文字列の連結に依存していました。

```js
var message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
    "string";
```

JavaScriptの複数行文字列の不便さを回避するために開発者がとった方法は、重要な示唆を残しました。

#### 簡単な方法で複数の文字列

ECMAScript6のテンプレートリテラルは、特別なシンタックスがないため、複数行の文字列を簡単にします。あなたが望む場所に改行を入れるだけで結果に現れます。例えば：

```js
let message = `Multiline
string`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

バッククォート内のすべての空白は文字列の一部です。インデントには注意してください。例えば、

```js
let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 31
```

このコードでは、テンプレートリテラルの2行目の前のすべての空白は、文字列自体の一部とみなされます。適切なインデントでテキスト行を生成することが重要な場合は、次のように、複数行のテンプレートリテラルの最初の行に何も残さず、その後にインデントを追加してください。

```js
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

このコードは、最初の行でテンプレートリテラルを開始しますが、2番目の行まではテキストがありません。 HTMLタグは正しく見えるようインデントされ、`trim()`メソッドが呼び出されて最初の空行が削除されます。

A> 必要に応じて、テンプレートリテラルで`\n`を使って、改行を挿入する場所を指定することもできます：
A> {:lang="js"}
A> ~~~~~~~~
A>
A> let message = `Multiline\nstring`;
A>
A> console.log(message);           // "Multiline
A>                                 //  string"
A> console.log(message.length);    // 16
A> ~~~~~~~~

### 置換を行う

この時点で、テンプレートリテラルは通常のJavaScript文字列のように見やすいものに見えるかもしれません。 2つの間の実際の違いは、テンプレートのリテラル*置換*にあります。置換により、テンプレートリテラル内に有効なJavaScript式を埋め込み、その結果を文字列の一部として出力することができます。

JavaScriptの式を内部に含めるには、`${`と`}`で囲みます。最も単純な置換は、ローカル変数を次のように結果の文字列に直接埋め込むことです。

```js
let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas."
```

置換`${name}`はローカル変数`name`にアクセスして`name`を`message`文字列に挿入します。`message`変数は置換の結果を直ちに保持します。

I> テンプレートリテラルは、定義されているスコープ内でアクセス可能な変数にアクセスできます。テンプレートリテラルで宣言されていない変数を使用しようとすると、strictモードとnon-strictモードの両方でエラーがスローされます。

すべての置換がJavaScript式なので、単純な変数名だけではなく、代用することもできます。計算、関数呼び出しなどを簡単に埋め込むことができます。例えば：

```js
let count = 10,
    price = 0.25,
    message = `${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

このコードは、テンプレートリテラルの一部として計算を実行します。変数`count`と`price`は、結果を得るために一緒に乗算され、`.toFixed()`を使って小数点以下2桁にフォーマットされます。2番目の置換の前のドル記号は、開始中括弧が続かないため、そのまま出力されます。

テンプレートリテラルはJavaScript式でもあります。つまり、次の例のように、テンプレートリテラルを別のテンプレートリテラルの内側に配置できます。

```js
let name = "Nicholas",
    message = `Hello, ${
        `my name is ${ name }`
    }.`;

console.log(message);        // "Hello, my name is Nicholas."
```

この例では、最初のテンプレートリテラルを最初のテンプレートリテラルの中にネストします。最初の`${`の後に別のテンプレートリテラルが始まります。 2番目の`${`は内部のテンプレートリテラルの中に埋め込まれた式の始まりを示します。その式は変数`name`であり、結果に挿入されます。

### タグ付きテンプレート

テンプレートリテラルが複数行の文字列を生成し、連結せずに文字列に値を挿入する方法を見てきました。しかし、テンプレートリテラルの本当の力は、タグ付きテンプレートに由来します。*tagテンプレート*はテンプレートリテラルに対して変換を実行し、最終的な文字列値を返します。このtagは、テンプレートの最初、最初の`` ` ``文字の直前に指定します。

```js
let message = tag`Hello world`;
```

この例では、`tag`は`` `Hello world` ``テンプレートリテラルに適用するテンプレートタグです。

#### タグの定義

*tag*は、単に処理されたテンプレートリテラルデータで呼び出される関数です。このタグは、テンプレートリテラルに関するデータを個別に受け取り、結合して結果を生成する必要があります。最初の引数は、JavaScriptによって解釈されるリテラル文字列を含む配列です。各後続の引数は、各置換の解釈された値です。

タグ関数は、通常、以下のようにrest引数を使用して定義され、データを扱いやすくします。

```js
function tag(literals, ...substitutions) {
    // return a string
}
```

何がtagに渡されるのかを理解するには、次の点を考慮してください。

```js
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
```

`passthru()`という関数があった場合、その関数は3つの引数を受け取ります。まず、以下の要素を含む`literal`配列を取得します。

* 最初の置換(`""`)の前の空の文字列
* 最初の置換後の文字列と2番目の文字列(`" items costs $"`)の前の文字列
* 2回目の置換後の文字列(`"."`)

次の引数は`10`で、これは`count`変数の解釈された値です。これは`substitutions`配列の最初の要素になります。最後の引数は`"2.50"`で、`(count * price).toFixed(2)`のための解釈された値であり、`substitutions`配列の2番目の要素です。

`literal`の最初の項目は空文字列です。これは、`literal[literal.length - 1]`が常に文字列の終わりであるように、`literal[0]`が常に文字列の先頭であることを保証します。literalより常に置換が1つ少なくなります。つまり、`substitutions.length === literals.length - 1`という表現は常に真です。

このパターンを使うと、`literal`と`substitute`配列を合成して結果の文字列を作ることができます。`リテラル`の最初の項目が最初に来て、`置換`の最初の項目が次に、そして文字列が完成するまで続きます。たとえば、次の2つの配列の値を交互に入れて、テンプレートリテラルのデフォルトの動作を模倣することができます。

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

この例では、デフォルトのテンプレートリテラル動作と同じ変換を実行する`passthru`タグを定義しています。唯一のやり方は`literalals.length`ではなく`substitutions.length`を使って、`substitutions`配列の終わりを過ぎてしまうことを避けることです。これは、ECMAScript6では`リテラル`と`置換`の関係が明確に定義されているためです。

I> `substitutions`に含まれる値は必ずしも文字列ではありません。前の例のように式が数値に評価される場合、数値が渡されます。結果にそのような値を出力する方法を決定することは、tagの役割の一つです。

#### テンプレートリテラルの生値を使う

テンプレートタグは生の文字列情報にもアクセスできます。文字情報は、主に文字エスケープにアクセスして文字に変換する前にアクセスすることを意味します。生の文字列値を扱う最も簡単な方法は、組み込みの`String.raw()`タグを使うことです。例えば。

```js
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring"
```

このコードでは、`message1`の`\n`は改行として解釈され、`message2`の`\n`は未加工の形式の`"\\n"`(スラッシュと`n`)。このような生の文字列情報を取得することで、必要に応じてより複雑な処理が可能になります。

生の文字列情報もテンプレートタグに渡されます。タグ関数の最初の引数は、`raw`という余分なプロパティを持つ配列です。`raw`プロパティは、各リテラル値の生の等価物を含む配列です。たとえば、`literal[0]`の値は、生の文字列情報を含む`literal.raw[0]`と等価です。それを知って、あなたは次のコードを使って`String.raw()`を模倣することができます：

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

これは`literal`の代わりに`literalals.raw`を使って文字列resultを出力します。つまり、Unicodeコードポイントエスケープを含むすべての文字エスケープは、そのままの形で返される必要があります。生の文字列は、エスケープ文字を含める必要があるコードを含む文字列を出力する場合に役立ちます(たとえば、コードについてのドキュメントを生成する場合は、実際のコードをそのまま出力することができます) 。

## まとめ

完全なUnicodeサポートにより、JavaScriptは論理的な方法でUTF-16文字を扱うことができます。`codePointAt()`と`String.fromCodePoint()`を使ってコードポイントと文字をやりとりする能力は、文字列操作にとって重要なステップです。正規表現`u`フラグを追加することで、16ビット文字ではなくコードポイントでの操作が可能になり`normalize()`メソッドではより適切な文字列比較が可能になります。

ECMAScript6では、文字列を操作するための新しいメソッドも追加され、親ストリング内の位置に関係なく、部分文字列をより簡単に識別することができます。さらに多くの機能が正規表現に追加されました。

テンプレートリテラルはECMAScript6に重要な追加機能であり、文字列の生成を容易にするドメイン固有言語(DSL)を生成できます。変数をテンプレートリテラルに直接埋め込むことは、開発者が長い文字列を変数で構成するための文字列連結よりも安全なツールであることを意味します。

複数行文字列の組み込みサポートにより、テンプレートリテラルは、この能力を持っていない通常のJavaScript文字列よりも便利なアップグレードになります。テンプレートリテラルの中に改行を直接入れることもできますし、`\n`やその他のエスケープシーケンスを使用することもできます。

テンプレートタグは、DSLを生成するためのこの機能の最も重要な部分です。タグは、引数としてテンプレートリテラルを受け取る関数です。そのデータを使用して、適切な文字列値を返すことができます。提供されるデータには、リテラル、未処理の等価物、および任意の置換値が含まれます。これらの情報を使用して、タグの正しい出力を判定することができます。
