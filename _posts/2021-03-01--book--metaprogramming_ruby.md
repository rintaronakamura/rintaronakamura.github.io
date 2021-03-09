---
layout: post
title:  "メタプログラミング Ruby 第2版"
date:   2021-03-01 18:30:00 +0900
categories: book
---

# 第2章 月曜日：オブジェクトモデル

- インスタンスは複数のインスタンス変数とクラスへのリンクで構成されている。
- インスタンスのメソッドはオブジェクトのクラスに住んでいる。
  - クラスから見れば、それはインスタンスメソッドと呼ばれる。
- クラスはClassクラスのオブジェクトである。クラス名は単なる定数である。
- ClassはModuleのサブクラスである。モジュールは基本的にはメソッドをまとめたものである。それに加えてクラスは、`new`でインスタンス化したり、`supercclass`で階層構造を作ったりできる。
- 定数はファイルシステムのようなツリー状に配置されている。モジュールやクラスの名前がディレクトリ、通常の定数がファイルのようになっている。
- クラスはそれぞれBasicObjectまで続く継承チェーンを持っている。
- メソッドを呼び出すと、Rubyはレシーバーのクラスに向かって一歩右へ進み、それから継承チェーンを上へ向かって進んでいく。メソッドを発見するか継承チェーンが終わるまでそれは続く。
- クラスにモジュールをインクルードすると、そのクラスの継承チェーンの真上にモジュールが挿入される。モジュールをプリペンド(`prepend`)すると、そのクラス継承チェーンの真下にモジュールが挿入される。
- メソッドを呼び出すときには、レシーバが`self`になる。
- モジュール(あるいはクラス) を定義するときには、そのモジュールが`self`になる。
- インスタンス変数は常に`self`のインスタンス変数と見なされる。
- レシーバーを明示的に指定せずにメソッドを呼び出すと、`self`のメソッドだと見なされる。
- Refinementsはクラスコードにパッチを当てて、通常のメソッド探索をオーバライドするようなものである。ただし、Refinementsは`using`を呼び出したところから、ファイルやモジュールの定義が終わるところまでの限られた部分でのみ有効になる。

# 第3章 火曜日：メソッド

- 重複コードを削るために「動的メソッド」と「動的ディパッチ」の組み合わせと、ゴーストメソッド(正確には、動的プロキシとブランクスレートも使った)を学んだ。
  - これを実現できるのは、Rubyが動的言語であるため。
  - 静的言語では、存在しないメソッド/関数呼び出しを行うとすると、コンパイラに怒られてしまうのでプログラムの起動ができない。
  - 動的言語では、メソッドの呼び出しを取り締まるようなコンパイラ存在しないため、存在しないメソッドを呼び出しているプログラムを起動できる。
- 可能であれば動的メソッドを使い、しかたなければゴーストメソッドを使う。
  - ゴーストメソッドの問題は、それが本物のメソッドではなく、**メソッド呼び出しを途中で捕まえているだけ**という点。
    - それゆえ、実際のメソッドとは振る舞いが異なり、`Object#methods`の結果に出力されない。
  - 動的メソッドは`def`の代わりに`define_method`を使って定義をすることが異なるだけで、振る舞いは本物のメソッドと同じ。

```ruby
class Computer
  def initialize(computer_id, data_source)
    @id = computer_id
    @data_source = data_source
  end

  def mouse
    info = @data_source.get_mouse_info(@id)
    price = @data_source.get_mouse_price(@id)
    result = "Mouse: #{info} ($#{price})"
    return "* #{result}" if price >= 100
  end

  def display
    info = @data_source.get_display_info(@id)
    price = @data_source.get_display_price(@id)
    result = "Mouse: #{info} ($#{price})"
    return "* #{result}" if price >= 100
  end

  def keyboard
    info = @data_source.get_keyboard_info(@id)
    price = @data_source.get_keyboard_price(@id)
    result = "Mouse: #{info} ($#{price})"
    return "* #{result}" if price >= 100
  end
end
```

動的メソッドと動的ディスパッチを使ったサンプルコード
```ruby
class Computer
  def initialize(computer_id, data_source)
    @id = computer_id
    @data_source = data_source
    data_source.methods.grep(/^get_(.*)_info$/) { Computer.define_component $1 }
  end

  def self.define_component(name)
    define_method(name) do
      info = @data_source.send("get_#{name}_info", @id)
      price = @data_source.send("get_#{name}_price", @id)
      result = "#{name.capitalize}: #{info} ($#{price})"
      return "* #{result}" if price >= 100
      result
    end
  end
end
```

ゴーストメソッドを使ったサンプルコード
```ruby
class Computerr < BasicObject
  def initialize(computer_id, data_source)
    @id = computer_id
    @data_source = data_source
  end

  def method_missing(name, *args)
    super if !@data_source.respond_to?("get_#{name}_info")
    info = @data_source.send("get_#{name}_info", @id)
    price = @data_source.send("get_#{name}_price", @id)
    result = "#{name.capitalize}: #{info} ($#{price})"
    return "* #{result}" if price >= 100
    result
  end
end
```
`BasicObject` を継承しているのは、`Object`のインスタンスメソッドに`display`という名前のものが存在していて、`Object`を継承すると、その`display`メソッドが呼び出されてしまうから。
このように、ゴーストメソッドの名前と、継承した本物の名前が衝突する問題を避けるために、不要なメソッドはあらかじめ削除しておく。
こうした最小限のメソッドしかない状態のクラスを**ブランクスレート**と呼ぶ。

# 第4章 水曜日：ブロック

## クロージャまとめ

- ブロックとはクロージャである。
  - ブロックを使うことで、ローカル束縛を包んで、持ち運ぶことができる。
- Rubyのスコープは`class`、`module`、`def`といったスコープゲートで仕切られる。
- クロージャに関する基本的な魔術として、**フラットスコープ**というものがある。
  - フラットスコープとは、スコープゲートをメソッド呼び出しで置き換え、現在の束縛をクロージャで包み、そのクロージャをメソッドに渡すことで、スコープゲートを飛び越えることができる。
  - `class`は`Class.new`と、`module`は`Module.new`と`def`は`Module.define_method`と置き換えることができる。
- **共有スコープ**とは、同じフラットスコープに複数のメソッドを定義して、スコープゲートで守ることで、束縛を共有する方法である。

フラットスコープ1
```ruby
my_var = '成功'

class MyClass
  # my_varをここに表示したい

  def my_method
    # my_varをここに表示したい
  end
end

# スコープゲート(`class`)をメソッド(`Class.new`)に置き換える。
my_var = '成功'

MyClass = Class.new do
  puts "クラス定義の中は#{my_var}！"

  def my_method
    # my_varをここに表示したい
  end
end

# スコープゲート(`method`)をメソッド(`define_method`)に置き換える。
my_var = '成功'

MyClass = Class.new do
  puts "クラス定義の中は#{my_var}！"

  define_method :my_method do
    "メソッド定義の中も#{my_var}！"
  end
end

puts Myclass.new.my_method
# => クラス定義の中は成功！
#    メソッド定義の中も成功！
```

共有スコープ
```ruby
def define_methods
  shared = 0

  Kernel.send :define_method, :counter do
    shared
  end

  Kernel.send :define_method, :inc do |x|
    shared += x
  end
end

define_methods

counter # => 0
inc(4)
counter # => 4
```
