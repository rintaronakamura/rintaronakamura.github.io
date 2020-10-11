---
layout: post
title:  "HerokuのMySQLに接続する"
date:   2019-05-27 10:00:00 +0900
categories: heroku
---

herokuのMySQLに接続するために必要となる情報を取得する。

```bashrc
$ heroku config | grep CLEARDB_DATABASE_URL

CLEARDB_DATABASE_URL:
mysql://AAAAAAAAAAAAAA:BBBBBBBB@us-cdbr-iron-east-02.cleardb.net/heroku_CCCCCCCCCCCCCCC?reconnect=true
```

上記の情報を元にMySQLへ接続する。

```bashrc
$ mysql -u AAAAAAAAAAAAA -h us-cdbr-east-05.cleardb.net heroku_CCCCCCCCCCC -p
Enter password: BBBBBBBB
```

毎度忘れちゃうのでメモしといた。
以上。
