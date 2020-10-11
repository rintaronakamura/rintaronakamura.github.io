---
layout: post
title:  "Vimにインストール済みのプラグインのdoc一覧を表示する"
date:   2019-05-04 19:00:00 +0900
categories: vim
---

Vimで`:help`してヘルプを開いたら、一番下の方まで移動すると現時点でVimにインストールされているプラグインのdoc一覧が確認できる。

```bash
LOCAL ADDITIONS:				*local-additions*
|dein.txt|	Dark powered Vim/Neovim plugin manager
|airline-themes.txt|  Themes for vim-airline
|AutoClose.txt| Documentation for script AutoClose.vim
|caw.txt| Comment plugin: Operator/Dot-repeatable/300+ filetypes
|context_filetype.txt|	Context filetype library for Vim script
|dein.txt|	Dark powered Vim/Neovim plugin manager
|emmet.txt|        *Emmet* for Vim
|indentLine.txt|        Show vertical lines for indent with conceal feature
|neocomplete.txt|	Next generation of auto completion framework.
|neosnippet.txt|
|NERDTree.txt|   A tree explorer plugin that owns your momma!
|previm.txt|	Preview Plugin
|syntastic-checkers.txt|	Syntastic checkers
|winresizer.txt|  Easily resize, focus, and move your windows
```
プラグイン一覧を表示するコマンドがあったらいいなとか思って探してみたんだけど無いのかな。これを機に作成してみるのもいいかもなぁ。インストール済みのプラグインを一覧表示してdocのリンククリックするとdocが表示されるみたいなのあったら嬉しい。
