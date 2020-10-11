---
layout: post
title:  "[Ruby] URLを結合するときは file.join を使う"
date:   2019-07-09 01:00:00 +0900
categories: ruby
---

URLを結合する処理でバグを出しそうになったので、自戒としてまとめとく。
結論書いちゃうと、表題の通りURLを結合する時は、File.join を使うと良いよという記事。

これまでURLを結合する時は、文字列結合か式展開を使っていた。

```
base_url = "https://sample.co.jp"

# 文字列結合
base_url + "/users"
=> "https://sample.co.jp/users"

# 式展開
"#{base_url}/users"
=> "https://sample.co.jp/users"
```

で、何をやらかしたかというと`base_url`に代入されているURLが`/`で終わると勘違いし、続く文字列の先頭に`/`を付けずに結合したため不正なURLを生成していた。

```
base_url = "https://sample.co.jp"

# 文字列結合
base_url + "users"
=> "https://sample.co.jpusers"

# 式展開
"#{base_url}users"
=> "https://sample.co.jpusers"
```

気を付けていれば防げそうにも思えるけど、人間うっかりはあるし、これに気を張りながらコーディングするのは疲れてしまう。

File.join は良しなにURLを結合してくれるので、上記のことを考えずに済む。

```
File.join("https://sample.co.jp", "users")
=> "https://sample.co.jp/users"


File.join("https://sample.co.jp/", "/users")
=> "https://sample.co.jp/users"
```
## 参考サイト

・[RubyでURLの結合をするときは`File.join`使うと幸せになった](https://qiita.com/ryonext/items/0bfd2592d713211bbc2f)
