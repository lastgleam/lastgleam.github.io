---
layout: post
title: "Core JavaScript 4.コールバック関数（2）"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)を要約した内容になります。

# 4. コールバック関数

## 目次

1. コールバック関数とは
2. 制御権
3. **コールバック関数は関数だ**
4. **コールバック内部のthisにbindする方法**
5. **コールバック地獄と非同期制御**

## 4-3.コールバック関数は関数だ

- コールバック関数としてあるオブジェクトのメソッドを引き渡してもそのメソッドはメソッドでなく関数として呼び出される。

```javascript
var obj = {
  vals: [1, 2, 3],
  logValues: function (v, i) {
    console.log(this, v, i);
  }
};
obj.logValues(1, 2);
// { vals: [1, 2, 3], logValues: f } 1 2
[4, 5, 6].forEach(obj.logValues);
// window { ... } 4 0
// window { ... } 5 1
// window { ... } 6 2
```

`this`が指すオブジェクトが呼び出し元である`obj`から`window`に変わった。

コールバック関数として呼び出される場合には単純に関数として渡されるのでthisは**グローバルオブジェクト**を指すことになる。

## 4-4.コールバック内部のthisにbindする方法

コールバック関数でも`this`を元のオブジェクトにしたい場合がある。

### 4-4-1.伝統的な方法

```javascript
var obj1 = {
  name: 'obj1',
  func: function () {
    var self = this;
    return function () {
      console.log(self.name);
    };
  }
};
var callback = obj1.func();
setTimeout(callback, 1000);
```

`self`変数に`this`を割り当てて、無名関数を宣言・返却することで`obj1`を出力することに成功した。
が、この方法は実際に`this`を使っているわけでもないし、コードが冗長になる。

### 4-4-2.func関数の使い回し

```javascript
var obj1 = {
  name: 'obj1',
  func: function () {
    var self = this;
    return function () {
      console.log(self.name);
    };
  }
};
var obj2 = {
  name: 'obj2',
  func: obj1.func
};
var callback2 = obj2.func();
setTimeout(callback2, 1000);

var obj3 = { name: 'obj3' };
var callback3 = obj1.func.call(obj3);
setTimeout(callback3, 2000);
```

1秒後に`obj2`が、2秒後に`obj3`が出力される。
これで少しでも簡潔なコードにすることができるが、`bind`メソッドを使用するとさらに簡潔なコードにすることができる。
（コードの見通しとパフォーマンス両方改善される。）

```javascript
var obj1 = {
  name: 'obj1',
  func: function () {
    console.log(this.name);
  }
};
setTimeout(obj1.func.bind(obj1), 1000);
var obj2 = { name: 'obj2' };
setTimeout(obj1.func.bind(obj2), 2000);
var obj3 = { name: 'obj3' };
setTimeout(obj1.func.bind(obj3), 3000);
```

## 4-5.コールバック地獄と非同期制御

### 4-5-1.コールバック地獄（Callback hell）とは

![callback-hell](/assets/images/callback-hell.jpeg)

> 関数の入れ子が積み重なっていき、コードがどんどん読みにくくなっていく様

JavaScriptではイベント処理や通信作業を行うコードがコールバック地獄になりがち。
コードが読みにくくなるし、それに伴って修正もしにくくなる。

### 4-5-2.非同期処理

- 同期処理
  - 現在実行中のコードが完了してから次のコードを実行する。
- 非同期処理
  - 現在実行中のコードの完了有無と関係なく次のコードを実行させる。

### 4-5-3.JavaScriptの代表的な非同期処理

- setTimeout
  - 指定された時間が経過するまで処理を保留する
- addEventListener
  - 特定のイベントが発生するまで処理を保留する
- XMLHttpRequest
  - リクエストに対する応答が来るまで処理を保留する

#### 4-5-4.コールバック地獄の例

コーヒーの名前をリストに追加し出力する。

```javascript
setTimeout(function (name) {
  var coffeeList = name;
  console.log(coffeeList);
  setTimeout(function (name) {
    coffeeList += ', ' + name;
    console.log(coffeeList);
    setTimeout(function (name) {
      coffeeList += ', ' + name;
      console.log(coffeeList);
      setTimeout(function (name) {
        coffeeList += ', ' + name;
        console.log(coffeeList);
      }, 500, 'ココア');
    }, 500, 'フラペチーノ');
  }, 500, 'ドリップコーヒー');
}, 500, 'コールドブリュー');
```

