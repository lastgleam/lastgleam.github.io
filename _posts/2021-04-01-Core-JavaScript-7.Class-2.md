---
layout: post
title: "Core JavaScript #16 Chap.07 Class - 2"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)を要約した内容になります。

## 目次

1. JavaScriptにおけるクラス
1. **prototypeによるクラスの実現**
    1. **基本実装**
    1. **クラスに具体的なデータを持たせない**
    1. **コンストラクタ復旧**
    1. **上位クラスへのアクセサ提供**
1. **ES6からのクラスと継承**
1. **まとめ**

## prototypeによるクラスの実現

ES6から`class`が導入される前はどのように実現したか。

### 基本実装

多重プロトタイプチェーンを使って継承を実現する。

#### Gradeの例

```javascript
// Arrayを継承したクラスを作成する例
var Grade = function () {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};

Grade.prototype = [];

var g = new Grade(100, 80);
console.log(g.length);
console.log(g.filter(function (v) { return v > 90; }));
console.log(g.every(function (v) { return v > 50; }));
```

プロトタイプのチェーンを連結させるだけで継承が実現できる。
だが、上の例は、`length`プロパーティが削除できる（configurableな状態である）ということと`Grade.prototype`が空の配列を参照しているということに問題がある。

```javascript
g.push(90);
console.log(g); // Grade: { 0: 100, 1:80, 2:90, length:3 }

delete g.length;
g.push(70);
console.log(g); // Grade: { 0: 70, 1:80, 2:90, length:1 }
```

`length`を削除したことで`g.length`が`g.__proto__.length`を参照することになった。
`push`で1個追加されたので、`g.length`は0→1になる。

#### RectangleとSquareの例

```javascript
var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};

Rectangle.prototype.getArea = function () {
  return this.width * this.height;
};

var rect = new Rectangle(3, 4);
console.log(rect.getArea()); // 12

var Square = function (width) {
  this.width = width;
};

Square.prototype.getArea = function () {
  return this.width * this.width;
}

var sq = new Square(5);
console.log(sq.getArea()); // 25
```

そもそも正方形は長方形に対して四つの辺の長さが等しいという条件を追加された概念なので
長方形を親クラス、正方形を子クラスとして抽象化することができる。

```javascript
// Rectangleのコンストラクタをcallする
var Square = function (width) {
  Rectangle.call(this, width, width);
};
Square.prototype = new Rectangle();

var sq = new Square(5); // Rectangle { width:5, height:5, getArea: f() }
console.log(sq.getArea()); // 25
```

上の例もクラスの値の変更がインスタンスに影響を与えてしまう問題を抱えている。
`delete`で`sq.width`を削除して`Square.prototype.width`に`4`を割り当てるとgetAreaが`20`を返すことになる。

```javascript
delete sq.width;
Square.prototype.width = 4;
// sq.widthがundefinedのためsq.__proto__.widthを参照するため
// NaNが返される想定だったが4 * 5 = 20が返される
console.log(sq.getArea());
```

### クラスに具体的なデータを持たせない

クラス（prototype）の値の変更がインスタンスに影響を与えてしまう問題の対策として、
シンプルにprototypeのプロパーティを削除し凍結される。

```javascript
delete Square.prototype.width;
delete Square.prototype.height;
Object.freeze(Square.prototype);
```

2つ目の方法として、空のコンストラクタ（Bridge）を作成し
そのprototypeは親クラスのprototypeを参照するようにして
子クラスのprototypeにはBridgeのインスタンスを割り当てる方法がある。

```javascript
var Bridge = function () {};
Bridge.prototype = Rectangle.prototype;
Square.prototype = new Bridge();
Object.freeze(Square.prototype);
```

`Square.prototype`が`Rectangle.prototype`を参照しつつ
`Square.prototype`に不要なインスタンスプロパーティが存在ないようにすることが重要だ。

### コンストラクタの復旧

SquareのコンストラクタがRectangleを参照しているのでSquare.prototype.constructorがSquareを参照するようにする

```javascript
Square.prototype.constructor = Square;
```

これで継承と抽象化の実現が一通りできるようになった。

### 上位クラスへのアクセサ提供

親クラスのメソッドを呼び出す`super`を追加する。

```javascript
Square.prototype.super = function (propName) {
  var self = this;
  if (!propName) {
    return function () {
      Rectangle.apply(self, arguments);
    }
  }
  var prop = Rectangle.prototype[propName];
  if (typeof prop !== 'function') {
    return prop;
  }
  return function () {
    prop.apply(self, arguments);
  }
}

var sq = new Square(10);
console.log(sq.getArea());          // 100
console.log(sq.super('getArea')()); // 100
```

## ES6からのクラスと継承

ES6（ECMAScript2015）からは`class`文法が正式導入された。
ただし、ES6のクラスもprototypeで実現されたのでJavaScriptがプロトタイプベース言語であることに変わりはない。

> JavaScriptのclass文法はあくまでプロトタイプを使って他の言語のクラスに相当することを実現するためのシンタックスシュガー(糖衣構文)に過ぎません。[^1]

```javascript
//---------ES6以前----------
var ES5 = function (name) {
  this.name = name;
};
ES5.staticMethod = function () {
  return this.name + ' staticMethod';
}

ES5.prototype.method = function () {
  return this.name = + ' method';
}

var es5Instance = new ES5('es5');
console.log(ES5.staticMethod()); // es5 staticMethod
console.log(es5Instance.method()); // es5 method

//---------ES6以降----------
var ES6 = class {
  constructor (name) {
    this.name = name;
  }

  static staticMethod () {
    return this.name + ' staticMethod';
  }

  method () {
    return this.name + ' method';
  }
};

var es6Instance = new ES6('es6');
console.log(ES6.staticMethod()); // es6 staticMethod
console.log(es6Instance.method()); // es6 method
```

継承も簡単に実現できる

```javascript
var Rectangle = class {
  constructor (width, height) {
    this.width = width;
    this.height = height;
  }
  getArea () {
    return this.width * this.height;
  }
};

var Square = class extends Rectangle {
  constructor (width) {
    super(width, width);
  }
  getArea () {
    // superをオブジェクトとして使用することも可能
    // Rectangle.prototypeを参照することになる
    console.log('size is ', super.getArea());
  }
};

var sq = new Square(10);
sq.getArea(); // size is 100
```

## まとめ

- JavaScriptにはそもそもクラスと継承の概念が存在しない。
- プロトタイプメソッドとスタティックメソッド
  - プロトタイプ内部に定義されたメソッドはプロトタイプメソッドと呼ばれ、インスタンスから呼び出すこともできる。
  - クラス（コンストラクタ関数）に定義したメソッドはスタティックメソッドと呼ばれ、インスタンスから呼び出すことができない。
- 継承を実現する方法
  1. SubClass.prototypeにSuperClassのインスタンスを割り当てる。
  1.
     - その後SubClass.prototypeから不要なプロパーティを削除する
     - または空のコンストラクタ（Bridge）のprototypeにSuperClassのprototypeを参照させてSubClass.prototypeにBridgeのインスタンスを割り当てる
  1. SubClass.prototype.constructorがSuperClassでなくSubClassを参照するようにする。
  1. superで親クラスのメソッドを呼び出すようにする

## 参考

- [継承とプロトタイプチェーン](https://developer.mozilla.org/ja/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

[^1]: [Google流JavaScriptにおけるクラス定義の実現方法(ES6以前)](https://www.yunabe.jp/docs/javascript_class_in_google.html) から抜粋
