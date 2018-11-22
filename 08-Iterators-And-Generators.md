# イテレータとジェネレータ

多くのプログラミング言語は、コレクション内の位置を追跡するための変数の初期化を必要とする`for`ループによるデータの反復処理から、コレクション内の次の項目をプログラムで返す反復子オブジェクトへの移行に移行しました。 イテレータはデータの集まりを扱うのを容易にし、ECMAScript6はイテレータをJavaScriptに追加します。 新しい配列メソッドや新しいタイプのコレクション(セットやマップなど)と組み合わせると、イテレーターは効率的なデータ処理の鍵となり、言語の多くの部分でそれらを見つけることができます。 イテレータと連動する新しい`for-of`ループがあり、スプレッド(`...`)演算子はイテレータを使用し、イテレータは非同期プログラミングを容易にします。

この章では、イテレータの多くの用途について説明しますが、まずイテレータがJavaScriptに追加された理由を理解することが重要です。

## ループ問題

JavaScriptでプログラミングしたことがある人は、おそらく次のようなコードを書いているはずです。

```js
var colors = ["red", "green", "blue"];

for (var i = 0, len = colors.length; i < len; i++) {
    console.log(colors[i]);
}
```

この標準の`for`ループは`i`変数を使ってインデックスを`colors`配列にトラッキングします。`i`の値は`i`が配列​​の長さ(`len`に格納されている)よりも大きくなければ、ループが実行されるたびにインクリメントされます。

このループはかなり簡単ですが、ループをネストすると複雑になり、複数の変数を追跡する必要があります。追加の複雑さはエラーにつながる可能性があり、`for`ループの定型的な性質は、同様のコードが複数の場所で書かれているので、より多くのエラーに役立ちます。イテレータはその問題を解決するためのものです。

## イテレータとは何ですか？

反復子は、反復のために設計された特定のインターフェースを持つオブジェクトに過ぎません。すべての反復子オブジェクトには結果オブジェクトを返す`next()`メソッドがあります。結果オブジェクトには、次の値である`value`と返される値がなくなると真となる`true`という2つのプロパティがあります。イテレータは、値のコレクション内の場所への内部ポインタを保持し、`next()`メソッドを呼び出すたびに、次の適切な値を返します。

最後の値が返された後に`next()`を呼び出すと、メソッドは`done`を`true`として返し、`value`はイテレータの戻り値*を返します。その戻り値はデータセットの一部ではなく、関連するデータの最後の部分です。そうしたデータが存在しない場合は`undefined`です。イテレータの戻り値は、呼び出し元に情報を渡す最後の方法であるという点で、関数の戻り値に似ています。

これを念頭において、ECMAScript5を使用したイテレータの作成はかなり簡単です。

```js
function createIterator(items) {

    var i = 0;

    return {
        next: function() {

            var done = (i >= items.length);
            var value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };

        }
    };
}

var iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// for all further calls
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

`createIterator()`関数は`next()`メソッドを持つオブジェクトを返します。メソッドが呼び出されるたびに、`items`配列の次の値が`value`として返されます。`i`が3のとき、`done`は`true`になり、`value`を設定する三項条件演算子は`undefined`と評価されます。これらの2つの結果は、最後のデータが使用された後にイテレータでnext()が呼び出されるECMAScript6のイテレータの特別な最後のケースを満たします。

この例が示すように、ECMAScript6で定義された規則に従って動作するイテレータを作成するのは少し複雑です。

幸いにも、ECMAScript6はジェネレータも提供しています。これにより、イテレータオブジェクトの作成がはるかに簡単になります。

## 発電機とは何ですか？

A *ジェネレータ*はイテレータを返す関数です。ジェネレータ機能は、`function`キーワードの後に​​スター文字(`*`)で示され、新しい`yield`キーワードを使用します。`star`が`function`のすぐ隣にあるのか、`star`と`*`文字の間に空白があるかどうかは関係ありません。

```js
// generator
function *createIterator() {
    yield 1;
    yield 2;
    yield 3;
}

// generators are called like regular functions but return an iterator
let iterator = createIterator();

console.log(iterator.next().value);     // 1
console.log(iterator.next().value);     // 2
console.log(iterator.next().value);     // 3
```

`createIterator()`の前の`*`は、この関数をジェネレータにします。`yield`キーワードは、ECMAScript6の新バージョンでも、`next()`が呼び出されたときに返される値を、返される順序で指定します。この例で生成されるイテレータは、`next()`メソッドへの連続した呼び出しで返される3つの異なる値を持っています：最初は`1`、その後は`2`、最後は`3`です。ジェネレータは、`iterator`が作成されたときに示されるように、他の関数と同様に呼び出されます。

おそらく、ジェネレータ関数の最も興味深い点は、各`yield`ステートメントの後に実行を停止することです。例えば、このコードで`yield 1`を実行した後、イテレータの`next()`メソッドが呼び出されるまで関数は何も実行しません。その時点で、`yield 2`が実行されます。関数の途中で実行を停止するこの機能は非常に強力で、ジェネレータ関数の興味深い使い方につながります(「高度なイテレータの機能」のセクションで説明します)。

`yield`キーワードは任意の値や式で使うことができるので、項目を1つずつリストするだけでイテレータに項目を追加するジェネレータ関数を書くことができます。例えば、`for`ループの中で`yield`を使う方法の一つがあります：

```js
function *createIterator(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
}

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// for all further calls
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

この例では`items`という配列を`createIterator()`ジェネレータ関数に渡します。 関数の中で、`for`ループは、ループが進むにつれて配列からイテレータに要素を返します。`yield`が出現するたびにループが止まり、`next()`が`iterator`で呼び出されるたびに、ループは次の`yield`ステートメントでピックアップします。

