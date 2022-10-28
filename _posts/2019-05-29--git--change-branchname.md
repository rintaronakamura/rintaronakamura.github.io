---
layout: post
title:  "[Git] Macのターミナルで現在のブランチ名を表示する"
date:   2019-05-29 10:00:00 +0900
categories: git
---

## はじめに

カレントブランチを確認するために、毎度コマンドを打つのが面倒になってきたので、タイトルの通りターミナルにカレントブランチ名を表示するようにした。

手順を説明する前に、変更前後の違いは下記の通り。

< 変更前 >
```
rintaronakamura@lime:Blog $
```

< 変更後 >
```
rintaronakamura@lime:Blog (master) $
```

## 参考サイト

・[MacのターミナルでGitのブランチ名を表示する](http://blog.ruedap.com/entry/20110706/mac_terminal_git_branch_name)
・[bash\_completionで「-bash:\_\_gitps1: command not found」となった時の対処法](https://sue445.hatenablog.com/entry/2012/08/30/005627)
・[BashでPromptの色を変更する方法](https://qiita.com/wildeagle/items/5da17e007e2c284dc5dd)

## 手順

1.必要なソースを配置する

[ここから](https://github.com/git/git/tree/master/contrib/completion)次の2つのファイルを~/Desktopでもどこでもいいので配置する。

・git-prompt.sh
・git-completion.bash

2.`.bashrc`に設定を書き込む

```.bashrc
    ・
    ・
    ・
# git settings
source ~/dotfiles/git-prompt.sh
source ~/dotfiles/git-completion.bash
GIT_PS1_SHOWDIRTYSTATE=true
export PS1='\u@\h:\W\[\033[33m\]$(__git_ps1)\[\033[00m\] \$ '
```

＊Bashを使っているので`git-completion.bash`を配置しているが、Zshユーザーなら`git-completion.zsh`を配置するなど、適宜読み換える。

これで.barchを読み込み直すと設定が反映され、現在のブランチ名が表示されるはず。
また、設定で書き込んだ最後の1文(`export PS1=・・・`)にある`PS1`はBashのPromptの書式に関する情報を保持するShell変数で、ここをいじると表示されフォーマットを変更することができる。
