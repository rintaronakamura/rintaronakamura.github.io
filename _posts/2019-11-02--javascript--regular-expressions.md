---
layout: post
title:  "[JavaScript] 正規表現"
date:   2019-11-02 01:00:00 +0900
categories: javascript
---

最近入力フォームを実装していて、バリデーションチェックAPI側だけでなく、フロント側でも行うことになり、jsで正規表現を使ってフォーマットチェックを行っていたので、その時の知識メモ。

## 決められた意味を表す文字パターン

・「.」何らかの1文字
・「\w」大文字小文字を含む英数字とアンダースコア
・「\d」0〜9の数字
・「\s」タブ、改行、スペース

例えば、「No.01」~「No.99」までの文字をマッチさせるパターンは、
```js
/No.\d\d/
```
「No.」は固定で、「01」~「99」は半角数字が2つなので「\d\d」で表せる。

次に、「中田です」や「鈴木です」は、
```js
/..です/
```
「です」は固定で、「中田」や「鈴木」の部分には何らかの文字を表す「.」を使う。

## 繰り返しを表すパターン

・「\*」0文字以上の繰り返し
・「+」1文字以上の繰り返し
・「?」1文字が「ある」か「ない」か
・「{}」繰り返す回数を指定

例えば、文字種は半角数字なんだけど桁数が決まっていない場合は、
```js
/\d*/
```
この場合、0以上にマッチするため未入力にもマッチする。
そのため、必ず1文字は必要なら「\*」の代わりに「+」を使う。
```js
/\d+/
```

郵便番号のように桁が決まっている場合は、
```js
/\d{3}-\d{4}/
```
と「{}」で文字数を指定することが可能。
また、自宅電話番号のように桁数が特定の回数ではなく特定の範囲の場合は、
```js
/\d{2,4}-\d{4}-\d{4}
```
「00-0000-0000」、「000-0000-0000」、「0000-0000-0000」などにマッチする。

次に、「1 time」~ 「9 times」の末尾の「s」のように任意の文字が存在するケースと存在しないケースがある場合は、
```js
/\d\stimes?/
```
任意の文字の後ろに「?」を指定することでマッチできる。

## 文字の位置を表すパターン

・「\^」行の先頭
・「$」行の末尾

上で「No.01」~「No.99」にマッチするパターンを
```js
/No.\d{2}/
```
と表たが、実はこのままだと「hogeNo.01」や「No.01hoge」もマッチする。
「No.xx」という単語だけにマッチさせたい場合は、
```js
/^No.\d{2}$/
```

## 任意の文字列を表すパターン

・「[]」一連の文字セットを表す
・「()」文字パターンをグループ化する

先ほど郵便番号の正規表を次のように表たが、
```js
/^\d{3}-\d{4}$/
```
「-」だけでなく「ー」や「–」も許可したい場合は「[]」を使い次のように表せる。
```js
/^\d{3}[-ー–]\d{4}$
```
「[]」は他にも、「[0-9」と書くと「\d」と同じ意味になったりする。

次に、先ほどの「1 time」~ 「9 times」の例は、「()」を使うことで次のようにも表せる。
```js
/^\d\s(time|times)$/
```