Generator関数はECMAScript6の重要な機能であり、関数なので同じ場所でも使用できます。 このセクションの残りの部分では、ジェネレータを書くための他の便利な方法に焦点を当てています。

W>`yield`キーワードは、ジェネレータの内部でのみ使用できます。 他の場所で`yield`を使用すると、以下のようなジェネレータの内部にある関数を含む構文エラーです：
W>
W>```js
W> function *createIterator(items) {
W>
W>     items.forEach(function(item) {
W>
W>         // syntax error
W>         yield item + 1;
W>     });
W> }
W>```
W>
W>`yield`は技術的に`createIterator()`の内部にありますが、`yield`は関数の境界を越えることができないので、このコードは構文エラーです。 このように、`yield`は`return`と似ています。入れ子関数はその関数を含む値を返すことができません。

### ジェネレータ関数の式

`function`キーワードと開始括弧の間にスター(`*`)文字を入れるだけで、関数式を使ってジェネレータを作成することができます。 例えば：

```js
let createIterator = function *(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
};

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// for all further calls
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

このコードでは、`createIterator()`は関数宣言の代わりにジェネレータ関数式です。 アスタリスクは、関数式が匿名であるため、`function`キーワードと開始括弧の間に入ります。 さもなければ、この例は`for`ループを使った`createIterator()`関数の前のバージョンと同じです。

>ジェネレータである矢印関数を作成することはできません。

### ジェネレータオブジェクトメソッド

ジェネレータは単なる関数なので、オブジェクトにもジェネレータを追加することができます。 たとえば、ファンクション式を使用してECMAScript5スタイルのオブジェクトリテラルでジェネレータを作成できます。

```js
var o = {

    createIterator: function *(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);
```

ECMAScript6メソッドの省略形は、メソッド名の前にスター(`*`)を付けて使うこともできます：

```js
var o = {

    *createIterator(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);
```

これらの例は、機能的には「ジェネレータ関数式」の例の例と同等です。彼らは異なる構文を使用します。省略形では、`createIterator()`メソッドは`function`キーワードなしで定義されているので、スターとメソッド名の間に空白を残すことはできますが、スターはメソッド名の直前に置かれます。

## Iterablesとfor-of

イテレータと密接に関連している* iterable *は、`Symbol.iterator`プロパティを持つオブジェクトです。良く知られている`Symbol.iterator`シンボルは、与えられたオブジェクトのイテレータを返す関数を指定します。 ECMAScript6では、すべてのコレクションオブジェクト(配列、セット、マップ)と文字列がiterableであるため、デフォルトイテレーターが指定されています。イテラブルは、ECMAScriptに新たに追加されたfor-ofループを使用するように設計されています。

I>ジェネレータによって生成されるすべてのイテレータは、デフォルトで`Symbol.iterator`プロパティをジェネレータが割り当てるので、イテラブルです。

この章の初めに、私は`for`ループの中でインデックスを追跡する問題について述べました。イテレータは、この問題の解決策の最初の部分です。`for-of`ループは第2の部分です：コレクションの中身を完全に追跡する必要がなくなり、コレクションの内容を自由に扱うことができます。

`for-of`ループは、ループが実行され、結果オブジェクトからの`value`を変数に格納するたびに、iterable上で`next()`を呼び出します。ループは、返されたオブジェクトの`done`プロパティが`true`になるまでこのプロセスを継続します。ここに例があります：

```js
let values = [1, 2, 3];

for (let num of values) {
    console.log(num);
}
```

このコードは次を出力します：

```
1
2
3
```

この`for-of`ループは`values`配列の`Symbol.iterator`メソッドを最初に呼び出してイテレータを取得します。 (`Symbol.iterator`への呼び出しは、JavaScriptエンジン自体の中で起こります。)`iterator.next()`が呼び出され、イテレータの結果オブジェクトの`value`プロパティが`num`に読み込まれます。`num`変数は、最初は1、次に2、そして最後に3です。結果オブジェクトで`done`が`true`のとき、ループは終了します.`num`は決して`undefined`の値に割り当てられません。

単純に配列やコレクションの値を反復処理するのであれば、`for`ループの代わりに`for-of`ループを使うことをお勧めします。`for-of`ループは、追跡する条件が少なくて済むため、一般的にエラーが起こりにくいです。より複雑な制御条件のために伝統的な`for`ループを保存します。

W>`for-of`ステートメントは、on、iterable以外のオブジェクト、`null`、`undefined`を使用するとエラーをスローします。

### デフォルトイテレータへのアクセス

`Symbol.iterator`を使ってオブジェクトのデフォルトイテレータにアクセスすることができます：

```js
let values = [1, 2, 3];
let iterator = values[Symbol.iterator]();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

このコードは、`values`のデフォルトイテレータを取得し、それを使って配列内の項目を反復処理します。 これは`for-of`ループを使用したときの背後で起こる同じプロセスです。

`Symbol.iterator`はデフォルトのイテレータを指定するので、それを使ってオブジェクトが次のように反復可能かどうかを検出することができます：

```js
function isIterable(object) {
    return typeof object[Symbol.iterator] === "function";
}

console.log(isIterable([1, 2, 3]));     // true
console.log(isIterable("Hello"));       // true
console.log(isIterable(new Map()));     // true
console.log(isIterable(new Set()));     // true
console.log(isIterable(new WeakMap())); // false
console.log(isIterable(new WeakSet())); // false
```

`isIterable()`関数は、単にオブジェクトにデフォルトのイテレータが存在するかどうかを調べ、関数であるかどうかを調べます。`for-of`ループは実行前に同様のチェックを行います。

ここまでの例では、組み込みのiterable型を持つ`Symbol.iterator`を使用する方法を示しましたが、`Symbol.iterator`プロパティを使用して独自のiterableを作成することもできます。

### イテラブルを作成する

開発者定義のオブジェクトは、デフォルトでは反復可能ではありませんが、ジェネレータを含む`Symbol.iterator`プロパティを作成することで反復可能にすることができます。 例えば：

```js
let collection = {
    items: [],
    *[Symbol.iterator]() {
        for (let item of this.items) {
            yield item;
        }
    }

};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}
```

このコードは次を出力します：

```
1
2
3
```

まず、この例は、`collection`と呼ばれるオブジェクトのデフォルトイテレータを定義します。デフォルトのイテレータは`Symbol.iterator`メソッドによって生成されます。このメソッドはジェネレータです(名前の前にはまだ星があります)。その後、ジェネレータは`for-of`ループを使用して`this.items`の値を反復処理し、`yield`を使用してそれぞれを返します。`collection`オブジェクトは、手動で反復して`collection`のデフォルトイテレータの値を定義する代わりに、`this.items`のデフォルトイテレータに依存して作業を行います。

I>この章の「ジェネレータの委譲」では、別のオブジェクトのイテレータを使用する別の方法について説明します。

今では、デフォルトの配列イテレータの用途をいくつか見てきましたが、ECMAScript6にはより多くのイテレータが組み込まれており、データのコレクションを扱いやすくしています。

## 組み込みイテレータ

イテレータはECMAScript6の重要な部分であり、多くのビルトイン型に対して独自のイテレータを作成する必要はありません。言語にはデフォルトでそれらが含まれています。ビルトインのイテレータが目的を果たさないときは、イテレータを作成する必要があります。これは、独自のオブジェクトまたはクラスを定義するときに最も頻繁に発生します。それ以外の場合、ビルトインイテレーターを使用して作業を行うことができます。使用する最も一般的なイテレータは、おそらくコレクションで動作するイテレータです。

コレクションイテレータ

ECMAScript6には、配列、マップ、セットという3種類のコレクションオブジェクトがあります。これらの3つには、コンテンツのナビゲートに役立つ組み込みのイテレータが組み込まれています。

*`entries()`- 値がキーと値のペアであるイテレータを返します
* values()`- 値がコレクションの値であるイテレータを返します。
* keys()`- コレクションに含まれるキーの値を持つイテレータを返します。

これらのメソッドの1つを呼び出すことによって、コレクションのイテレータを取得できます。

#### entries()Iterator

`entries()`イテレータは`next()`が呼び出されるたびに2項目の配列を返します。 2項目配列は、コレクション内の各項目のキーと値を表します。配列の場合、最初の項目は数値インデックスです。セットの場合、最初のアイテムも値です(値はセット内のキーの2倍です)。マップの場合、最初の項目はキーです。

このイテレータを使用するいくつかの例を以下に示します。

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript6");
data.set("format", "ebook");

for (let entry of colors.entries()) {
    console.log(entry);
}

for (let entry of tracking.entries()) {
    console.log(entry);
}

for (let entry of data.entries()) {
    console.log(entry);
}
```

`console.log()`呼び出しは次の出力を与えます：

```
[0, "red"]
[1, "green"]
[2, "blue"]
[1234, 1234]
[5678, 5678]
[9012, 9012]
["title", "Understanding ECMAScript6"]
["format", "ebook"]
```

このコードは、イテレータを取得するためにコレクションの各タイプで`entries()`メソッドを使用し、`for-of`ループを使用してアイテムを反復します。 コンソール出力は、オブジェクトごとにキーと値がどのようにペアで返されるかを示します。

#### values()Iterator

`values()`イテレータは単にコレクションに格納されている値を返します。 例えば：

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript6");
data.set("format", "ebook");

for (let value of colors.values()) {
    console.log(value);
}

for (let value of tracking.values()) {
    console.log(value);
}

for (let value of data.values()) {
    console.log(value);
}
```

このコードは次を出力します：

```
"red"
"green"
"blue"
1234
5678
9012
"Understanding ECMAScript6"
"ebook"
```

この例のように`values()`イテレータを呼び出すと、コレクション内のそのデータの場所に関する情報を持たずに、各コレクションに含まれる正確なデータが返されます。

#### keys()Iterator

`keys()`イテレータは、コレクションに存在する各キーを返します。 配列の場合、数値キーのみを返します。配列の他のプロパティは決して返しません。 セットの場合、キーは値と同じであるため、`keys()`と`values()`は同じイテレータを返します。 マップの場合、`keys()`イテレータはそれぞれのユニークキーを返します。 次の3つの例を示します。

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript6");
data.set("format", "ebook");

for (let key of colors.keys()) {
    console.log(key);
}

for (let key of tracking.keys()) {
    console.log(key);
}

for (let key of data.keys()) {
    console.log(key);
}
```

この例では、次のものが出力されます。

```
0
1
2
1234
5678
9012
"title"
"format"
```

`keys()`イテレータは、`colors`、`tracking`、`data`の各キーを取り出し、3つの`for-of`ループの内側から出力します。 配列オブジェクトでは、数値のインデックスのみが出力されます。これは、名前付きのプロパティを配列に追加したとしても発生します。 これは`for-in`ループが単に数値インデックスではなくプロパティを繰り返し処理するため、`for-in`ループが配列で動作する方法とは異なります。

#### コレクション型のデフォルトイテレータ

各コレクション型には、イテレータが明示的に指定されていないときに`for-of`によって使用されるデフォルトのイテレータもあります。`values()`メソッドは配列とセットのデフォルトイテレータであり、`entries()`メソッドはマップのデフォルトイテレータです。 これらのデフォルトは`for-of`ループ内のコレクションオブジェクトを使用するのを少し容易にします。 たとえば、次の例を考えてみましょう。

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript6");
data.set("format", "print");

// same as using colors.values()
for (let value of colors) {
    console.log(value);
}

// same as using tracking.values()
for (let num of tracking) {
    console.log(num);
}

// same as using data.entries()
for (let entry of data) {
    console.log(entry);
}
```

イテレーターは指定されていないので、デフォルトのイテレーター関数が使用されます。 配列、セット、マップのデフォルトのイテレーターは、これらのオブジェクトの初期化方法を反映するように設計されているため、このコードは以下を出力します。

```
"red"
"green"
"blue"
1234
5678
9012
["title", "Understanding ECMAScript6"]
["format", "print"]
```

配列とセットはデフォルトで値を返し、マップは`Map`コンストラクタに渡すことができる配列フォーマットを返します。 一方、弱いセットと弱いマップには、イテレータが組み込まれていません。 弱参照を管理するということは、これらのコレクションに含まれる値の数を正確に知る方法がないことを意味します。つまり、それらのコレクションを反復する方法はありません。

A> ###構造化とfor-ofループ
A>
A>マップのデフォルトのイテレータの動作は、この例のように、非構造化の`for-of`ループで使用すると役に立ちます：
A>
A>```js
A> let data = new Map();
A>
A> data.set("title", "Understanding ECMAScript6");
A> data.set("format", "ebook");
A>
A> // same as using data.entries()
A> for (let [key, value] of data) {
A>     console.log(key + "=" + value);
A> }
A>```
A>
A>このコードの`for-of`ループは、非構造化配列を使ってマップの各エントリに`key`と`value`を割り当てます。 このようにして、2つのアイテムの配列にアクセスしたり、キーまたは値のいずれかを取得するためにマップに戻ったりすることなく、キーと値を同時に簡単に操作できます。 マップに非構造化配列を使用すると、`for-of`ループはマップの場合と同じようにセットと配列のために便利です。

### ストリングイテレータ

JavaScript文字列は、ECMAScript5がリリースされて以来、ゆっくりと配列に似ています。 例えば、ECMAScript5は、文字列内の文字にアクセスするためのブラケット表記を形式化しています(つまり、最初の文字を得るために`text [0]`を使用します)。 しかし、ブラケット記法は文字ではなくコード単位で機能するので、2バイト文字に正しくアクセスするために使用することはできません。

```js
var message = "A  B";

for (let i=0; i < message.length; i++) {
    console.log(message[i]);
}
```

このコードでは、ブラケット記法と`length`プロパティを使用して、Unicode文字を含む文字列を繰り返して出力します。 出力は予期せぬものです。

```
A
(blank)
(blank)
(blank)
(blank)
B
```

2バイト文字は2つの別々のコード単位として扱われるので、出力には`A`と`B`の間に4つの空行があります。

幸いにも、ECMAScript6はUnicode(第2章を参照)を完全にサポートすることを目指しており、デフォルトの文字列反復子は文字列反復問題を解決する試みです。 そのため、文字列のデフォルトイテレータはコード単位ではなく文字で動作します。 この例を変更して、デフォルトの文字列イテレータを`for-of`ループで使用すると、より適切な出力が得られます。 ここには微調整されたコードがあります：


```js
var message = "A  B";

for (let c of message) {
    console.log(c);
}
```

これにより、以下が出力されます。

```
A
(blank)

(blank)
B
```

この結果は、文字を扱うときの期待に沿ったものになります。ループはUnicode文字だけでなく残りの文字も正常に出力します。

### NodeListイテレータ

DOM(Document Object Model)には、ドキュメント内の要素のコレクションを表す`NodeList`型があります。 JavaScriptをWebブラウザで実行する人にとって、`NodeList`オブジェクトと配列の違いを理解することは、常に少し難しかったです。`NodeList`オブジェクトと配列はどちらも`length`プロパティを使って項目の数を示し、両方とも括弧表記を使って個々の項目にアクセスします。しかし、内部的には、`NodeList`と配列は全く異なった振る舞いをするので、多くの混乱を招いています。

ECMAScript6にデフォルトイテレータを追加すると、NodeList(ECMAScript6自体ではなくHTML仕様に含まれる)のDOM定義には、配列のデフォルトイテレータと同じように動作するデフォルトイテレータが含まれます。つまり、`for-of`ループやオブジェクトのデフォルトイテレータを使用する他の場所で`NodeList`を使うことができます。例えば：

```js
var divs = document.getElementsByTagName("div");

for (let div of divs) {
    console.log(div.id);
}
```

このコードは、`document`オブジェクトのすべての`<div>`要素を表す`NodeList`を取得するために`getElementsByTagName()`を呼び出します。`for-of`ループは各要素を繰り返し処理して要素IDを出力し、標準配列と同じようにコードを効果的に作ります。

## スプレッドオペレータと非配列イテラブル

第7章から拡散演算子(`...`)を使って、集合を配列に変換することができることを思い出してください。 例えば：

```js
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

このコードでは、配列リテラル内のspread演算子を使用して、その配列を`set`の値で埋めています。 スプレッド演算子はすべてのiterableで動作し、デフォルトのiteratorを使用してどの値を含めるかを決定します。 すべての値はイテレータから読み込まれ、イテレータから返された値の順序で配列に挿入されます。 この例は、セットがiterableなので動作しますが、どのiterableでも同じように動作します。 別の例があります：

```js
let map = new Map([ ["name", "Nicholas"], ["age", 25]]),
    array = [...map];

console.log(array);         // [ ["name", "Nicholas"], ["age", 25]]
```

ここで、スプレッド演算子は`map`を配列の配列に変換します。 マップのデフォルトのイテレータはキーと値のペアを返すので、結果の配列は`new Map()`の呼び出しで渡された配列のように見えます。

配列リテラルでは、スプレッド演算子を何度でも使用できます。繰り返し演算子から複数の項目を挿入する場合は、どこでも使用できます。 これらの項目は、スプレッド演算子の位置にある新しい配列内に順番に表示されます。 例えば：

```js
let smallNumbers = [1, 2, 3],
    bigNumbers = [100, 101, 102],
    allNumbers = [0, ...smallNumbers, ...bigNumbers];

console.log(allNumbers.length);     // 7
console.log(allNumbers);    // [0, 1, 2, 3, 100, 101, 102]
```

spread演算子は`smallNumbers`と`bigNumbers`の値から`allNumbers`を作成するために使われます。値は、`allNumbers`が作成されたときに配列が追加されるのと同じ順序で`allNumbers`に配置されます。`0`が最初に続き、`smallNumbers`の値に続いて`bigNumbers`の値が続きます。しかし元の配列は変更されていませんが、その値は`allNumbers`にコピーされただけです。

スプレッド演算子は任意のiterableで使用できるので、iterableを配列に変換するのが最も簡単な方法です。文字列を文字の配列(コード単位ではない)に変換し、ブラウザの`NodeList`オブジェクトをノードの配列に変換することができます。

for-ofやspreadオペレータなど、イテレータの動作の基本を理解したので、イテレータの複雑な使い方を見てみましょう。

## 高度なイテレータ機能

イテレーターの基本機能とジェネレーターを使用したイテレーターの作成の利便性によって、多くのことが達成できます。しかし、イテレータは、単に値の集合を反復する以外のタスクに使用すると、はるかに強力です。 ECMAScript6の開発中に、クリエイターがより多くの機能を追加するように促すユニークなアイデアやパターンが多数登場しました。これらの追加の一部は微妙ですが、一緒に使用すると面白いやりとりを実現できます。

### イテレータへの引数の受け渡し

この章では、イテレータが`next()`メソッドで値を渡すか、ジェネレータで`yield`を使う例を示しました。しかし、`next()`メソッドを使って反復子に引数を渡すこともできます。引数が`next()`メソッドに渡されると、その引数はジェネレータ内部の`yield`ステートメントの値になります。この機能は、非同期プログラミングなどの高度な機能にとって重要です。基本的な例を次に示します。

```js
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;       // 4 + 2
    yield second + 3;                   // 5 + 3
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next(4));          // "{ value: 6, done: false }"
console.log(iterator.next(5));          // "{ value: 8, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

`next()`への最初の呼び出しは、それに渡された引数が失われる特殊なケースです。`next()`に渡される引数は`yield`によって返される値になるので、`next()`への最初の呼び出しからの引数は、`yield`ステートメント。これは不可能なので、`next()`が最初に呼び出されたときに引数を渡す理由はありません。

`next()`の2回目の呼び出しでは、引数として値 '4'が渡されます。`4`は、ジェネレータ関数内の変数`first`に割り当てられます。代入を含む`yield`ステートメントでは、式の右辺は`next()`の最初の呼び出しで評価され、左辺は関数が実行を続ける前に`next()`の2回目の呼び出しで評価されます。`next()`への2回目の呼び出しは`4 'で渡されるので、その値は`first`に割り当てられてから実行が続けられます。

2番目の`yield`は最初の`yield`の結果を使用して2を加算します。つまり、6の値を返します。`next()`が3回目に呼び出されると、値として '5'が引数として渡されます。その値は変数`second`に代入され、次に第3の`yield`ステートメントで`8`を返すために使われます。

実行がジェネレータ関数内で継続するたびに、どのコードが実行されているかを検討することで、何が起こっているのかを考えるのは簡単です。図8-1では、色を使用して降伏前に実行されているコードを示しています。

![Figure 8-1: Code execution inside a generator](images/fg0601.png)

黄色は`next()`への最初の呼び出しと、その結果としてジェネレータ内部で実行されるすべてのコードを表します。カラーアクアは、`next(4)`の呼び出しとその呼び出しで実行されるコードを表します。紫色は`next(5)`への呼び出しとその結果として実行されるコードを表します。トリッキーな部分は、左辺が実行される前に、各式の右側のコードがどのように実行され、停止するかです。これにより、複雑なジェネレータは通常の関数をデバッグするよりも複雑になります。

これまでは、`yield()`メソッドが`next()`メソッドに値が渡されたときに`yield`が`return`のように働くことが分かりました。しかし、ジェネレータの中で実行できる唯一の実行トリックではありません。イテレータがエラーをスローする可能性もあります。

### イテレータでの投げ込みエラー

イテレータにデータを渡すだけでなく、エラー条件も渡すことができます。反復子は、イテレータが再開時にエラーをスローするよう指示する`throw()`メソッドを実装することを選択できます。これは非同期プログラミングにとって重要な機能ですが、戻り値とスローされたエラー(関数を終了する2つの方法)の両方を模倣できるようにする、ジェネレータ内部の柔軟性のためにも重要です。イテレータが処理を続行するときにスローされるべき`throw()`にエラーオブジェクトを渡すことができます。例えば：

```js
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;       // yield 4 + 2, then throw
    yield second + 3;                   // never is executed
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // error thrown from generator
```

この例では、最初の2つの`yield`式は通常通り評価されますが、`throw()`が呼び出されると、`let second`が評価される前にエラーがスローされます。 これにより、エラーを直接スローするのと同様に、コードの実行が停止します。 唯一の違いは、エラーがスローされる場所です。 図8-2に、各ステップで実行されるコードを示します。

![Figure 8-2: Throwing an error inside a generator](images/fg0602.png)

この図では、赤色は`throw()`が呼び出されたときに実行されるコードを表し、赤い星印はジェネレータ内部でエラーがスローされたときにほぼ表示されます。 最初の2つの`yield`文が実行され、`throw()`が呼び出されると、他のコードが実行される前にエラーがスローされます。

これを知っているなら、`try-catch`ブロックを使ってジェネレータ内部でそのようなエラーを捕まえることができます：

```js
function *createIterator() {
    let first = yield 1;
    let second;

    try {
        second = yield first + 2;       // yield 4 + 2, then throw
    } catch (ex) {
        second = 6;                     // on error, assign a different value
    }
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false }"
console.log(iterator.next());                   // "{ value: undefined, done: true }"
```

この例では、`try-catch`ブロックが2番目の`yield`ステートメントの周りにラップされています。この`yield`はエラーなしで実行されますが、`second`に値を代入する前にエラーがスローされるため、`catch`ブロックには値6が代入されます。実行は次の`yield`に流れ、9を返します。

興味深いことが起こったことに注目してください。`throw()`メソッドは`next()`メソッドのように結果オブジェクトを返しました。エラーがジェネレータ内部で捕捉されたため、コード実行は次の`yield`に続き、次の値`9`を返しました。

これは、`next()`と`throw()`をイテレータの両方の命令として考えるのに役立ちます。`next()`メソッドは反復子に実行を継続するよう指示し、`throw()`は反復子にエラーを投げて実行を続けるよう指示します。その時点以降は、ジェネレーター内のコードに依存します。

`next()`と`throw()`メソッドは`yield`を使うときイテレータ内の実行を制御しますが、`return`ステートメントを使うこともできます。しかし、`return`は通常の関数とは少し違って動作します。これについては次のセクションで説明します。

### 発電機のリターン・ステートメント

ジェネレータは関数なので、`return`文を使って早期に終了し、`next()`メソッドへの最後の呼び出しの戻り値を指定することができます。この章のほとんどの例では、イテレータの`next()`を最後に呼び出すと`undefined`が返されますが、他の関数と同様に`return`を使って代替値を指定することができます。ジェネレータでは、`return`はすべての処理が完了したことを示しているので、`done`プロパティは`true`に設定され、値が与えられれば`value`フィールドになります。早期に`return`を使用して終了する例を次に示します。

```js
function *createIterator() {
    yield 1;
    return;
    yield 2;
    yield 3;
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

このコードでは、ジェネレータには`yield`ステートメントとそれに続く`return`ステートメントがあります。`return`はそれ以上来るべき値がないことを示し、残りの`yield`ステートメントは実行されません(到達できません)。

また、返されるオブジェクトの`value`フィールドに終わる戻り値を指定することもできます。 例えば：

```js
function *createIterator() {
    yield 1;
    return 42;
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 42, done: true }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

ここで、`next`(`done`が`true`である最初の)メソッドへの2回目の呼び出しで`value`フィールドに値`42`が返されます。`next()`への3回目の呼び出しは、`value`プロパティが再び`undefined`であるオブジェクトを返します。`return`で指定した値は、`value`フィールドが`undefined`にリセットされる前に、返されたオブジェクトで1回だけ使用できます。

I>スプレッド演算子と`for-of`は`return`ステートメントで指定された値を無視します。`done`が`true`であるとすぐに`value`を読まずに停止します。 ただし、ジェネレータを委譲するときは、イテレータの戻り値が役立ちます。

### ジェネレータの委任

場合によっては、2つのイテレータの値を1つに組み合わせると便利です。 ジェネレータは、星型(`*`)の文字を持つ特別な形式の`yield`を使って他のイテレータに委譲することができます。 ジェネレータ定義の場合と同様に、星が現れる場所は、星が`yield`キーワードとジェネレータ関数名の間にある限り重要ではありません。 ここに例があります：

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
}

function *createColorIterator() {
    yield "red";
    yield "green";
}

function *createCombinedIterator() {
    yield *createNumberIterator();
    yield *createColorIterator();
    yield true;
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "red", done: false }"
console.log(iterator.next());           // "{ value: "green", done: false }"
console.log(iterator.next());           // "{ value: true, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

この例では、`createCombinedIterator()`ジェネレータは`createNumberIterator()`から返されたイテレータに、次に`createColorIterator()`から返されたイテレータに委譲します。`createCombinedIterator()`から返されたイテレータは、外部からすべての値を生成した一貫したイテレータになるように見えます。`next()`に対する各呼び出しは、`createNumberIterator()`と`createColorIterator()`によって作成されたイテレータが空になるまで適切なイテレータに委譲されます。 最終的な`yield`が`true`を返すために実行されます。

ジェネレータの委任によって、ジェネレータの戻り値をさらに利用することもできます。 これは、そのような戻り値にアクセスする最も簡単な方法であり、複雑なタスクを実行する上で非常に役立ちます。 例えば：

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator(count) {
    for (let i=0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

ここで、`createCombinedIterator()`ジェネレータは`createNumberIterator()`に委譲し、戻り値を`result`に代入します。`createNumberIterator()`は`return 3`を含んでいるので、返される値は`3`です。`result`変数は、同じ文字列を返す回数(この場合は3回)を示す引数として`createRepeatingIterator()`に渡されます。

値`3`は`next()`メソッドの呼び出しから決して出力されなかったことに注意してください。 今は`createCombinedIterator()`ジェネレータの内部にのみ存在します。 しかし、次のような別の`yield`ステートメントを追加して、その値を出力することもできます：

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator(count) {
    for (let i=0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield result;
    yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

このコードでは、余分な`yield`文が`createNumberIterator()`ジェネレータから返された値を明示的に出力します。

戻り値を使用するジェネレータの委譲は、特に非同期操作と組み合わせて使用​​する場合、非常に興味深い可能性を可能にする非常に強力なパラダイムです。

I>文字列(`yield *" hello "`など)に`yield *`を直接使うことができ、文字列のデフォルトイテレータが使用されます。

## 非同期タスク実行中

ジェネレータの周りの興奮の多くは、非同期プログラミングに直接関係しています。 JavaScriptの非同期プログラミングは両刃の剣です。シンプルなタスクは非同期で行うのが簡単で、複雑なタスクはコード構成の邪魔になります。ジェネレータを使用すると、実行中にコードを効果的に停止することができるため、非同期処理に関連する多くの可能性が開かれます。

非同期操作を実行する従来の方法は、コールバックを持つ関数を呼び出すことです。たとえば、Node.js内のディスクからファイルを読み取る場合を考えます。

```js
let fs = require("fs");

fs.readFile("config.json", function(err, contents) {
    if (err) {
        throw err;
    }

    doSomethingWith(contents);
    console.log("Done");
});
```

`fs.readFile()`メソッドは、読み込むファイル名とコールバック関数とともに呼び出されます。 操作が終了すると、コールバック関数が呼び出されます。 コールバックはエラーがあるかどうかをチェックし、そうでなければ、返された`contents`を処理します。 小さな非同期タスクを完了するために、限られた数の非同期タスクがある場合はうまくいくが、コールバックをネストしたり、一連の非同期タスクをシーケンスする必要がある場合は複雑になります。 これはジェネレータと`yield`が役に立ちます。

### 単純なタスクランナー

`yield`は実行を停止し、`next()`メソッドが再び呼び出されるのを待っているので、コールバックを管理せずに非同期呼び出しを実装できます。 まず、ジェネレータを呼び出してイテレータを開始できる関数が必要です。

```js
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            result = task.next();
            step();
        }
    }

    // start the process
    step();

}
```

`run()`関数はタスク定義(ジェネレータ関数)を引数として受け付けます。 ジェネレータを呼び出してイテレータを作成し、イテレータを`task`に格納します。`task`変数は関数の外にあり、他の関数からアクセスすることができます。 私はこのセクションの後半の理由を説明します。`next()`への最初の呼び出しはイテレータを開始し、結果は後で使用するために保存されます。`step()`関数は`result.done`が偽であるかどうかを調べ、もしそうなら、再帰的に自身を呼び出す前に`next()`を呼び出します。`next()`を呼び出すたびに戻り値が`result`に格納されます。これは常に最新の情報を含むように上書きされます。`step()`への最初の呼び出しは、`result.done`変数を調べて、それ以上のことがあるかどうかを確認するプロセスを開始します。

この`run()`の実装では、以下のような複数の`yield`文を含むジェネレータを実行することができます：

```js
run(function*() {
    console.log(1);
    yield;
    console.log(2);
    yield;
    console.log(3);
});
```

この例では、単に`next()`への呼び出しが行われていることを示す3つの数字をコンソールに出力します。 しかし、2〜3回しか得られないことはあまり有用ではありません。 次のステップでは、イテレータに値を渡したり、イテレータから値を渡したりします。

### データで実行中のタスク

タスクランナーにデータを渡す最も簡単な方法は、`yield`で指定された値を`next()`メソッドの次の呼び出しに渡すことです。 これを行うには、次のコードのように`result.value`を渡すだけです：

```js
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            result = task.next(result.value);
            step();
        }
    }

    // start the process
    step();

}
```

`result.value`が引数として`next()`に渡されたので、`yield`呼び出しの間で次のようにデータを渡すことができます：

```js
run(function*() {
    let value = yield 1;
    console.log(value);         // 1

    value = yield value + 3;
    console.log(value);         // 4
});
```

この例では、コンソールに1と4の2つの値を出力します。値1は、`value`変数に1が直ちに渡されるため、`yield 1`から取り出されます。 4は`value`に3を加え、その結果を`value`に返すことで計算されます。 データが`yield`への呼び出しの間に流れているので、非同期呼び出しを許可するにはちょっとした変更が必要です。

### 非同期タスクランナー

前の例では、`yield`呼び出しの間で静的なデータを渡しましたが、非同期処理を待っているのとは少し異なります。 タスクランナーは、コールバックとその使用方法を知る必要があります。 そして、`yield`式はそれらの値をタスクランナーに渡すので、どの関数呼び出しも、その呼び出しがタスクランナーが待つべき非同期操作であることを何とか示す値を返す必要があります。

値が非同期操作であることを知らせる1つの方法は次のとおりです。

```js
function fetchData() {
    return function(callback) {
        callback(null, "Hi!");
    };
}
```

この例では、タスクランナーによって呼び出されるすべての関数は、コールバックを実行する関数を返します。`fetchData()`関数は、引数としてコールバック関数を受け入れる関数を返します。 返された関数が呼び出されると、それは単一のデータ(``Hi！ "`文字列)でコールバック関数を実行します。`callback`引数はタスクランナから来て、コールバックの実行が根底にあるイテレータと正しく相互作用するようにする必要があります。`fetchData()`関数は同期式ですが、次のようなわずかな遅れでコールバックを呼び出すことで、非同期に簡単に拡張することができます：

