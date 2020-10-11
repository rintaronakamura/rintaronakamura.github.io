---
layout: post
title:  "vim設定の確認"
date:   2019-05-04 17:00:00 +0900
categories: vim
---

## オプションの確認

今開いているvimのオプションを全て確認するには、`set`コマンドを使う。

```
:set
--- Options ---
  ambiwidth=double    conceallevel=2      helplang=ja,en      list                shiftwidth=2        syntax=markdown     ttymouse=xterm2
  background=dark     expandtab           hlsearch            modified            smartcase           tabstop=2           wildmenu
  clipboard=unnamed   filetype=markdown   ignorecase          number              smartindent         title
  concealcursor=inc nofixendofline        incsearch           scroll=25           softtabstop=2       ttyfast
  backspace=indent,eol,start
  comments=fb:*,fb:-,fb:+,n:>
~ 省略 ~
```

次に、特定のオプションについて有効・無効を確認したい場合には、`set {オプション}?`を使う。

指定したオプションが有効な場合。
```
:set number?
=> number
```

指定たオプションが無効な場合。
```
:set number?
=> nonumber
```

### 変数の確認

今開いているvimの変数を全て確認するには、`let`コマンドを使う。

```
:let
---------------
NERDTreeMapPreviewSplit  gi
neocomplete#sources#buffer#max_keyword_width #80
winresizer_gui_enable #0
NERDTreeMapCloseChildren  X
winresizer_horiz_resize #3
~ 省略 ~
```

### マッピングの確認

今開いているvimのマッピングを全て確認するには、`map`コマンドを使う。

```
:map
---------------
n  [c           @<Plug>GitGutterPrevHunk
n  \hp          @<Plug>GitGutterPreviewHunk
n  \hu          @<Plug>GitGutterUndoHunk
n  \hs          @<Plug>GitGutterStageHunk
n  ]c           @<Plug>GitGutterNextHunk
~ 省略 ~
```

その他、インサート用の`imap`、ノーマル用の`nmap`、ビジュアル用の`vmap`もある。

### autocmdの確認

今開いているvimのautocmdを全て確認するには、`autocmd`コマンドを使う。
どのイベントの時にどれが発火しているとか見れるので、予期せぬ挙動を確認する時に便利そう。

```
:autocmd
---------------
:autocmd
--- Auto-Commands ---
dein  BufNew
    *?        call dein#autoload#_on_default_event('BufNew')
fzf_buffers  BufDelete
    *         silent! call remove(g:fzf#vim#buffers, expand('<abuf>'))
~ 省略 ~
```

### autocmdグループの確認

今開いているvimのautocmdグループを全て確認するには、`augroup`コマンドを使う。

```
:augroup
---------------
:augroup
--- Auto-Commands ---
MyAutoCmd  dein  dein-events  filetypedetect  filetypeplugin  filetypeindent  syntaxset  fzf_popd  fzf_buffers  gitgutter  lightline  neocomplete  neosnippet  NERDTree  NERDTreeH
~ 省略 ~
```
