---
layout: post
title:  "ClearDBに追加したレコードのIDが10ずつ増加する理由"
date:   2019-05-04 22:00:00 +0900
categories: cleardb
---

ある日、SELECT文で記事の一覧を取得するとidの値が10ずつ増加している事に気づいた。
初めはDBの設定を間違えたかなと思っていたが、[公式サイト](https://w2.cleardb.net/faqs/#general_16)を見た限りどうやらそうではなく、利用しているClearDBの仕様らしい。
なぜ、このようにidが10ずつ増えるのかというと、[レプリケーション](https://wa3.i-3-i.info/word12869.html)時にidの重複が発生しないようにするためだとか。下記は公式サイトからの引用。

> Q. When I use auto_increment keys (or sequences) in my database, they increment by 10 with varying offsets. Why?

> A. ClearDB uses circular replication to provide master-master MySQL support. As such, certain things such as auto_increment keys (or sequences) must be configured in order for one master not to use the same key as the other, in all cases. We do this by configuring MySQL to skip certain keys, and by enforcing MySQL to use a specific offset for each key used. The reason why we use a value of 10 instead of 2 is for future development.


ちなみに、DBのidの増加値と初期値を調べたい時は次のコマンドを使う。

```sql
mysql> SHOW variables LIKE "%auto_increment%";

+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| auto_increment_increment | 10    |
| auto_increment_offset    | 2     |
+--------------------------+-------+
2 rows in set (2.76 sec)
```

idは1ずつ増えるのが当たり前だと思い込んでたけど、そうじゃなくて何かしらの理由があってそうなっているだけで、理由が変わればidの増加値もそれを満たすために変わるのか。