```js
function fetchData() {
    return function(callback) {
        setTimeout(function() {
            callback(null, "Hi!");
        }, 50);
    };
}
```

このバージョンの`fetchData()`は、コールバックを呼び出す前に50msの遅延をもたらし、このパターンが同期コードと非同期コードで同じように機能することを示しています。`yield`を使って呼び出す各関数が同じパターンに従っていることを確認するだけです。

関数が非同期プロセスであることをどのように伝えるかをよく理解しているので、タスクランナーを変更してその事実を考慮に入れることができます。`result.value`が関数であるときはいつでも、タスクランナーはその値を`next()`メソッドに渡すのではなく、それを実行します。 更新されたコードは次のとおりです。

```js
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            if (typeof result.value === "function") {
                result.value(function(err, data) {
                    if (err) {
                        result = task.throw(err);
                        return;
                    }

                    result = task.next(data);
                    step();
                });
            } else {
                result = task.next(result.value);
                step();
            }

        }
    }

    // start the process
    step();

}
```

`result.value`が(`===`演算子でチェックされた)関数であるとき、それはコールバック関数で呼び出されます。 そのコールバック関数は、可能なエラーを第1引数(`err`)として渡し、結果を第2引数として渡すというNode.js規約に従います。`err`が存在する場合、エラーが発生したことを意味し、`task.throw()`は`task.next()`の代わりにエラーオブジェクトと共に呼び出され、正しい場所にエラーがスローされます。 エラーがなければ`data`が`task.next()`に渡され、結果が格納されます。 次に、`step()`が呼び出されて処理が続行されます。`result.value`が関数でないときは、`next()`メソッドに直接渡されます。

