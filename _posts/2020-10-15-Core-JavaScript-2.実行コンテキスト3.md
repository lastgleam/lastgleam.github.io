---
layout: post
title: "Core JavaScript 2.実行コンテキスト - スコープチェーン・this"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)を要約した内容になります。

# 2. 実行コンテキスト

## 目次

- コンテキストとは
- VariableEnvironment
- LexicalEnvironment
  - ホイスティング
  - 関数宣言と関数式
- **スコープとスコープチェーン**
- **グローバル変数とローカル変数**
- **まとめ**

## スコープとスコープチェーン

JavaScriptにおけるスコープとは、**識別子の有効範囲**である。
その有効な識別子を関数の中から外へ順次検索していく動作を**スコープチェーン**という。

### スコープチェーン

```javascript
var a = 1;
var hoge = function () {
  var fuga = function () {
    console.log(a); //------(1)
    var a = 3;
  };
  fuga();
  console.log(a);   //------(2)
};
hoge();
console.log(a);     //------(3)
```

上のコードの実行コンテキストは以下のようになる。

- グローバルコンテキスト
  - LexicalEnvironment（L.E）
    - environmentRecord（e）: a, hoge
- hogeの実行コンテキスト
  - L.E
    - e: fuga
    - outerEnvironReference（o）: グローバルL.E
- fugaの実行コンテキスト
  - L.E
    - e: a
    - o: hogeのL.E

実行コンテキストはグローバルコンテキスト→`hoge`のコンテキスト→`fuga`のコンテキストの順でコンテキストの範囲は小さくなっていくが、その分スコープチェーンによって参照できる識別子の範囲は広がる。
`hoge`からは`fuga`の内部で宣言された変数に参照できないが、`fuga`内部からは`fuga`・`hoge`・グローバルスコープ全て参照できる。

が、必ずしも参照できるわけではない。
変数`a`は`hoge`の外でも宣言されているし、また`fuga`の中でも宣言されている。
この場合、スコープチェーン上の1番目の`fuga`のLEに`a`が存在するため、これ以上検索せず`fuga`の`a`を返却する。
このように外部変数がが内部変数によって隠される現象を **変数のシャドウイング（Variable Shadowing）** と言う。

### グローバル変数とローカル変数

- グローバル変数
  - グローバルスコープで宣言された変数
- ローカル変数
  - ある関数の内部で宣言された変数

```javascript
var a = 1;                  //------グローバル変数
var hoge = function () {    //------グローバル変数
  var fuga = function () {  //-----ローカル変数
    console.log(a);
    var a = 3;              //-----ローカル変数
  };
  fuga();
  console.log(a);
};
hoge();
console.log(a);
```

関数宣言と関数式ですでに分かる通り、コードの安全性のためグローバル変数を宣言するのは避けるべきだ。

## this

基本、実行コンテキストの`thisBinding`には`this`と指定されたオブジェクトが保存される。thisが指定されなかった場合はグローバルオブジェクトが保存される。
また、関数を呼び出す方法によって`this`に保存される対象が異なってくる。

## まとめ

### 実行コンテキスト

実行するコードに提供する環境情報を集めたオブジェクト

#### 種類

- グローバルコンテキスト
- `eval()`で実行されたJavaScriptコードのコンテキスト
- 関数のコンテキスト

#### 内部構造

- VaribleEnvironment（不変）
- LexicalEnvironment（可変）
  - environmentRecord（変数名・関数名などの識別子）
  - outerEnvironmentReference（直前に生成されたコンテキストのLLexicalEnvironment）
- thisBinding

### スコープ

変数の有効範囲

#### 検索順序

現在のコンテキストのLexicalEnvironment
↓（outerEnvironmentReference）
外のコンテキストのLexicalEnvironment
↓
undefined

### 変数のタイプ

- グローバル変数
  - グローバルコンテキストの変数
- ローカル変数
  - 関数によって生成された実行コンテキストの変数

### this

実行コンテキストを作成するタイミングで指定されたオブジェクトが保存される。
関数の呼び出し方によって値は異なってくる。
指定されなかった場合、グローバルオブジェクトが保存される。