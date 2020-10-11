---
layout: post
title:  "[Rails] ネストしたStrong Paramtersの書式"
date:   2019-10-02 02:00:00 +0900
categories: rails
---

「あれ？ネストしてるStrong Paramtersってどうやって書くんだっけ？」みたいなのが、起きるのでメモしとく。

## ネスト無し
```json
{
  "name": "Jhon",
  "email": "sample@email.com"
}
```

```ruby
def params_for_create
  params.permit(:name, :email)
end
```

## ネストが1回

### オブジェクト
```json
{
  "name": "Jhon",
  "plan": {
    "name": "自動車保険"
    "price": 1900
  }
}
```

```ruby
def params_for_create
  params.permit(:name, plan: [:name, :price])
end
```

### 配列
```
{
  "name": "Jhon",
  "insureds": [
    {
      "name": "Alice"
    },
    {
      "name": "Bob"
    }
  ]
}
```

```ruby
def params_for_create
  params.permit(:name, insureds: [:name])
end
```

ちなみに、配列の中身がオブジェクトではなく数字や文字列が直接並ぶ場合は次の通り。

```
{
  "name": "Jhon",
  "ids": [1, 2, 3]
}
```

```ruby
def params_for_create
  params.permit(:name, ids: [])
end
```

## 参考サイト
・[ネストするStrong Parametersの書きかた](https://qiita.com/kymmt90/items/4ce8618ca8f537b2ef7e)
・[Rails5 Strong Parametersで複数の配列を許可する](https://qiita.com/sayama0402/items/576345a1f7d37571f5a6)
