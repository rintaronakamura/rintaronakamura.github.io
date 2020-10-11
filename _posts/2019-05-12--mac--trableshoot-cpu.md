---
layout: post
title:  "MacのCPU使用率が下がらない"
date:   2019-05-12 10:00:00 +0900
categories: mac
---

MacのCPU使用率が下がらなくなり、それの対応をしたので残しとく(サポートに問い合わせる時にも使えるので)。

長年使ってきたMacBook Air(CPUが第5世代)が動作しなくなったので最新のMacBook Air(CPUが第8世代)に買い換えた。
事前に作成しておいたバックアップをもとに復元を開始、初め完了まで35時間とか出てげんなりしたけど最終的に2,3時間程度で無事に復元が完了。
この時CPU使用率を確認すると、起動したばかりもあって結構使われていたが、いつも通り少し経てば下がるだろうと思っていた。
しかし、1時間ほど経っても一向に下がらないので、初期不良かもと不安に煽られつつ確認してみると「bird」と「icloudd」というプロセスのCPU利用率が80%と大半を占めていることを知り、Googleで調べてみると、[「cloudd」はicloud同期をするためのものらしく](https://books.google.co.jp/books?id=giiKDwAAQBAJ&pg=PA44&lpg=PA44&dq=cloudd+CPU&source=bl&ots=FaZz-04epf&sig=ACfU3U1i1-8vtqsyU92jRvG6OHkZd9ISvw&hl=ja&sa=X&ved=2ahUKEwixoIW1p5PiAhWrwosBHZoECbQQ6AEwD3oECAUQAQ#v=onepage&q=cloudd%20CPU&f=false)、今回PCを買い換えた事でicloud同期が盛んになり使用率が一時的に上がっていたようで、しばらく放置したらプロセスが消えた。
他方「bird」は、調べた感じどうやらこちらもicloud関連のプロセスらしいが、「bird cpu使用率 下がらない」で[検索結果に出てきた記事](https://mac-tegaki.com/trouble-shooting/you-suspect-icloud-if-you-feel-that-mac-s-behavior-has-been-delayed.html)を読む限り、こっちの場合は放置するだけでは解決しなさそうだったので、記事に書いてある通り「システム環境設定」>「iCloud」>「iCloud Drive」を無効に。
すると、CPU使用率は0%になった。しかし、何故かメモリに35MBくらい存在したので終了することに(よく分からないものを終了するのに抵抗はあったが。。。)。
この対応ではiCloud DriveをMacで使えなくなってしまうので、後日Apple Supportを受けることにしたので、その時の内容を後で追記する。

追記 2019/05/12

Appleサポートに問い合わせたところ、iCloudの同期が初回だったため一時的に「bird」も上がっていたかもしれないとのこと、過去にも同じような問い合わせが数件あったらしいが、すでにOSアップデートで対応している。
とりあえず、今回は無効にしたiCloud Driveをもう一度有効にし、clouddとbirdのプロセスが落ち着くか確認するために1時間ほど待つことにした。

結果、3時間ほどで同期が完了したらしくclouddとbirdのCPU使用率はそれぞれ0%と22%まで下がったので無事解決できてそう。初期不良とかじゃなくて良かったなと、また復元で3時間待ちとか嫌だもんね。今回は以上！
