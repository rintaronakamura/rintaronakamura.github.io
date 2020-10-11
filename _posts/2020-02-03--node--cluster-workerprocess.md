---
layout: post
title:  "[Node.js] clusterモジュールで各ワーカープロセス間でのメモリの共有は可能か"
date:   2020-02-03 01:00:00 +0900
categories: node
---

結論から書くと**clusterモジュールは、マスタープロセスがワーカープロセスをフォークするモデルなのでメモリの共有はできない。**

すでに同じ疑問を持った人が質問サイトに投稿してた。
[javascript - how to keep variables that share all node processes in node cluster? - Stack Overflow](https://stackoverflow.com/questions/14826349/how-to-keep-variables-that-share-all-node-processes-in-node-cluster)

ただ、プロセス間でのメッセージ(文字列やオブジェクトなど)の共有はできるらしい。
> プロセス間での変数の共有などは行われない。ただし、後述するIPC機構を利用してプロセス間でメッセージを送信し合うことは可能だ。

[はじめてのNode.js：マルチプロセスアプリケーションを作成する | OSDN Magazine](https://mag.osdn.jp/13/04/23/090000)より引用。

## 参考サイト

- [javascript - how to keep variables that share all node processes in node cluster? - Stack Overflow](https://stackoverflow.com/questions/14826349/how-to-keep-variables-that-share-all-node-processes-in-node-cluster)
- [はじめてのNode.js：マルチプロセスアプリケーションを作成する | OSDN Magazine](https://mag.osdn.jp/13/04/23/090000)
- [使った node.js ライブラリ – メモリキャッシュ周り比較 « Ooharabucyou](https://www.bucyou.net/blog/1224)
- [Webアプリケーションにおける正しいキャッシュ戦略 - Sansan Builders Box](https://buildersbox.corp-sansan.com/entry/2019/03/25/150000)
- [Webアプリケーションにおけるキャッシュ戦略の比較 - kenju's blog](https://itiskj.hatenablog.com/entry/2017/08/18/000000)
- [第1回　memcachedの基本：memcachedを知り尽くす｜gihyo.jp … 技術評論社](https://gihyo.jp/dev/feature/01/memcached/0001)
- [memcached, Redis 何をどう使う - 補習ほぼ確](https://ave-h.hateblo.jp/entry/2018/08/09/234051)
- [3rd-Eden / memcached](https://github.com/3rd-Eden/memcached)
- [手を動かして学ぶ Redis 入門 | Black Everyday Company](https://kuroeveryday.blogspot.com/2019/03/redis-get-started-quickly.html)
