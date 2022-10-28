---
layout: post
title:  "はじめてのRedis"
date:   2019-06-04 10:00:00 +0900
categories: redis
---

## 参考サイト

・[KVS（Key-Value Store）とは | TechCrowd](https://www.techcrowd.jp/nosql/kvs/)
・[KVS 超入門 - footmark](http://yuk.hatenablog.com/entry/2014/06/10/055640)
・[Redis入門: Redisとは？ インメモリDBについて、memcachedとの違いなど](http://post.simplie.jp/posts/116)
・[Ruby on Railsのセッション管理にRedisを使ってみた](https://qiita.com/takanori_yamashita/items/5d666e943a3368848da9)

## Redisとは

インメモリDBのひとつで、その名の通りメモリー上にデータを保存するタイプのDB。データがCPUから直接アクセスできるメモリー上に保存されているためRDBのようなストレージにデータを保存するオンディスクDBと比べると、非常に高速に動作する。ただし、インメモリDBは揮発性メモリにデータを保存するので、電源を切るとデータが失われてしまう。中には、スナップショットによる復元を行うことで永続性を生み出そうとしているものもあるが、完全な永続性の担保は難しい。

また、KVS(key Value Store)の1種でもある。

ざっくり言うと、Redisはメモリ上にKVSを構築することができるソフトウェア。

## Redisの利点

Redisには、他のインメモリDB(memcachedなど)と比べると、次のような特徴がある。

**1.データの永続化**

上述の通り、もとは揮発性のRedisだけど一定期間ごとなどの条件でスナップショットをとり、再起動時にメモリ上に展開することでデータの永続化を実現している。高速な読み書きに加え、データの永続化も可能。

**2.さまざまなデータ型を持ったKVS**

Redisは、単純なKVSには無いさまざまなデータ型(文字列/ハッシュ/セットなど)が存在する。
> データ型を利用することで、たとえばソートやランキングといった機能を容易に実現することができます。 永続化やデータ構造が必要であればRedis、もっとシンプルなものがよければmemcached、といったように用途に応じて使い分ければよいかなと思います。

## Redisの欠点

RedisはRDBに比べて次のような欠点がある。

**1.容量が制限される**

安価なハードディスクに比べて、メモリは高価。またその容量にも制限がある。高速な分容量が少ないので、大きなデータの保存には向かない。

**2.データ紛失の恐れ**

メモリは揮発性のため、クラッシュするとデータが失われてしまう。また、スナップショットによる永続化も可能だが原理的に完全では無い。

## Redisが適する条件

以上を踏まえると、Redisが適する条件は次のようになる。

・最悪の場合、データが失われても問題が無い
・高速な読み書きが必要

## 使ってみる

概要はこのくらいにしてさっそくRedisを使ってみる。今回はRuby on Railsのセッション管理として使う。セッション管理にRedisを使用する訳は次の2つの記事が参考になる。

・[ruby on rails - セッションを保存するとき、なぜ、Cookieではなくmemcachedやredisを使用するのでしょうか？ - スタック・オーバーフロー](https://ja.stackoverflow.com/questions/21539/%E3%82%BB%E3%83%83%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BF%9D%E5%AD%98%E3%81%99%E3%82%8B%E3%81%A8%E3%81%8D-%E3%81%AA%E3%81%9C-cookie%E3%81%A7%E3%81%AF%E3%81%AA%E3%81%8Fmemcached%E3%82%84redis%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B%E3%81%AE%E3%81%A7%E3%81%97%E3%82%87%E3%81%86%E3%81%8B)
・[Ruby on Rails - redisやmemcashedといったセッションストアの使用シーンがわかりません。(RDBではダメなんでしょうか？)(11303)｜teratail](https://teratail.com/questions/11303)

### 環境

・OS: macOS Mojave 10.14.4
・Ruby: 2.4.0
・Rails: 5.2.3
・Redis: stable 5.0.5 (bottled), HEAD
・Homebrew: 2.1.3
 ・Homebrew/homebrew-core (git revision fd1ef4; last commit 2019-05-25)
 ・Homebrew/homebrew-cask (git revision 16d50; last commit 2019-05-26)

### Redisのインストール

HomebrewでRedisをインストールする。

```
$ brew install redis
```

インストールできたかと情報を確認する。

```
$ brew info redis
```

### Redisクライアントの基本操作

Redisサーバーを起動する。

```
$ redis-server /usr/local/etc/redis.conf
86172:C 27 May 2019 18:01:23.111 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
86172:C 27 May 2019 18:01:23.111 # Redis version=5.0.5, bits=64, commit=00000000, modified=0, pid=86172, just started
86172:C 27 May 2019 18:01:23.111 # Configuration loaded
86172:M 27 May 2019 18:01:23.113 * Increased maximum number of open files to 10032 (it was originally set to 2560).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 5.0.5 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 86172
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

86172:M 27 May 2019 18:01:23.122 # Server initialized
86172:M 27 May 2019 18:01:23.122 * Ready to accept connections
```

このアスキーアートの努力の結晶感がすごい。。。

次にクライアントを起動して、基本操作を試してみる。

```
$ redis-cli

# DB選択
127.0.0.1:6379> select 1
OK

# keyとvalueをセット
127.0.0.1:6379[1]> set key val
OK

# keyを指定してvalueを取得する
127.0.0.1:6379[1]> get key
"val"

# 保存済みのkey一覧
127.0.0.1:6379[1]> keys *
1) "key"

# 保存済みのkeyの数
127.0.0.1:6379[1]> dbsize
(integer) 1

# 削除
127.0.0.1:6379[1]> flushdb
OK

# 削除されたか確認
127.0.0.1:6379[1]> dbsize
(integer) 0

# 終了
127.0.0.1:6379[1]> exit
```

### RailsアプリにRedisを導入する

まずはサンプル用のRailsアプリを作成する。

今回作成したサンプルアプリは[これ](https://github.com/rintaronakamura/redis_sample_app)。

```
$ mkdir redis_sample_app
$ cd redis_sample_app
$ bundle init
$ bundle install --path vendor/bundle
$ bundle exec rails new .
```

サンプル用のRailsアプリが作成できので、[こちら](https://qiita.com/takanori_yamashita/items/5d666e943a3368848da9#rails%E3%82%A2%E3%83%97%E3%83%AA%E3%81%ABredis%E3%82%92%E5%B0%8E%E5%85%A5)を参考にRedisをインストールする。

1.Gemfileに必要なgemを追記してインストールする

```ruby
# Gemfile

gem 'redis-rails'
```

2.development.rbに設定を追記する(下記のコードは上記のサイトから引用したもの)

```ruby
# config/environments/development.rb

Rails.application.configure do
  # (中略)
  # ↓この行を追加(6379/0の0はDBのID、expire_inはセッションの生存期間、ここでは1分に設定している)
  config.session_store :redis_store, servers: 'redis://localhost:6379/0', expire_in: 1.minutes
end
```

### Redisにセッションを保存する

1.コントローラーを作成する。

```ruby
$ bundle exec rails g controller redis
```
2.生成されたredis_controller.rbを次のように書き換える。

```ruby
# app/controllers/redis_controller.rb
class RedisController < ApplicationController
  require 'redis'

  def test
    # Rails4の場合は render text: 'テスト' かな?
    render plain: 'テスト'
    Redis.current.set("hoge", "fuga")
    # これってchromeの開発者ツールで確認できる？
    session[:user_email] = 'test@a.a'
  end
end
```

3.ルーティングを作成する

```ruby
# config/route.rb
Rails.application.routes.draw do
  match ':controller(/:action(/:id))', via: [ :get, :post, :patch ]
end
```

4.Redisの接続情報を追記する

```ruby
# config/inisitalizers/redis.rb
require 'redis'
Redis.current = Redis.new(:host => '127.0.0.1', :port => 6379)
```

これで必要な実装は完了。

Railsサーバー/Redisサーバー/Railsコンソール/Redisクライアンの4つを起動する。

Railsコンソールもしくは、ブラウザでhttp://localhost:3000/redis/testにアクセスする。

```ruby
irb(main):006:0* app.get "http://localhost:3000/redis/test"
Started GET "/redis/test" for 127.0.0.1 at 2019-05-27 19:13:52 +0900
DEPRECATION WARNING: Using a dynamic :controller segment in a route is deprecated and will be removed in Rails 6.0. (called from block in <main> at /Users/NRintaro/Documents/Program/environment/Private/RubyOnRails/redis_sample_app/config/routes.rb:3)
DEPRECATION WARNING: Using a dynamic :action segment in a route is deprecated and will be removed in Rails 6.0. (called from block in <main> at /Users/NRintaro/Documents/Program/environment/Private/RubyOnRails/redis_sample_app/config/routes.rb:3)
Processing by RedisController#test as HTML
  Rendering text template
  Rendered text template (0.0ms)
Completed 200 OK in 6ms (Views: 3.3ms | ActiveRecord: 0.0ms)


=> 200
```

最後にRedisクライアントでセッションが保存されているかを確認する。

```
127.0.0.1:6379> keys *
1) "7f5ee899f397b0f2c065209700770e7b"
2) "hoge"
```

きちんと保存されてたのでOK。今回はここまで！
