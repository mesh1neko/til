# CSSセレクタについてざっくり
## セレクタとは
とてもざっくりいうとCSS内でよくみる
```
h2 { color : #000080; }
↑
これ
↓
#sample p { line-width : 150%; }
```

まじめにいうと
* DOMのツリー構造の要素にマッチするパターン
* XML文書からノードを選択する手法の１つ

※HTMLやXMLとの利用に最適化されているらしい。

CSSにおけるセレクタとは、ドキュメント内の要素に対して、
スタイルやプロパティを関連付ける手段となっている。

マッチするかどうかのパターンゆえ、以下のように関数で定義される

　式(expression) * 要素(element) -> 真偽値(boolean)

cf. (https://www.w3.org/TR/2011/REC-css3-selectors-20110929/)


## 用語
* タグ：文書内で境界を表します。
* 要素：タグで区切られた文書内を構成する部品です。
* 属性：要素に情報を付与します。

## わかりやすいセレクタ
### 型セレクタ
要素名をそのまま書いて指定　※該当する要素すべてに適用される.
```
p {
    color:red;
}
```

### 全称セレクタ
*で指定　全要素に適用する.
```
* {
    color:blue;
}
```
### idセレクタ
接頭に#をつけて指定　該当するid属性をもつ要素に適用する.
```
#hoge {
    color:yellow;
}
```

### classセレクタ
接頭に.をつけて指定　該当するclass属性をもつ要素に適用する.
```
.piyo {
    color:black;
}
/* 特定の要素のクラス属性に限定する場合... */
p.piyo {
    color:red;
}
```
#### 子孫セレクタ
要素と要素を （空白）で区切って指定　ある要素の子孫要素に適用する.

※子孫なので、子だけでなく孫も対象となる.
```
/* divタグ内'の'aタグ */
div a {
    color:blue;
}
```
#### 子セレクタ
要素と要素を>で区切って指定　ある要素の子要素だけに適用する.
```
.hoge > span {
    color:yellow;
}
```


### セレクタのグループ化
カンマ(,)で区切ることで、複数の要素にマッチするようグループ化できる。
```
h1 { font-family: sans-serif }
h2 { font-family: sans-serif }
h3 { font-family: sans-serif }
/* どちらも同義 */
h1, h2, h3 { font-family:sans-serif}
```
