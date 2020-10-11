---
layout: post
title:  "Railsガイド(5.2対応)"
date:   2019-05-26 10:00:00 +0900
categories: rails
---

最近になって、もう一度きちんとRailsについて体系的に学びたくなったので、[Railsガイド 5.2対応 電子書籍版 (EPUB/PDF)](https://railsguides.jp/options.html)を購入。Kindleに変換してそっちで読もうと思っていたのに、何故か変換すると英語になるし目次が使えないので、色々試したが結果、諦めてKindle for  macで読むことに。これならわざわざ購入しなくてもRailsガイドの公式ページ見れば良かった感が拭い切れないけど、いくつか便利な機能が使えるし良しとしたい。。。ここにはRailsガイドを読み進めるにあたり、今まで知らなかったけど重要そうなだと思った内容を記載しておく。

## 第1章 Railsをはじめよう

### 1.9 セキュリティ

#### 1.9.1 BASIC 認証

`http_basic_authenticate_with`メソッドは、特定のメソッドに認証したユーザー以外が実行できないようにする。
例えば、以下の場合ならindex,showメソッド以外のメソッドを実行するためには、認証が必要となる。

```ruby
class ArticlesController < ApplicationController

  http_basic_authenticate_with name : "dhh", password : "secret ", except : [:index , :show ]

  def index
    @articles = Article .all
  end

  # (省 略)

end
```

### 1.11 設定の落とし穴

> Rails での無用なトラブルを避けるための最も初歩的なコツは、外部データを常に UTF-8 で保存しておくことです。このとおりにしないと、Ruby ライブラリや Rails はネ イティブデータをたびたび UTF-8 に変換しなければならず、しかも場合によっては失敗 します。外部データを常に UTF-8 にしておくことをぜひお勧めします。

## 第2章 Active Record の基礎

### 2.1 Active Record について

#### 2.1.3 ORM フレームワークとしての Active Record

Active Record に搭載されている機能の中でも特に重要なもの。

> - モデルおよびモデル内のデータを表現する
> - モデル同士の関連付け (アソシエーション) を表現する
> - 関連付けられているモデル間の継承階層を表現する
> - データをデータベースで永続化する前にバリデーション (検証) を行なう
> - データベースをオブジェクト指向スタイルで操作する

### 2.2 Active RecorにおけるCoC(Convention over configure)

一般的なORMアプリケーションでは、設定のためのコードを大量に書くこと必要があったりするけど、Railsには「設定がほとんどの場合で共通ならば、その設定をアプリケーションのデフォルトにすべきである」という考えがあり、それに準拠した実装をしていれば、Active Recordモデルの作成時に必要となる設定用コードを最小限または0にすることができる。

2.2.2 スキーマのルール

Active Recordには、DBのテーブルで使うカラム名にもルールがある。
例えば、外部キーなら対応するテーブル名の単数形\_id、主キーid、レコード作成/更新時はcreated\_id/updated\_atだったり。
ここら辺よく見かけるので、馴染みがあったが、実はその他にも知っておくと便利そうなのがあったので載せとく。単一継承テーブル(STI)なんて全然知らなかったけど便利そう。

> - lock\_version: モデルに [optimistic locking](https://ja.wikipedia.org/wiki/%E6%A5%BD%E8%A6%B3%E7%9A%84%E4%B8%A6%E8%A1%8C%E6%80%A7%E5%88%B6%E5%BE%A1) を追加します
> - type: モデルで[Single Table Inheritance](https://ruby-rails.hatenadiary.com/entry/20141206/1417839458)を使う場合に指定します
> - 関連付け名\_type: ポリモーフィック関連付けの種類を保存します
> - テーブル名\_count: 関連付けにおいて、所属しているオブジェクトの数をキャッ シュするのに使われます。たとえば、Article クラスに comments_count という カラムがあり、そこに Comment のインスタンスが多数あると、ポストごとのコメ ント数がここにキャッシュされます。

注意

1. これらの予約されたカラム名は特別な理由がない限りは利用を避ける

### 2.4 命名ルールを上書きする

テーブルの主キーに使われているカラム名を上書きしたい時は、`ActiveRecord::Base.primary_key=`メソッドを使う。

```ruby
class Product < ApplicationRecord
  self.primary_key = "product_id"
end
```

### 2.5 CRUD: データの読み書き

#### 2.5.1 Create

*Question*

どうゆうこと？利点は？

> `create`や`new`にブロックを渡すと、そのブロックで初期化された新しいオブジェクトが`yield`されます。

```ruby
user = User.new do |u|
  u.name = "David"
  u.occupation = "Code Artist"
end
```

#### 2.5.3 Update

[`update_all`](https://apidock.com/rails/v4.0.2/ActiveRecord/Relation/update_all)クラスメソッドが紹介されていて、最近これを使ってハマったことがあったので注意として書いとく。

*注意点*

1. `updated_at`カラムが更新されないので注意。
2. `ActiveRecord::Relation`のクラスメソッドなのでArrayには使えない(当たり前なんだけど)。

## 第3章 Active Record マイグレーション

### 3.2 マイグレーションを作成する

#### 3.2.1 単独のマイグレーションを作成する

**マイグレーションファイルの実行順序**

下記はRailsのマイグレーションの実行順序について、知っておかないと地味にハマりそうだったのでそのまま抜き出しとく。

> Railsではマイグレーションの実行順序をファイル名のタイムスタンプで決定します。

まぁ、`generate`コマンドでマイグレーションファイルを生成してればいいんだけど。こういう仕様だよって言うのは把握しといて損はないかな。

**テーブル結合を生成するジェネレーターを作成する**

名前に`JoinTable`を含めることで、テーブル結合を生成するジェネレーターも作成できる。

```ruby
$ bin/rails g migration CreateJoinTableCustomerProduct customer product
```

生成されるマイグレーションファイル。

```ruby
class CreateJoinTableCustomerProduct < ActiveRecord::Migration[5.0]
  def change
    create_join_table :customers, :product do |t|
      # t.index [:customer_id, :product_id]
      # t.index [:product_id, :customer_id]
    end
  end
end
```

*Question*

マイグレーションファイルの名前にJoinTableを含めることで、テーブル結合を生成するジェネレーターが作成できるのを今さっきrailsガイドで読んで、よく分からないからググってみたものの 、それらしき記事が中々見つからないってことは、余り使われることがないからなのかな。都度includeやらをすれば

#### 3.2.3 修飾子を渡す

```ruby
$ bin/ rails generate migration AddDetailsToProducts 'price : decimal {5 ,2} ' supplier : references { polymorphic }
```

```ruby
class AddDetailsToProducts < ActiveRecord :: Migration [5.0]
  def change
    add_column : products , :price , : decimal , precision : 5, scale : 2 
    add_reference : products , : supplier , polymorphic : true
  end
end
```

### 3.3 マイグレーションを自作する

#### 3.3.1 テーブルを作成する

**特定のデータベースで使われるオプションを指定する**

特定のデータベースで使われるオプションが必要な場合は、`:options`オプションに続けてSQLフラグメントを記述する。
例えば、次のマイグレーションでは、テーブルを生成するSQLステートメントに`ENGINE=BLACKHOLE`を指定している。

```ruby
create_table :products, options: "engine=blackhole" do |t|
  t.string :name, null: false
end
```

**テーブル説明文を書いてデータベース自身に保存する**

`:comment`オプションを使うと、テーブルの説明文を書いてデータベース自身に保存することができる。
この機能は[Rails5から標準機能として装備](https://techracho.bpsinc.jp/hachi8833/2017_02_23/36083)されたもので、現時点では、MySQLとPostgreSQLアダプタのみがコメント機能をサポートしているみたい。
ドキュメント生成されてデータモデルを理解し易くなっていいな。データベース数が多い大規模なアプリケーションなら尚更便利そう。

マイグレーションは次のように、各カラムに対して`comment:`オプションでコメントを付与してあげる。

```ruby
class CreatePatients < ActiveRecord::Migration[5.0]
  def change
    create_table :patients do |t|
      t.string :name, comment: "患者のニックネーム"
      t.string :first_name, comment: "患者の下の名前"
      t.string :last_name, comment: "患者の名字"

      t.timestamps
    end
  end
end
```

これを`rake db:migrate`で実行すると、`db/schema.rb`に次のようにコメントが反映される。

```ruby
create_table "patients", force: :cascade do |t|
  t.string "name",                      comment: "患者の名前"
  t.string "first_name",                comment: "患者の下の名前"
  t.string "last_name",                 comment: "患者の名字"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
end
```

また、登録したコメントは[PGAdmin](https://www.pgadmin.org/)や[MySQL WorkBench](https://www.mysql.com/jp/products/workbench/)などのGUIツールでも確認できる。

#### 3.3.4 カラムを変更する

*注意*

1. `change_column`コマンドはロールバックされない(可逆的でない)

次の例は、productsテーブルのpart_numberカラムの種類を:textフィールドに変更している。

```ruby
change_column :products , :part_number , :text
```

`change_column`の他に`change_column_null`、`change_column_default`メソッドがあり、それぞれnot null制約を変更したりデフォルト値を指定したりするのに使われる。
下のマイグレーションはproductsテーブルの:nameフィールドにNOT NULL制約を設定し、:approvedフィールドのデフォルト値をtrueからfalseに変更する。

```ruby
change_column_null :products, :name, false
change_column_default :products, :approved, from: true, to: false

# 次の書き方だとマイグレーションを可逆的にできる
# change_column_default :products, :approved, false
```

#### 3.3.7 ヘルパーの機能だけでは足りない場合

Active Record が提供するヘルパーの機能だけでは不十分な場合には、`execute`メソッドで任意のSQLを実行することができる。

```ruby
Product.connection.execute (" UPDATE products SET price = 'free ' WHERE 1=1")
```