ネストが余計に深くなってしまったし値が渡される順番が逆順になっていて分かりにくい。

#### 4-5-5. 解決方法1 - 名前付き関数に変換

```javascript
var coffeeList = '';

var addColdBrew = function (name) {
  coffeeList = name;
  console.log(coffeeList);
  setTimeout(addDripCoffee, 500, 'ドリップコーヒー');
};

var addDripCoffee = function (name) {
  coffeeList += ', ' + name;
  console.log(coffeeList);
  setTimeout(addFrappuccino, 500, 'フラペチーノ');
};

var addFrappuccino = function (name) {
  coffeeList += ', ' + name;
  console.log(coffeeList);
  setTimeout(addCocoa, 500, 'ココア');
};

var addCocoa = function (name) {
  coffeeList += ', ' + name;
  console.log(coffeeList);
};

setTimeout(addColdBrew, 500, 'コールドブリュー');
```

コードの見通しが少し改善されたが再利用できない関数を複数定義してしまい、
あまり望ましくないコードになってしまった。

ES6で導入された`Promise`と、ES2017で導入された`async/await`を活用してみよう。

### 4-5-6. 非同期作業の同期的な表現： Promise

```javascript
new Promise(function (resolve) {
  setTimeout(function () {
    var name = 'コールドブリュー';
    console.log(name);
    resolve(name);
  }, 500);
}).then(function (prevName) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      var name = prevName + ', ドリップコーヒー';
      console.log(name);
      resolve(name);
    }, 500);
  });
}).then(function (prevName) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      var name = prevName + ', フラペチーノ';
      console.log(name);
      resolve(name);
    }, 500);
  });
}).then(function (prevName) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      var name = prevName + ', ココア';
      console.log(name);
      resolve(name);
    }, 500);
  });
});
```

`Promise`は`resolve`か`reject`が呼び出されるまで`then`と`catch`に進まない。
非同期作業を完了してから`resolve`を呼び出すことで同期的な表現をすることができる。

共通処理を関数化して短くすると以下のようになる。
（2行目と3行目で登場したクロージャー次の章で説明する予定）

```javascript
var addCoffee = function (name) {
  return function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var newName = prevName ? (prevName + ', ' + name) : name;
        console.log(newName);
        resolve(newName);
      }, 500);
    });
  };
};
addCoffee('コールドブリュー')()
  .then(addCoffee('ドリップコーヒー'))
  .then(addCoffee('フラペチーノ'))
  .then(addCoffee('ココア'));
```

### 4-5-6. 非同期作業の同期的な表現： Promise + async/await

`Promise`のさらに進化した機能。
非同期処理を実行する関数に`async`を、必要な位置に`await`をつけことで
`Promise`を生成し`then`で紐付けるような効果を得ることができる。

```javascript
var addCoffee = function (name) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(name);
    }, 500);
  });
}

var coffeeMaker = async function () {
  var coffeeList = '';
  var _addCoffee = async function (name) {
    coffeeList += (coffeeList ? ', ' : '') + await addCoffee(name);
  };
  await _addCoffee('コールドブリュー');
  console.log(coffeeList);
  await _addCoffee('ドリップコーヒー');
  console.log(coffeeList);
  await _addCoffee('フラペチーノ');
  console.log(coffeeList);
  await _addCoffee('ココア');
  console.log(coffeeList);
};
coffeeMaker();
```

## 4-6. まとめ

- コールバック関数
  - 他のコードに引数で引き渡すことで制御権も共に移譲する関数
- 特徴
  - コールバック関数を引き受けたコードの中で実行するタイミングを決める。
  - コールバック関数のパラメータはすでに決まっているので注意を（Array.prototype.mapの例）。
  - オブジェクトのメソッドを渡しても関数として実行される。
  - そのため、グロバールオブジェクトがセットされるが`bind`などを使って変えることもできる。
- コールバック地獄
  - 非同期制御のためコールバック関数を使いすぎると現れる現象。
  - `Promise`や`async/await`などを使うことで回避できる。

## 出典・参考文献

- 非同期処理:コールバック/Promise/Async Function. JavaScript Primer
  - https://jsprimer.net/basic/async/
- JavaScriptとコールバック地獄. Yahoo! JAPAN Tech Blog
  - https://techblog.yahoo.co.jp/programming/js_callback/
- Async Flow: From callback hell to promise to Async-Await. Quyet Vu
  - https://medium.com/@quyetvv/async-flow-from-callback-hell-to-promise-to-async-await-2da3ecfff997


　

