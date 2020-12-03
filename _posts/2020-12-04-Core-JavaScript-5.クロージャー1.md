---
layout: post
title: "Core JavaScript 5.クロージャー1"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)を要約した内容になります。

# 5. クロージャー

## 目次

1. **クロージャーの意味と原理の理解**
2. クロージャーとメモリ管理
3. クロージャーの活用例

## 1.クロージャーの意味と原理の理解

## クロージャー（Closure）とは

- 多くの関数型プログラミング言語で登場する一般的な概念
- ECMAScript明細にも特に定義されていない

### JS書籍によってバラバラなクロージャー定義

他のJS本でも１行にまとめて色んな説明をしている。

>  - 自分自身を含めているコンテキストを参照できる関数
>  - 関数が特定のスコープを参照できるように意図的にそのスコープで定義すること
> - 関数を宣言する時に作られた有効範囲が消えた後も呼び出すことができる関数
> - すでにライフサイクル上終わった外部関数の変数を参照する関数
> - 自由変数を持つ関数と自由変数を知る環境の結合
> - ローカル変数を参照している関数内の関数
> - 自分自身が生成される時のスコープで知った変数の中でいつか実行される時使う変数のみ覚えて維持する関数


### MDNの場合

特にMDNでは、「クロージャーは関数とその関数が宣言される当時のLexical Environmentの双方関係による現象」と紹介している。

Lexical Environmentは、outerEnvironmentReferenceの該当する。そのouterEnvironmentReferenceによってスコープが決定されスコープチェーンが可能になる。

あるコンテキストAの内部で宣言した内部関数Bの実行コンテキストが活性化される時点ではBからAへの参照ができて、AからBで宣言した変数などは参照できない。
なので内部関数BからAを参照する時に、双方関係が発生する。

上の内容をまとめると **「ある関数で宣言した変数を参照する内部関数でのみ発生する現象」** と言える。

### 外部関数の変数を参照する内部関数 - その1

```javascript
var outer = function () {
  var a = 1;
  var inner = function () {
    console.log(++a);
  };
  inner();
};
outer();
```

inner関数内部ではaを宣言しなかったのでouterのouterEnvironmentRefrenceにあるLexicalEnvironmentからaを探す。その後、aから1増加した2を出力する。

### 外部関数の変数を参照する内部関数 - その2

```javascript
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner();
};
var outer2 = outer();
console.log(outer2);    //2
```

6行目でinnerの実行結果を返しているので、outerの実行コンテキストが終了した時点でaを参照する対象がなくなる。

### 外部関数の変数を参照する内部関数 - その3

```javascript
var outer = function () {
  var a = 1;
  var inner = function () {
      return ++a;
  };
  return inner;
};
var outer2 = outer();
console.log(outer2());     //2
console.log(outer2());     //3
```

実行結果でなく、inner関数を返している。
outer実行コンテキストが終了される時outer2はouterの実行結果のinner関数を参照することになる。
その下の行ではinnerが2回実行される。

inner関数のenvironmentRecordには情報がないため、outerのLexicalEnvironmentを参照する。
スコープチェーンにによってouterで宣言したaに1増加させ2を返す。

inner関数が実行される時点でouterはもう終了された状態だけどどうやってouterのLexicalEnvironmentを参照できてしまうのか。
それはガベージコレクションの動作方式に起因する。

ガベージコレクタはある値を参照する変数が一つでも存在すると収集対象に含めない。

収集対象から除外されるのは外部変数を参照する内部関数が外部に引き渡された場合しかない。
それが例1と2もaを参照しているのに収集されて、例３はされない理由だ。

もう一回まとめると、クロージャーとは
**「ある関数Aで宣言した変数aを参照する内部関数Bを外部に引き渡した場合Aの実行コンテキストが終了された後も変数aが参照できる現象」**になる。

### returnがなくてもクロージャーが発生するケース

```javascript
// (1) setInterval / setTimeout
(function () {
  var a = 0;
  var intervalId = null;
  var inner = function () {
    if (++a >= 10) {
      clearInterval(intervalId);
    }
    console.log(a);
  };
  intervalId = setInterval(inner, 200);
})();
```

windowオブジェクトのメソッドに渡すコールバック関数内部でローカル変数を参照する。

```javascript
// (2) eventListener
(function () {
  var count = 0;
  var button = document.createElement('button');
  button.innerText = 'click';
  button.addEventListener('click', function () {
    console.log(++count, '回目');
  });
  document.body.appendChild(button);
})();
```

DOMのメソッド（addEventListener）に登録するハンドラー内部でローカル変数を参照している。

(1)と(2)両方ともローカル変数を参照する内部関数を外部に引き渡したのでクロージャーだ。

