---
layout: post
title: "Core JavaScript 2.実行コンテキスト"
author: "lastgleam"
---
> この記事はweb開発に欠かせないJavaScriptのコアな概念をしっかり理解しようということで[Core Javascript（韓国語）](https://wikibook.co.kr/corejs/?ckattempt=1)を要約した内容になります。

# 2. 実行コンテキスト

実行コンテキストとは、実行するコードに提供する感情情報をまとめたオブジェクトである。
JavaScriptエンジンはある実行コンテキストが活性化される時点に宣言された変数を巻き上げて（ホイスティング）外部環境情報を構成し、`this`を設定するなど動作を行う。

環境情報はコールスタックとして積み上げておいて、一番上の情報データと関連のあるコードを実行する形で全体のコードの環境と実行順を保証する。
自動的に生成されるグローバル領域とStringのJavaScriptコードを評価する`eval()`の実行、そして関数（function）を実行することでコンテキストを構成することができる。

## 実行コンテキストとコールスタック

```javascript
// ---------------------------------- (1)
var a = 1;
function outer() {
  function inner() {
    console.log(a); // undefined
    var a = 3;
  }
  inner(); // ----------------------- (2)
  console.log(a); // 1
}
outer(); // ------------------------- (3)
console.log(a);  // 1
```

このコードを実行する瞬間、まずグローバルコンテキストがコールスタックに入れ込まれる。
引数のない関数が実行された時と同じ状態になる。
コードの上から順番に読み込みながら（３）で`outer()`を実行し、outerに関する環境情報を収集しouterの実行コンテキストを生成しコールスタックに入れる。
そしてグローバルコンテキストに関わるコードの実行を一時中断され、outer内部のコードを順次実行する。
また（２）でinner関数のコンテキストがスタックに入ると、outerに関わるコードの実行が中断され、inner関数内部のコードが実行される。

inner関数で`a`に`3`を割り当ててinner関数の実行が終了されinnerのコンテキストがコールスタックから取り除かれる。
その後中断したouterコンテキストが実行され、`a`をconsole.logで出力し終了される。outerのコンテキストも取り除かれる。
最終的に（３）の下のconsole.logを実行され、コールスタックが空になる。

## 活性化された実行コンテキストの収集情報

実行コンテキストに含まれる情報には以下の3つの要素で構成されている。

- VariableEnvironment
  - 現コンテキスト内部の識別子の情報
  - 外部環境情報
  - 宣言した時点のLexicalEnvironmentnoスナップショット。
- LexicalEnvironment
  - VariableEnvironmentと一緒だが変更内容がリアルタイムで反映される。
- ThisBinding
  - 識別子が指しているオブジェクト

またVariableEnvironmentとLexicalEnvironmentの内部は`environmentRecord`と`outer-EnvironmentReference`で構成されている。
