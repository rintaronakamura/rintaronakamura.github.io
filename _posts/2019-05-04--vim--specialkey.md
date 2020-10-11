---
layout: post
title:  "特殊キーをCtrlとの組み合わせで表現する"
date:   2019-05-04 20:00:00 +0900
categories: vim
---

最近[スパルタンVim](http://files.kaoriya.net/docs/SpartanVim/SpartanVim-1.0-online.pdf)を読んでいるんですが、その中でESCなどの特殊キーはCtrlキーとの組み合わせで表現できるとのことで

``` text
ESC: Ctrl+[
TAB: Ctrl+I
Enter: Ctrl+M
BACKSPACE: Ctrl+H
```

本書では上のようなCtrlキーを使った代用が紹介されていて(もしかしたらもっとあったかも)、特にESCキーをCtrl+\[で代用することをおすすめしていました。というのも、ESCキーはキーボードの種類によってその場所が大きく異なるからだそうです。個人的にはESCキーよりもCtrl+\[の方がホームポジションが崩れないしタイピングもしやすいので代用していこうと思っています。そのほかBACKSPACEキーの代用も結構便利そうです。

ちなみに、なぜCtrl+\[がESCの代用になるのかは[端末アプリで Ctrl-\[ が Esc になる理由](http://tyru.hatenablog.com/entry/2018/10/04/151740)を参考にどうぞ。それと自分が読んでいるスパルタンVimは少し前のものらしいです。。。おそらく[これ](https://www.kaoriya.net/blog/2016/09/15/)かなと思うのですが念のため調べてみてください(丸投げ)。
