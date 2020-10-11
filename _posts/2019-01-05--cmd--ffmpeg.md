---
layout: post
title:  "ffmpegコマンドで動画圧縮"
date:   2019-01-05 17:00:00 +0900
categories: cmd
---

動画と音声を記録/変換/再生できるフリーソフトフェア、[FFmpeg](https://ffmpeg.org/)(エフエフエムペグ)を使って動画を圧縮してみました。インストールはmacOSなので[こちら](https://qiita.com/Ryosuke-Hujisawa/items/6a1c47d31ac299dc1c46)を参考にさせていただきました。
```
$ ffmpeg -i exapmle_original.mp4 -crf 10 exapmle_compressed.mp4
```
上記は圧縮元ファイルのexapmle_original.mp4を圧縮率10でexapmle_compressed.mp4として出力しています。また、下記参考サイトより圧縮率は値が大きいほど高圧縮になり、スマホで撮った動画の場合は32くらいで圧縮するとちょうどよい容量になるとのこと。(詳しく調べたら追記しますー)

[FFmpegの動画圧縮・変換コマンドの使い方 – Mankind Inc.](http://mankindinc.jp/wp01/2018/03/14/ffmpeg%E3%81%AE%E5%8B%95%E7%94%BB%E5%9C%A7%E7%B8%AE%E3%83%BB%E5%A4%89%E6%8F%9B%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9/)にはここで紹介した動画圧縮の他に「サイズ変更/動画向き変更/サウンド無効化/PinP映像化」なども紹介されています。
