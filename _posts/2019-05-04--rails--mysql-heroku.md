---
layout: post
title:  "Rail5×MySQL×Heroku"
date:   2019-05-04 23:00:00 +0900
categories: rails
---

## はじめに

最近[Rails5で作成したアプリ](https://github.com/rintaronakamura/Blog)をHerokuにデプロイした。
本記事には、次回同じような作業をする際にまた調べるのが面倒なので、その際の手順を記載しておく。
ちなみに表題の通りDBにはMySQLを使ってる。

## 環境

OS: MacOS Mojave 10.14.4
Rails: 5.2.3
Ruby: 2.4.0p0 (2016-12-24 revision 57164) [x86_64-darwin18]
Git: 2.20.1 (Apple Git-117)
Heroku: 7.22.10 darwin-x64 node-v11.10.1

その他GemのバージョンなどはGitHubに作成したアプリのソースを上げているので、それの[Gemfile.lock](https://github.com/rintaronakamura/Blog/blob/master/Gemfile.lock)などを参考

## Herokuへデプロイ手順

### Herokuにターミナルでログイン

```
$ heroku login
```

上のコマンドを叩くとブラウザでHerokuのログイン画面が表示されてそこからログインした。

### Heroku上にアプリを登録

```
$ heroku create rintaronakamura-blog
```

下記のコマンドを叩いて登録できたか確認
```
$ git remote -v
heroku	https://git.heroku.com/rintaronakamura-blog.git (fetch)
heroku	https://git.heroku.com/rintaronakamura-blog.git (push)
origin	git@github.com:rintaronakamura/Blog.git (fetch)
origin	git@github.com:rintaronakamura/Blog.git (push)
```

### HerokuにDBを追加して設定

```
$ heroku addons:create cleardb:ignite
```

今回はMySQLを使うので、[ClearDB MySQL](https://elements.heroku.com/addons/cleardb)というアドオンをherokuに追加した。
＊HerokuのDBはデフォルトではPostgreで、MySQLを使いたいのでこんなことをしている
ちなみに、MySQLアドオンはClearDB以外にも[JawsDB MySQL](https://elements.heroku.com/addons/jawsdb)とか言うのもあって、どちらが良いの調べてみたがそこまで変わらなそうなのでとりあえずClearDBでやってみた。


DBを追加後は、環境変数の設定をした。
まず、すでに設定されている環境変数を知るには次のコマンドを叩く。

```
$ heroku config
=== rintaronakamura-blog Config Vars
CLEARDB_DATABASE_URL: mysql://username:password@hostname/db_name?reconnect=true
```

で、ここに表示されたusernameやらなんやらをherokuに設定してあげる。
注意としては、最後のDATABASE_URLはmysql2ライブラリを使っているので先頭の`mysql://`を`mysql2://`に修正する。

```
$ heroku config:add DBTABASE_USERNAME=username
$ heroku config:add DBTABASE_PASSWORD=password
$ heroku config:add DBTABASE_HOSTNAME=hostname
$ heroku config:add DATABASE_NAME=db_name
$ heroku config:add DATABASE_URL=mysql2://username:password@hostname/db_name?reconnect=true
```

それから、タイムゾーンと LANG の設定もしとく。
```
$ heroku config:add TZ=Asia/Tokyo
$ heroku config:add LANG=ja_JP.UTF-8
```

設定できてるか確認する

```
$ heroku config
=== rintaronakamura-blog Config Vars
CLEARDB_DATABASE_URL:     mysql://username:password@hostname/db_name?reconnect=true
DATABASE_USERNAME:        username
DATABASE_PASSWORD:        password
DATABASE_HOSTNAME:        hostname
DATABASE_NAME:            db_name
DATABASE_URL:             mysql2://username:password@hostname/db_name?reconnect=true
LANG:                     ja_JP.UTF-8
TZ:                       Asia/Tokyo
```

### config/database.ymlの設定

productionの設定を環境変数を使ったものに修正する。
＊自分の場合は、ソースコードをGitHubに上げていてpasswordなど他人に見られるとセキュリティ的にアウトなので、[`dotenv-rails`](https://github.com/bkeepers/dotenv)を使って環境変数を管理した。このGemを調べてたらちらほらデメリットを見かけたが開発してるの自分だけなので関係なさそうに見えた。

```yml
production:
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000
  database: <%= ENV['DATABASE_NAME'] %>
  username: <%= ENV['DATABASE_USERNAME'] %>
  password: <%= ENV['DATABASE_PASSWORD'] %>
  url: <%= ENV['DATABASE_URL'] %>
```

`dotenv-rails`の導入と使い方はすごく簡単で、まずGemをインストールして、次に、プロジェクトのカレントディレクトリに下記のように環境変数用のファイルを作成し、各ファイルに環境変数を定義するだけで、それぞれの環境の環境変数を自動で登録してくれる。
もちろん、こららのファイルは`.ignore`ファイルで除外した。

導入は[これ](http://shimadays.com/2018/07/22/railsdotenv/)が分かりやすかった。

|ファイル名 |内容 |
|---|---|
|.env.development |開発環境用の環境変数を定義 |
|.env.production |本番環境用の環境変数を定義 |
|.env.test |テスト環境用の環境変数を定義 |
|.env.local |ローカル環境用の環境変数を定義 |

### Herokuへデプロイする

Herokuに戻ってきて、リモートへプッシュ。
＊Gemfileにsqlite3書いてある場合は先に削除しとく、herokuはsqlite3対応してないのでgem install エラーになる。

```
$ git push heroku master
```

次にDBやらテーブルを作成

```
$ heroku run rake db:migrate
```

アプリを動かす

```
heroku open
```

これでOK.


### ログ内容を充実させる

デプロイ中のデバッグで`$ heroku logs -t`でログを見てたけど、やたら情報量が少ないので、[`rails_12factor`](https://github.com/heroku/rails_12factor)を入れた。導入は[これ](https://qiita.com/mm36/items/f92fe3b2484ced37a3fd)を参考にした 。ただgemをインストールするだけ。

## 参考記事

[Heroku コマンド・設定 メモメモ](https://qiita.com/pugiemonn/items/0e69b7a29a384b356e65)
[mysqlを使ったRailsアプリをHerokuにデプロイする流れ](https://qiita.com/shou1012/items/bf55a185920e717f4011)
[Railsの環境変数管理gem、dotenvの使い方徹底解説](http://shimadays.com/2018/07/22/railsdotenv/)
[Heroku で Rails4 + MySQL を動かす](http://xyk.hatenablog.com/entry/2014/09/29/184727)
[RailsアプリをHerokuにデプロイして、エラーがでたけど、１行しか出なくて困った時・・](https://qiita.com/mm36/items/f92fe3b2484ced37a3fd)

殴り書きみたいなものだけど、次やるときはこれ見ればスムーズにできるかな。
