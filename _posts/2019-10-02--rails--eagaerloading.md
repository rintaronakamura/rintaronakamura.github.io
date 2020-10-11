---
layout: post
title:  "[Rails] 孫のテーブルまでegaer loadingする"
date:   2019-10-02 01:00:00 +0900
categories: rails
---

初めて親から孫をegaer loadingしたのでメモ。
「users(親) - posts(子) - comments(孫)」といったテーブル構造で、親のレコードを取得しインスタンスに詰めた後、モデルの関連付けを利用して子(post)と孫(comment)のレコードを取得する時に、効率的なSQLが発行されるよう親レコードの取得の際にeager loadingでテーブルをjoinしておく。

```ruby
# user.rb
class User < ActiveRecord::Base
  has_many :posts
end

# post.rb
class Post < ActiveRecord::Base
  belongs_to :user
  has_many :comments
end

# comment.rb
class comment < ActiveRecord::Base
  belongs_to :post
end
```

```ruby
user = User.find(1).includes(posts: [:comments])

posts = user.posts
comments = user.posts.comments
```

発行されるSQL
```sql

```

## 参考サイト
・[ActiveRecordのjoinsとpreloadとincludesとeager_loadの違い](https://qiita.com/k0kubun/items/80c5a5494f53bb88dc58)
