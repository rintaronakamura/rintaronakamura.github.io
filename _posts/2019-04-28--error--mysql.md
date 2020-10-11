---
layout: post
title:  "Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2) / (38) の対応"
date:   2019-04-28 17:00:00 +0900
categories: error
---

## はじめに

railsのdbを後からmysqlに変更しようとした。
https://qiita.com/reeenapi/items/9fc38c4f2f8186c78288

## mysql gem のインストールができない
https://qiita.com/thunders/items/101c6b329830fb1fb27d

```
An error occurred while installing mysql2 (0.4.5), and Bundler cannot continue.
Make sure that `gem install mysql2 -v '0.4.5'` succeeds before bundling.
         run  bundle exec spring binstub --all
Could not find gem 'mysql2 (< 0.5, >= 0.3.18)' in any of the gem sources listed in your Gemfile.
Run `bundle install` to install missing gems.

$ bundle config --local build.mysql2 
     "--with-ldflags=-L/usr/local/opt/openssl/lib --with-cppflags=-I/usr/local/opt/openssl/include"
```

## mysql が使えない

```
$ bundle exec mysql -u root
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
$ touch /tmp/mysql.sock
$ bundle exec mysql -u root
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (38)
```

config/database.ymlのsocketで指定してるとこと合ってるか確認
(socket: を省略している場合は設定ファイルをみにいく https://qiita.com/mdew150/items/f32d190b1e58bc9ac57d)

```
$ bundle exec mysql_config --socket
/tmp/mysql.sock
```

設定ファイルの場所を確認

```
$ mysql --help | grep my.cnf
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf
```

my.cnfの読み込まれる順序は、Linux環境では以下の順
https://qiita.com/yoheiW@github/items/bcbcd11e89bfc7d7f3ff

ファイル名	目的
/etc/my.cnf	グローバルオプション
/etc/mysql/my.cnf	グローバルオプション
SYSCONFDIR/my.cnf	グローバルオプション
$MYSQL_HOME/my.cnf	サーバー固有のオプション
defaults-extra-file	--defaults-extra-file=path によって指定されるファイル (ある場合)
~/.my.cnf	ユーザー固有のオプション
~/.mylogin.cnf	ログインパスオプション


自分の場合
/usr/local/etc/my.cnf を次のように編集した。
これで無事使えるようになった。

```
# Default Homebrew MySQL server config
[mysqld]
# Only allow connections from localhost
bind-address = 127.0.0.1
# socket  = /var/lib/mysql/mysql.sock
socket  = /tmp/mysql.sock
```

## 参考サイト

https://qiita.com/thunders/items/101c6b329830fb1fb27d
https://qiita.com/reeenapi/items/9fc38c4f2f8186c78288
https://qiita.com/mdew150/items/f32d190b1e58bc9ac57d
https://qiita.com/yoheiW@github/items/bcbcd11e89bfc7d7f3ff
https://dotinstall.com/lessons/basic_mysql_v2/7403
https://yoku0825.blogspot.com/2015/03/mysql-576set-password-password-syntax.html
