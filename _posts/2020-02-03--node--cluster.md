---
layout: post
title:  "Nuxt.js ×Node.js (Express) で クラスタ化したかった話し"
date:   2020-02-03 03:00:00 +0900
categories: node
---

Express を serverMiddleware として導入していた Nuxt.js のプロジェクトで、Node.js の[clusterモジュール](https://nodejs.org/api/cluster.html#cluster_cluster)を導入できるのか調査した。
node が1プロセスしか動かないので、CPU が複数個あっても1プロセスしか動かない。N個でプロセスを動かして欲しいならclustor化が必要。

結論を言うと、clusterモジュールを導入するには serverMiddleware のままじゃダメで、**Node.jsのクラスタモジュールを導入するには、Node.jsサーバーのミドルウェアとしてNuxt.jsを使うという構図にする必要があった。**

実装は、[eggplanetio / nuxt-cluster](https://github.com/eggplanetio/nuxt-cluster)が参考になる。

ポイントとしては、`app.use(nuxt.render)` とする事で Nuxt.js を Express のミドルウェアとして利用することができる。
- [API: Nuxt.js をプログラムで使う - NuxtJS](https://ja.nuxtjs.org/api/nuxt/)
- [API: nuxt.render(req, res) - NuxtJS](https://ja.nuxtjs.org/api/nuxt-render/)

## 参考サイト
- [フロントエンドエンジニア (実稼働まで) ひとりでできるもん](https://speakerdeck.com/gyarasu/hurontoendoenzinia-shi-jia-dong-made-hitoridedekirumon?slide=20)
- [【Node.js+Express】Clusterモジュールでマルチスレッド化 - Qiita](https://qiita.com/mkeisuke/items/76229aec7c4d513a1d2f)
- [Node.js の Cluster モジュールを使って Express サーバを並列化する - Corredor](http://neos21.hatenablog.com/entry/2019/04/18/080000)
- [node.js:clusterモジュールを使ってMySQLに接続する雛形](https://mayer.jp.net/?p=5313)
- [Node.js clusterモジュールについて - kakts-log](http://kakts-tec.hatenablog.com/entry/2017/01/11/023735)
- [Node.js 日本ユーザグループ Blog: Cluster](http://blog.nodejs.jp/2011/11/cluster.html)
- [Node.js徹底攻略 ─ ヤフーのノウハウに学ぶ、パフォーマンス劣化やコールバック地獄との戦い方 - エンジニアHub｜若手Webエンジニアのキャリアを考える！](https://employment.en-japan.com/engineerhub/entry/2019/08/08/103000)
- [Node.jsのClusterをセットアップして、処理を並列化・高速化する | POSTD](https://postd.cc/setting-up-a-node-js-cluster/)
- [はじめてのNode.js：マルチプロセスアプリケーションを作成する | OSDN Magazine](https://mag.osdn.jp/13/04/23/090000)
- [ExpressのミドルでNuxt.jsを利用](https://www.wakuwakubank.com/posts/666-nuxtjs-express-middle/)
- [Node.js v13.7.0 Documentation](https://nodejs.org/api/cluster.html#cluster_cluster)
- [API: Nuxt.js をプログラムで使う - NuxtJS](https://ja.nuxtjs.org/api/nuxt/)
- [API: nuxt.render(req, res) - NuxtJS](https://ja.nuxtjs.org/api/nuxt-render/)
- [eggplanetio / nuxt-cluster](https://github.com/eggplanetio/nuxt-cluster)
