---
layout: post
title:  "Core JavaScript 1.データタイプ"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)という書籍を要約した内容になります。

# 第１章 データタイプ

基本型と参照型のタイプがどうやって動作するのか理解しましょうー！

## データタイプの種類

JavaScriptのデータタイプには
基本型（プリミティブ型）primitive typeと参照型 reference typeの２種がある。

- Data type
  - Primitive Type
    - Nunber
    - String
      - null
      - undefined
      - Symbol（ES6）
  - Reference Type
    - Object
      - Array
      - Function
      - Date
      - RegExp
      - Map, WeakMap（ES6）
      - Set, WeakSet（ES6）

一般的に基本型は割り当てまたは演算が行われると値のアドレスを直接コピーするが、参照型は複数の値を束ねたアドレスをコピーするのが一番大きい違いである。

基本型は不変性を持っている。この不変性を理解するためにはまず識別子と変数を区別する必要がある。

## 背景知識

### メモリ

コンピューターはデータをビットで記憶する。メモリはそのビット単位で構成されていて、固有な識別子(Unique Identifier)を使って位置を確認することができる。ビットを適切に集めてデータを保存することで効率よくメモリ全体を活用することができる。

C/C++みたいな静的タイプ言語はメモリを最適に使うためタイプごとにメモリサイズを２byte、4byteなどの単位で制限している。例え 2バイト整数(short)は `-32768 ~ +32767`のみを許容する。
だがJavaScriptのNumberの場合、整数・[浮動少数](https://itmanabi.com/fixed-floating/)など区別せず64ビットのメモリ空間を確保する。

### 識別子と変数

変数はデータであって、数字・文字列・オブジェクト・配列などを保存することができる。識別子はそのデータを区別するため使われる名前、すなわち変数名である。

## 変数宣言と割り当て

### 変数宣言

以下のようにメモリの任意の空間を確保して`a`という識別子を指定する。

```javascript
var a;
```

アドレス | ... | 1002 | 1003 | 1004 | 1005 | ...
---|---|---|---|---|---|---
データ|||名前: a<br>値: |||

### データ割り当て

```javascript
var a;               //変数aを宣言
a = 'abc';           //変数aにデータ割り当て

var a = 'abc';        //宣言と割り当てをワンライナーで表現
```
宣言と割り当ては2行に分けて命令しても、ワンライナーで命令してもJavaScriptエンジンは同じ動作をする。変数宣言した空間に値を直接保存することではなく、データだけを保存するための別途の空間を確保し'abc'を保存してそのアドレスを変数領域に保存する仕組みになっている。

#### 変数領域

アドレス | ... | 1002 | 1003 | 1004 | 1005 | ...
---|---|---|---|---|---|---
データ|||名前: a<br>値: @5004 |||

#### データ領域

アドレス | ... | 5002 | 5003 | 5004 | 5005 | ...
---|---|---|---|---|---|---
データ||||'abc'||

##### フロー

1. 変数領域から空いている空間（@1003）を確保する
2. 確保した空間の識別子を `a`と指定する
3. データ領域の空いている空間（@5004）に `'abc'`を保存する
4. 変数領域からaという識別子を検索する（@1003）
5. 保存した文字列のアドレス（@5004）を@1003に代入する

なぜ値を直接代入せずもうワンステップを挟んでいるのか。それはデータを自由に変更できるようにため、かつメモリを効率よく管理するためである。

JavaScriptのNumberは8バイトの空間を確保するが、文字列はサイズに関する規格がない。また仮に前もって確保した空間内でのみデータ変換が可能だとしたら確保された空間のサイズを変更されるデータタイプに合わせて増やさないといけなくなる。その分演算量が増えることになり、パフォーマンスに悪影響する。

```javascript
var a = 'abc'
a = a + 'def'
```

文字列の最後に `'def'`を追加しようとすると `'abc'`の代わりに `'abcdef'`を割り当てることではなく `'abcdef'`という文字列を生成し別途保存した上でそのアドレスをaに紐づける。

アドレス | ... | 1002 | 1003 | 1004 | 1005 | ...
---|---|---|---|---|---|---
データ|||名前: a<br>値: @5005 |||
アドレス | ... | 5002 | 5003 | 5004 | 5005 | ...
データ||||'abc'|'abcdef'|

例えば、500個の変数に全て5を割り当てる場合、まず500個の空間を確保するのは不可避だがその空間ごとに5を直接割り当てようとすると 4000(500 * 8)バイトのメモリを使うことになるが、アドレスだけを保存する仕組みを使うことで 8バイトで済ませる。
