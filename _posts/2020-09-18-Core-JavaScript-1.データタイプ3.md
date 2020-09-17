---
layout: post
title: "Core JavaScript 1.データタイプ - undefinedとnull"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)を要約した内容になります。

## undefinedとnull

undefinedとnullはどちらも「何もない」状態を示す値である。

undefinedはJavaScriptエンジンが勝手に付与する場合がある。

### undefinedを付与する場合

```javascript
var a;
console.log(a); // (1) undefined. 値を代入していない変数のため

var obj = { a: 1 };
console.log(obj.a); // 1
console.log(obj.b); // (2)  undefined. 存在しないプロパーティのため
console.log(b); // ReferenceError: b is not defined

var func = function() {};
var c = func();
console.log(c);  // (3)  undefined. returnがないと、undefinedが返却されたことになる
```

（１）の何も代入していない場合、配列だとまた異なる振る舞いをする。

### undefinedと配列

```javascript
var arr1 = [];

arr1.length = 3;
console.log(arr1); // [empty x 3]

var arr2 = new Array(3);
console.log(arr2); // [empty x 3]

var arr3 = [undefined, undefined, undefined];
console.log(arr3); // [undefined, undefined, undefined]
```

`arr1`と`arr2`は`[empty x 3]`が出力される。配列に3つの要素を確保するだけでは各要素にはnullもundefinedも割り当てられていない。
リテラルで宣言した`arr3`は、配列を生成する共にundefinedを各要素に付与したので、同然ながらundefinedと出力される。

`undefined`を割り当てた要素と違って`空いている要素`はループ関連メソッドの巡回対象から除外される。

### 空いている要素と配列の巡回

```javascript
var arr1 = [undefined, 1];
var arr2 = [];
arr2[1] = 1;

console.log(arr2[0]) //undefined
console.log(arr2[99]) //undefined

arr1.forEach(function (v, i) { console.log(v, i); });  //undefined 0 / 1 1
arr2.forEach(function (v, i) { console.log(v, i); });  // 1 1

arr1.map(function (v, i) { return v + i; });  //[NaN, 2]
arr2.map(function (v, i) { return v + i; });  //[empty, 2]
```

arr1は`undefined`と`1`を直接割り当てていて、arr2はインデックス１に`1`を割り当てている。
arr1は配列の全ての要素を取り出して結果を出力するが、arr2は空いている要素を飛ばしてしまい、インデックス1の要素だけだ出力される。

配列は必ず指定したlength分メモリ空間を確保すると思われがちだが、
オブジェクトのプリパーティと同じく、特定のインデックスに値を割り当てることでデータのアドレスを保存する等の動作をする。

undefinedは「何もない」ことを示す**値**なので、そのアドレスが要素の変数領域に保存されるので、繰り返しメソッドの巡回対象になる。
なのでメモリ上に存在するデータだが、JavaScriptエンジンが返すundefinedは値ないないことを示すだけだ。

undefinedにはこのようなクセがあるため、直接undefinedを割り当てるのは避けるべきだ。
JavaScriptユーザーは明視的に「ない」を値として使いたい場合はnullを使えば良い。
そうすることで、undefinedはJavaScrptエンジンが返却する値として扱うことができる。

### nullとundefinedの比較

等値演算子（`==`）で比較すると同じ値だと判定してtrueが返る。
同値演算子（`===`）で比較するとタイプまで厳密に比較するのでfalseが返る。

```javascript
var n = null;
console.log(n == undefined); //true
console.log(n === undefined); //false
```
