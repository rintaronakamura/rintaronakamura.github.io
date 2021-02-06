---
layout: post
title:  "[Ruby] リファクタリング サンプル集"
date:   2021-02-06 22:20:00 +0900
categories: ruby
---

## 配列 / 繰り返し処理

### inject / reduce

#### before

```ruby
def to_hex(r, g, b)
  '#' +
    r.to_s(16).rjust(2, '0') +
    g.to_s(16).rjust(2, '0') +
    b.to_s(16).rjust(2, '0')
end
```

#### after

```ruby
def to_hex(r, g, b)
  [r, g, b].inject('#') do |hex, n|
    hex + n.to_s(16).rjust(2, '0')
  end
end
```

### map / collect

#### before

```ruby
def to_ints(hex)
  r = hex[1..2]
  g = hex[3..4]
  b = hex[5..6]
  ints = []
  [r, g, b].each do |s|
    ints << s.hex
  end
  ints
end
```

#### after 1

```ruby
def to_ints(hex)
  r, g, b = hex[1..2], hex[3..4], hex[5..6]
  [r, g, b].map(&:hex)
end
```

#### after 2

範囲オブジェクトの代わりに正規表現とscanメソッドを使うことでより簡潔に。

```ruby
def to_ints(hex)
  hex.scan(/\w\w/).map(&:hex)
end
```

当たり前のように、ここまでリファクタリングできたらかっこいいよなぁ。。
