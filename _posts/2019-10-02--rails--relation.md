---
layout: post
title:  "[Rails] モデル間の関連情報を取得し処理を分岐する"
date:   2019-10-02 03:00:00 +0900
categories: rails
---

パラメーターで渡されたテーブル名を元に、モデル間の関連情報を取得。

```ruby
def sample_method

  table_name = params_for_sample_method.delete("table_name")

  if Application.reflect_on_association("#{table_name}").is_a?(ActiveRecord::Reflection::HasOneReflection)
    application.send("build_#{table_name.singularize}(params_for_sample_method)")
  else
    application.send(table_name).build(params_for_sample_method)
  end

end
```

## 参考サイト
・[モデル間のアソシエーションの情報を取得する方法](https://tnakamura.hatenablog.com/entry/2013/04/10/204648)
・[is_a?, kind_of? (Object)](https://ref.xaio.jp/ruby/classes/object/kind_of)
・[【rails】文字列を単数形や複数形にする方法](http://g08m11.hateblo.jp/entry/2013/12/21/115929)
・[Railsモデルの関連付けで、buildを使うときのメソッド名](https://gist.github.com/seak0503/7291012e7e99fc8aca80)