この新しいバージョンのタスクランナーは、すべての非同期タスクに対応しています。 Node.jsのファイルからデータを読み込むには、このセクションの先頭から`fetchData()`関数に似た関数を返す`fs.readFile()`の周りにラッパーを作成する必要があります。 例えば：

```js
let fs = require("fs");

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}
```

`readFile()`メソッドは単一の引数ファイル名を受け取り、コールバックを呼び出す関数を返します。 コールバックは`fs.readFile()`メソッドに直接渡され、完了時にコールバックを実行します。 次のように`yield`を使ってこのタスクを実行することができます：

```js
run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

この例では、メインコードでコールバックを表示せずに非同期の`readFile()`操作を実行しています。`yield`とは別に、コードは同期コードと同じように見えます。非同期操作を実行する関数がすべて同じインターフェイスに準拠している限り、同期コードのようなロジックを書くことができます。

もちろん、これらの例で使用されているパターンには欠点があります。つまり、関数を返す関数が非同期であることを必ずしも確かめることはできません。今のところ、実行中のタスクの背後にある理論を理解することが重要です。プロミスを使用すると、非同期タスクをスケジューリングするためのより強力な方法が提供されます。第11章では、このトピックについて詳しく説明します。

## まとめ

イテレータはECMAScript6の重要な部分であり、いくつかの重要な言語要素の根幹です。表面上では、イテレータは簡単なAPIを使用して一連の値を返す簡単な方法を提供します。しかし、ECMAScript6ではイテレータを使用するはるかに複雑な方法があります。

`Symbol.iterator`シンボルは、オブジェクトのデフォルトイテレータを定義するために使用されます。ビルトインオブジェクトと開発者定義オブジェクトの両方で、このシンボルを使用してイテレータを返すメソッドを提供できます。`Symbol.iterator`がオブジェクトに与えられると、そのオブジェクトは反復可能と見なされます。

`for-of`ループはiterablesを使ってループ内の一連の値を返します。`for-of`を使うのは、従来の`for`ループで繰り返すよりも簡単です。なぜなら、値を追跡する必要がなくなり、ループの終了時を制御する必要がなくなるからです。`for-of`ループはイテレータからすべての値を自動的に読み込み、それ以上がなくなると終了します。

for-ofを使いやすくするために、ECMAScript6の多くの値にはデフォルトイテレータがあります。すべてのコレクション型、つまり配列、マップ、およびセットには、内容に簡単にアクセスできるように設計されたイテレータがあります。文字列にはデフォルトのイテレータもあり、コード単位ではなく文字列の文字を簡単に繰り返します。

スプレッド演算子は任意の反復可能変数で動作し、反復可能配列を容易に配列に変換します。変換は、イテレータから値を読み取り、それらを配列に個別に挿入することによって機能します。

ジェネレータは、呼び出されると自動的にイテレータを作成する特殊な関数です。ジェネレータの定義は、スター(`*`)文字と、`next()`メソッドへの連続した呼び出しのたびに返される値を示す`yield`キーワードの使用によって示されます。

ジェネレータの委任では、既存のジェネレータを新しいジェネレータで再利用できるようにすることで、イテレータの動作を適切にカプセル化することを推奨します。既存のジェネレータを別のジェネレータで使用するには、`yield`ではなく`yield *`を呼び出します。このプロセスでは、複数のイテレータから値を返すイテレータを作成できます。

おそらく、ジェネレータとイテレータの最も興味深い面白い側面は、より洗練された非同期コードを作成する可能性です。どこでもコールバックを使用する必要はなく、同期的に見えるコードを設定できますが、実際には`yield`を使用して非同期操作が完了するまで待機します。
