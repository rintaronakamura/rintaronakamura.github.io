---
layout: post
title:  "[MySQL] アカウント追加と権限変更"
date:   2019-10-08 02:00:00 +0900
categories: mysql
---

とりあえずメモレベル。後で追記してく。

読み取り専用ユーザーの作成。
```sql
 -- どのホストから接続しても利用できるように'%'を指定する.
mysql> GRANT SELECT ON development.* to 'read_only_user'@'%' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT Host, User, Password FROM mysql.user;
+-----------+-------------+-------------------------------------------+
| Host      | User        | Password                                  |
+-----------+-------------+-------------------------------------------+
| %         | root           | *4235AB08DF5304B8EB842A7349D37C64B8B1EFB8 |
| %         | read_only_user | *64BB40CA633F94974EEEB2CE10D74D753EF8F8EC |
+-----------+-------------+-------------------------------------------+
5 rows in set (0.01 sec)

-- 作成したユーザーの権限を確認する.
mysql> SHOW GRANTS FOR 'read_only_user'@'%';
+-------------------------------------------------------------------------+
| Grants for read_only_user@%                                                |
+-------------------------------------------------------------------------+
| GRANT USAGE ON . TO 'read_only_user'@'%' IDENTIFIED BY PASSWORD <secret> |
| GRANT SELECT ON development.* TO 'read_only_user'@'%'                    |
+-------------------------------------------------------------------------+
2 rows in set (0.00 sec)

-- mysqlデータベース内の供与テーブルから権限を再ロードする(設定の反映).
mysql> flush privileges;
```

## 参考サイト
- [MySQL/ユーザの作成・変更・削除](https://db.just4fun.biz/?MySQL/%E3%83%A6%E3%83%BC%E3%82%B6%E3%81%AE%E4%BD%9C%E6%88%90%E3%83%BB%E5%A4%89%E6%9B%B4%E3%83%BB%E5%89%8A%E9%99%A4)
- [mysql server で外部から接続できるユーザを作成する。](https://qiita.com/yoshiokaCB/items/df4ae185be7cbc4f03ac)
