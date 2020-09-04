---
layout: post
title: "Core JavaScript 1.データタイプ - 基本型データと参照型データ"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)を要約した内容になります。

## 基本型データ参照型データ

### 不変値

変数と定数を区分する性質は「変更可能性」で、その対象となるのはめもりの「変数領域」である。また、不変性を区分する対象は「データ領域」だ。

基本型の数字、文字列、null、undefined、Symbolはすべて不変値である。

```javascript
var a = 'abc';
a = a + 'def';

var b = 5;
var c = 5;
b = 7;
```

- １・２行目
  - `'abc'`を`'abcdef'`に変えるのではなく新しい文字列`'abcdef'`を作ってそのアドレスを`a`に保存する。
- ４行目
  - `b`に`5`を割り当てる。データ領域からまず`5`があるかどうかを確認して、なければデータ空間を作って保存する。
- ５行目
  - また`5`を別の変数に割り当てているが４行目ですでに保存しといた`5`があるのでそれを再利用する。
- ６行目
  - `5`を`7`に変えるのではなく`7`を別の空間に保存してそのアドレスを`b`に保存する。

このように変更は必ず新しい値を作って代入する振る舞いをする。これがすなわち「不変性」である。
（一度作られた値はガベージコレクタによって開放されないうちには絶対に変わらない）

## 可変値

基本型が不変性を持っていることに対して参照型データは基本的に可変値である。
ただし、場合によっては変更不可なこともあるし、不変値として扱うこともある。

```javascript
var obj1 = {
  a: 1,
  b: 'bbb'
};
```

変数領域

アドレス | ... | 1001 | 1002 | 1003 | 1004 | ...
---|---|---|---|---|---|---
データ| | |名前: obj1<br/> 値: @5001 ||||

データ領域

アドレス | ... | 5001 | 5002 | 5003 | 5004 | ...
---|---|---|---|---|---|---
データ||@7103 ~ ? ||1|'bbb'||

@5001の変数領域

アドレス | ... | 7103 | 7104 | 7105 | 7106 | ...
---|---|---|---|---|---|---
データ| | 名前: a<br/> 値: @5003 | 名前: b<br/> 値: @5004|||

オブジェクトの変数（プロパーティ）領域が別途存在するため、プロパーティはいくらでも変えることができる。
ということで参照型データは**可変値**（Immutable）だと言える。

### プロパーティの再割り当て

```javascript
var obj1 = {
  a: 1,
  b: 'bbb'
};
obj1.a = 2;
```

５行目で `a`に`2`を割り当てようとしている。
データ領域に`2`が存在しないため、任意の空間に`2`を保存しそのアドレスを`@7103`に保存する。
この振る舞いによってobj1がアドレスは`@5001`のままで変わらない。
新しいオブジェクトが作られたわけではなく、オブジェクト内部の値だけが変わった結果になる。

### ネストされた参照型データのプロパーティ割り当て

```javascript
var obj = {
  x: 3,
  arr: [3, 4, 5]
}
```

変数領域

アドレス | 1001 | 1002 | 1003 | 1004 | 1005 | ...
---|---|---|---|---|---|---
データ| |名前: obj<br/> 値: @5001 ||||

データ領域

アドレス | 5001 | 5002 | 5003 | 5004 | 5005 | ...
---|---|---|---|---|---|---
データ| @7103 ~ ? | 3 | @8104 ~ ? | 4 | 5 | |

オブジェクト@5001の変数領域

アドレス | 7103 | 7104 | ...
---|---|---|---|---|---
データ | 名前: x<br/> 値: @5002 | 名前: arr<br/> 値: @5003 ||

オブジェクト@5003の変数領域

アドレス | 8104 | 8105 | 8106 | ...
---|---|---|---|---
データ | 名前: 0<br/> 値: @5002 | 名前: 1<br/> 値: @5004| 名前: 2<br/> 値: @5005 |

`obj.arr[1]`を検索しようとすると以下のような手順でデータを取得することになる

```text
@1002 -> @5001 -> (@7103 ~ ?) -> @7104 -> @5003 -> (@8104 ~ ?) -> @8105 -> @5004 -> 4を返却
```

### 変数コピー比較

```javascript
var a = 10;
var b = a;

var obj1 = { c: 10, d: 'ddd' };
var obj2 = obj1;

b = 15;
obj2.c = 20;

console.log(a); //10
console.log(b); //15
console.log(obj1); //{ c: 20, d: 'ddd' }
console.log(obj2); //{ c: 20, d: 'ddd' }
console.log(a === b); //false
console.log(obj1 === obj2); //true
```

