---
layout: post
title:  "[Go] 入門で参考にしたサイト一覧"
date:   2019-12-26 01:00:00 +0900
categories: go
---

・[5分で分かるガベージコレクションの仕組み](https://geechs-magazine.com/tag/tech/20160229)
・[インタープリタ方式とコンパイル方式](https://www.cuc.ac.jp/~miyata/classes/prg1/02/2way.html)
・[go getでインストールしたパッケージを削除したい](https://teratail.com/questions/3000)
・[他言語プログラマがgolangの基本を押さえる為のまとめ](https://qiita.com/tfrcm/items/e2a3d7ce7ab8868e37f7)
・[A tour of go](https://go-tour-jp.appspot.com/welcome/1)
・[Writing Web Applications](https://golang.org/doc/articles/wiki/)
・[はじめてのGo―シンプルな言語仕様，型システム，並行処理](http://gihyo.jp/dev/feature/01/go_4beginners/0001)
・[プリプロセスの概要](http://itdoc.hitachi.co.jp/manuals/3020/3020635643/W3560227.HTM)
・[動的言語だけやってた僕が、38日間Go言語を書いて学んだこと](https://qiita.com/suin/items/22662f43b6a6e8728798)
・[udemy | 現役シリコンバレーエンジニアの15時間のGo講座](https://www.udemy.com/course/go-fintech/)

## パッケージのバージョン管理

プロジェクト作成時に迷ったのがパッケージのバージョン管理方法。
RailsのときはGemfileで依存関係を管理してインストール先はプロジェクト配下とかいうふうに管理していたけど、Goはどう管理するのが良いのか分からなかったので、goは1.11から導入されたGo Modulesというバージョン管理システムを使ってみることにした。

・[DockerでGo言語の環境構築をしてGo Modulesを試してみた](https://blog.web.nifty.com/engineer/2197)
・[Go Modulesの概要とGo1.12に含まれるModulesに関する変更 #golangjp #go112party](https://budougumi0617.github.io/2019/02/15/go-modules-on-go112/#modules%E3%81%AE%E6%A6%82%E8%A6%81)
・[Go 1.13 に向けて知っておきたい Go Modules とそれを取り巻くエコシステム](https://syfm.hatenablog.com/entry/2019/08/10/170730)
