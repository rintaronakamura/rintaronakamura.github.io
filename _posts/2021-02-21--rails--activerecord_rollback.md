---
layout: post
title:  "[Rails] ActiveRecord::Base.transaction で ActiveRecord::Rollback が発生するとどうなる？？"
date:   2021-02-21 18:30:00 +0900
categories: rails
---

下記は'失敗⤵︎' が返ってくる。

```ruby
def destroy(id)
  ActiveRecord::Base.transaction do
    raise 'ほげ'
  end

  render json: { message: '成功!!' }
rescue => e
  render json: { message: '失敗⤵︎' }
end
```

下記の場合は `transaction` の中で `ActiveRecord::Rollback` が捕捉されるため、「'成功!!'」が返ってくる。

```ruby
def destroy(id)
  ActiveRecord::Base.transaction do
    raise ActiveRecord::Rollback
  end

  render json: { message: '成功!!' }
rescue => e
  render json: { message: '失敗⤵︎' }
end
```

[参考ソースコード](https://github.com/rails/rails/blob/c4d3e202e10ae627b3b9c34498afb45450652421/activerecord/lib/active_record/connection_adapters/abstract/database_statements.rb#L225-L236)