変数に値を割り当てるには「基本型は値を、参照型はアドレスをコピーする」と知られているが、厳密に言うとどのタイプの値であってもアドレスをコピーする。
ただし、「参照型は可変値だ」という時の「可変」は参照型データ自体を変更する場合ではなく内部のプロパーティを変更する時のみ成り立つ命題だ。

```javascript
var obj1 = {a: 1, b: 'abc'};
var obj2 = obj1;
console.log(obj1 === obj2); //true
obj1 = {a: 1, b: 'abc'};
console.log(obj1 === obj2); //false
```

このように`obj1`に同じプロパーティを持つオブジェクトを割り当てると `obj1`と`obj2`は違うアドレスを指しているので`===`演算の結果は`false`になる。

## 不変なオブジェクト

渡されたオブジェクトを変更するが元のオブジェクトはそのままにしたい場合がけっこう頻繁にある。
参照型データのプロパーティを変更したい場合、新しいオブジェクトを生成するように規則を作ったりライブラリ（e.g.,immutable.js）を活用することで不変性を確保することができる。

```javascript
var player = {
  name: 'Lionel Messi',
  club: 'FC Barcelona'
};

var changeClub = function (player, newClub) {
  var newPlayer = user;
  newPlayer.club = newClub;
  return newPlayer;
}

var player2 = changeClub (player, 'Machester City F.C.');

console.log(`AS-IS: ${player.club}, TO-BE: ${player2.club}`);
//AS-IS: Machester City F.C., TO-BE: Machester City F.C.
```

参照型データのプロパーティは可変性を持つため、`Lionel Messi`の履歴が前職も現職も`Machester City F.C.`になってしまった。

### 問題解決（１）：オブジェクト生成

新しいオブジェクトを生成するようにハードコーディングしよう。

```javascript
var player = {
  name: 'Lionel Messi',
  club: 'FC Barcelona'
};

var changeClub = function (player, newClub) {
  return {
    name: user.name,
    club: newClub
  };
}

var player2 = changeClub (player, 'Machester City F.C.');

console.log(`AS-IS: ${player.club}, TO-BE: ${player2.club}`);
//AS-IS: FC Barcelona, TO-BE: Machester City F.C.
```

この方法はプロパーティの数が多ければ多いほど入力する手間がかかるし、可読性が下がるので望ましくない。

### 浅いコピー

まずは浅いコピーを使ってオブジェクトをコピーしプロパーティを変える。

```javascript
var player = {
  name: 'Lionel Messi',
  club: 'FC Barcelona'
};

var copyObject = function (target) {
  var result = {};
  for (var prop in target) {
    result[prop] = target[prop];
  }
  return result;
}

var player2 = copyObject(player);
player2.club = 'Machester City F.C.';

console.log(`AS-IS: ${player.club}, TO-BE: ${player2.club}`);
//AS-IS: FC Barcelona, TO-BE: Machester City F.C.
```

### 問題解決（２）：深いコピー

浅いコピーの致命的な問題は、コピーする時参照型データが持つプロパーティのアドレスだけをコピーすることにある。
下のコードでは`player.status.expiry`を変えようとしている。

```javascript
var player = {
  name: 'Lionel Messi',
  status: {
    age: 33
    expiry: 'June 2021'
  }
};

var player2 = copyObject(player);
player.status.expiry = 'September 2020';

console.log(`AS-IS: ${player.status.expiry}, TO-BE: ${player2.status.expiry}`);
//AS-IS: September 2020, TO-BE: September 2020
```

また元のデータを変更してしまう結果になった。
この不具合を防ぐには、参照型データが持つ内部の参照型データまでコピーするようにしないといけない。

```javascript
var copyObjectDeep = function(target) {
  var result = {};
  if (typeof target === 'object' && target !== null) {
    for (var prop in target) {
      result[prop] = copyObjectDeep(target[prop]);
    }
  } else {
    result = target;
  }
  return result;
}
```

このように再帰でオブジェクトの内部プローパティまでコピーすると、`player`と`player2`が共有するプロパーティがないため、互い影響しなくなる。

```javascript
var player2 = copyObjectDeep(player);
player.status.expiry = 'September 2020';

console.log(`AS-IS: ${player.status.expiry}, TO-BE: ${player2.status.expiry}`);
//AS-IS: June 2021, TO-BE: September 2020
```

また、`JSON`を使って深いコピーする方法もある。

```javascript
var copyObjectViaJSON = function (target) {
  return JSON.parse(JSON.stringify(target));
}
var player2 = copyObjectViaJSON(player);
player.status.expiry = 'September 2020';

console.log(`AS-IS: ${player.status.expiry}, TO-BE: ${player2.status.expiry}`);
//AS-IS: June 2021, TO-BE: September 2020
```
