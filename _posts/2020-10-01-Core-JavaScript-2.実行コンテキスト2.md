---
layout: post
title: "Core JavaScript 2.実行コンテキスト２"
author: "lastgleam"
---

# Chap2. 実行コンテキスト

## 目次

- VariableEnvironment
- LexicalEnvironment
  - ホイスティング
  - 関数宣言と関数式

## VariableEnvironment

実行コンテキストが生成する時、まずVariableEnvironmentに情報を入れて、それをコピーしてLexicalEnvironmentと作る。
その後は主にLexicalEnvironmentを活用することになる。
各Environmentは、

 - EnvironmentRecord
 - Outer-EnvironmentReference

 で構成される。

 ## LexicalEnvironment

 `lexical`の意味
  > 「語彙的（な）」・「語彙の」

### environmentRecordとホイスティング

environmentRecordには以下の識別子情報が保存される。

ある関数の実行コンテキストの場合、

- 関数のパラメータの識別子
- 内部関数
- varなどで宣言した変数

の3つの情報をコンテキスト内部で上から順番に収集する。

識別子情報収集はコード実行する前に行われるため
「JavaScriptは識別子を最上段に巻き上げる」といっても問題ない。

### ホイスティングの規則

#### 例１

```javascript
function a (x) {  // 収集対象1（パラメータ）
  console.log(x); // (1)
  var x;          // 収集対象2（変数宣言）
  console.log(x); // (2)
  var x = 2;      // 収集対象3（変数宣言・割り当て）
  console.log(x); // (3)
}
a(1);
```

ホイスティングというのが行われないとすると、`(1)`で `1`、`(2)`で `undefined`、`(3)`で`2`が出力されると思うことができる。

#### 例２

```javascript
function a () {
  var x = 1;      // 収集対象1（変数宣言・割り当て）
  console.log(x); // (1)
  var x;          // 収集対象2（変数宣言）
  console.log(x); // (2)
  var x = 2;      // 収集対象3（変数宣言・割り当て）
  console.log(x); // (3)
}
a();
```

例２で収集対象１がパラメータから変数宣言に変わったが、１例１と例２は同じLexicalEnvironmentを構成する。ホイスティングが行われるからだ。

ホイスティング処理は変数宣言を巻き上げて、割り当てはそのまま置いておく。

例２は以下のようにホイスティングされる。

```javascript
function a() {
  var x;          //収集対象1の変数宣言
  var x;          //収集対象2の変数宣言
  var x;          //収集対象3の変数宣言

  x = 1;          //収集対象1の割り当て
  console.log(x); // (1)
  console.log(x); // (2)
  x = 2;          //収集対象2の割り当て
  console.log(x); // (3)
}
```

最初`(1)`で `1`、`(2)`で `undefined`、`(3)`で`2`が出力されると予測したが、実際に`(1)`で `1`、`(2)`で `1`、`(3)`で`2`が出力される。

#### 例３

```javascript
function a() {
  console.log(b);  // (1)
  var b = 'bbb';   // 収集対象1（変数宣言・割り当て）
  console.log(b);  // (2)
  function b() { } // 収集対象2（関数宣言）
  console.log(b);  // (3)
}
a();
```

`(1)`で `undefined`、`(2)`で `bbb`、`(3)`で関数が出力されそうなコードになっている。

ホイスティング完了後のコードは以下のように変わる。

```javascript
function a() {
  var b;           // 収集対象1（変数宣言）
  var b = function () { } // 収集対象2（関数宣言）

  console.log(b);  // (1)
  var b = 'bbb';   // 収集対象1（変数割り当て）
  console.log(b);  // (2)
  console.log(b);  // (3)
}
a();
```

関数宣言は、関数名で宣言した変数に関数を割り当てたような形になる。
実行結果、`(1)`で関数、`(2)`で`bbb`、`(3)`で`bbb`が出力される。

### 関数宣言と関数式

関数を定義する方法には2つ種類がある。

 - 関数宣言（function declaration）
   - 関数の定義のみ行って、別途割り当てなどはしない
 - 関数式（function expression）
   - 定義した関数を変数に割り当てる

#### 関数宣言と関数式の違い

#### 例４

```javascript
console.log(sum(1, 2));
console.log(multiply(3, 4));

// 関数宣言
function sum(a, b) {
  return a + b;
}

// 関数式
var multiply = function (a, b) {
  return a * b;
}
```

#### ホイスティング結果

```javascript
var sum = function sum(a, b) { //関数宣言文全体をホイスティングする
  return a + b;
}
var multiply; //変数宣言だけ巻き上げる

console.log(sum(1, 2)); // (1)
console.log(multiply(3, 4)); // (2)

multiply = function (a, b) { //割り当ては巻き上がらない
  return a * b;
}
```
`(1)`では問題なく3が出力されるが、
`(2)`で`multiply is not function.`というエラーメッセージが出力される。

このように関数宣言はホイスティングが行われるため、どこで定義してもどこで定義されて実行できているのか把握しづらくなる。

ホイスティングという概念を理解していルトしても、コードを作成するのはJavaScriptのエンジンでなくプログラマーなので「宣言した後、呼び出すことができる」という概念の方が自然だ。

### 関数宣言の危険性

ただ違和感の話ではなく、実務で関数宣言が障害の原因になる危険性もある。

```javascript
console.log (sum(3, 4)); // (1) '3 + 4 = 7'
...
function sum(x, y) {
  return x + y;
}
...
var a = sum(1, 2); // (2) '1 + 2 = 3'
...
function sum(x, y) {
  return x + ' + ' + y + ' = ' + (x + y);
}

var c = sum(1, 2); // (3)  '1 + 2 = 3'
```

グローバルコンテキストが構成される時同じ変数名でそれぞれ異なる値を割り当てる場合、後から割り当てた値が先に割り当てた値を上書きする（override）。

`(1)`と`(2)`で `7`と`3`を返すことを期待したが、 `sum()`が上書きされて意図してなかった結果になってしまう。

なので関数式を使って関数定義すると、ホイスティングで割り当て部分が元の行に残るので上から順番に実行され、意図通り動作する。
また、不具合も早い段階で把握することができ、バグを未然に防ぐことができる。

```javascript
console.log (sum(3, 4)); // Uncaught Type Error: sum is not a fuction.
...
var sum = function (x, y) {
  return x + y;
}
...
var a = sum(1, 2); // (1) 3
...
var sum = function (x, y) {
  return x + ' + ' + y + ' = ' + (x + y);
}

var c = sum(1, 2); // (3)  '1 + 2 = 3'
```
