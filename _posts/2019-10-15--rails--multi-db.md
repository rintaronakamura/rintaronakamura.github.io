---
layout: post
title:  "[Rails] Active Record で複数のデータベースを利用する"
date:   2019-10-15 01:00:00 +0900
categories: rails
---

新規システムで、すでに運用中の別システムのDBも使いたいたくなり、調べてたところRails6から公式でマルチDBに対応していたので導入してみた。

先ずは`config/database.yml`を修正に設定を追加していく。

```yaml
# Before
default: &default
 adapter: mysql2
 encoding: utf8mb4
 pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
 database: primary
 username: primary
 password: primary

# ローカル環境
 local:
   <<: *default
   host: db
```

```yaml
# After
primary_default: &primary_default
 adapter: mysql2
 encoding: utf8mb4
 pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
 database: primary
 username: primary
 password: primary

secondary_default: &secondary_default
 adapter: mysql2
 encoding: utf8mb4
 pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
 database: secondary
 username: secondary
 password: secondary
 migrations_paths: "db/migrate/secondary"

# ローカル環境
local:
 primary:
   <<: \*primary_default
   host: db
 secondary:
   <<: \*secondary_default
   host: ec2-3-111-225-99.ap-northeast-1.compute.amazonaws.com
```
上記の例では、`primary`DBと`secondary`DBを利用している。
`migrations_paths:`はマイグレーションファイルを探索する場所をRailsに認識させるための設定。
また、`secondary`DBのデータは読み込みのみする予定で、誤って書き込みとかされたくなかったので、読込専用のmysqlユーザーを作成して指定した。


次に、接続情報を追加する。
```ruby
# app/models/application_record.rb
class ApplicationRecord < ActiveRecord::Base
 self.abstract_class = true
 connects_to database: { writing: :primary, reading: :primary }
end
```

`SecondaryApplicationRecord`モデルを定義し接続情報を追加する。
```ruby
# app/models/secondary_application_record.rb
class SecondaryApplicationRecord < ActiveRecord::Base
 self.abstract_class = true
 connects_to database: { writing: :secondary, reading: :secondary }
end
```
`secondary`DBのモデルを定義するときは、この`SecondaryApplicationRecord`を継承したモデルを定義する。

例えば、`secondary`DBに保存されているDogsテーブルのレコードが取得したい場合はこんな感じ。
```ruby
# app/models/dog
class Dog < SecondaryApplicationRecord

  def self.get(id)
    Dog.find_by(id: id, test_flg: Settings.test_flg)
  end

end
```

その他、マルチDBにしたことでrailsのdb関連のコマンドの使用方法も変わってくるので注意が必要。その辺の細かいとこはRailsガイドを参考。

## schema.rbが最新化されない!?

マルチDBを導入して初めて、マイグレーションファイルを実行した。
`$ rails db:migrate:primary`
＊マルチDBにしているため、適用するDBを指定している。

実行後、DBを確認するとちゃんとマイグレーション情報が適用されていた。
しかし、困ったことにprimary_schema.rb(schema.rb)が最新化されていない。versionが以前のまま。。。
調べたところ、[公式サイト](https://railsguides.jp/active_record_migrations.html)に記載が有った。
>db:migrateタスクを実行すると、db:schema:dumpコマンドも同時に呼び出される点にご注意ください。このコマンドはdb/schema.rbスキーマファイルを更新し、スキーマがデータベースの構造に一致するようにします。

とのことだったので、`rails db:schema:dump`を実行してみた。
すると、無事にprimary_schema.rbが最新のバージョンに更新された。

本来、migrateコマンドを実行すると、dumpコマンドも実行されるはずなんだけど、今回はされなかった。マルチDBだからなんかあるのかな。

## 参考サイト
- [Rails ガイド | Active Record で複数のデータベース利用](https://railsguides.jp/active_record_multiple_databases.html#%E3%82%B3%E3%83%8D%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E8%87%AA%E5%8B%95%E5%88%87%E3%82%8A%E6%9B%BF%E3%81%88%E3%82%92%E6%9C%89%E5%8A%B9%E3%81%AB%E3%81%99%E3%82%8B)
- [Rails ガイド | Active Record マイグレーション](https://railsguides.jp/active_record_migrations.html)
- [Rails 6でマルチDBの設定を実運用に乗せてみました](https://tech.ga-tech.co.jp/entry/2019/06/rails-multi-db)
