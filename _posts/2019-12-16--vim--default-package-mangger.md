---
layout: post
title:  "[Vim] プラグインマネージャーからVim8標準のパッケージ機能に移行した話し"
date:   2019-12-16 02:00:00 +0900
categories: vim
---

みなさんVimのプラグイン管理はどうされていますか。

プラグイン管理として良く使われているサードパーティー製のものとしては、

|プラグイン名|GitHubスター数|
|:---:|:---:|
|[Vundle.vim](https://github.com/VundleVim/Vundle.vim)|20,500|
|[vim-plug](https://github.com/junegunn/vim-plug)|16,700|
|[dein.vim](https://github.com/Shougo/dein.vim)|2,400|
|[neobundle.vim](https://github.com/Shougo/neobundle.vim)|2,200|

辺りでしょうか。

自身はneobundle.vimを使っていて、1年程前にdein.vimへと移行しました。

dein.vimへと移行した時は、前職の先輩から貰ったneobundle.vimの設定ファイルをメンテもせず使い続けていたある日、vim起動時にプラグイの読み込みエラーが出るので、設定ファイルを修正していると、ファイルには記載されているが全く使っていないプラグインや設定を見つけ、きちんと管理(どのプラグインがインストールされ、どんな設定がされているのか把握している状態)したくなったのがきっかけでした。数あるプラグインマネージャーの中からdein.vimを選んだのは、それまで使っていたneobundle.vimの後継的な存在だったからです。

自身のdein.vimを使ったプラグイン管理構成をすごくざっくり書くとこんな感じでした。

- [`dein.toml`](https://github.com/rintaronakamura/dotfiles/commit/822aee9283d13b7bd46f7b31df3c6777a5745e7a#diff-edc7c12c9ac528a03ebe0fb206adc425)
  - インストールしたいプラグインと、そのプラグインの設定を記載
- [`.vimrc`](https://github.com/rintaronakamura/dotfiles/commit/822aee9283d13b7bd46f7b31df3c6777a5745e7a#diff-4e3abbeec5214ad1cc706a4616e2a28f)
  - dein.vimのインストール
  - dein.tomlに記載されているプラグインを読み込みまたは、インストールするための設定を追記

そして、この構成で1年間程やってきて、プラグインマネージャーほどの多機能さは必要ないなと思うようになったため、本記事のタイトルの通り、プラグインマネージャー(dein.vim)を辞めVim8標準のパッケージ機能を採用することにしました。

前置きが長くなりましたが、以降でVim8標準のパッケージ機能でのプラグイン管理について書いて行きます。

また、ここで行っている手順は[Vim 8 時代のがんばらないプラグイン管理のすすめ](http://tyru.hatenablog.com/entry/2017/12/20/035142)を参考にしています。

## Vim8標準のパッケージ機能について

まず、Vim8標準パッケージ機能について簡潔に述べると「ディレクトリ配下に配置されているプラグインを読み込む」機能になります。

より詳しい解説は[Vim 8.0 Advent Calendar 6 日目 パッケージ](https://qiita.com/thinca/items/cdc0169e3bcc5a55a5ba)を参照ください。

こちらの記事を要約すると、
> - ~/.vim/pack/mypackage/start/myplugin/ 等にプラグインのディレクトリを置くと、起動時に runtimepath に追加されロードされる (mypackage, myplugin は任意のディレクトリ名)
> - ~/.vim/pack/mypackage/opt/myplugin/ にプラグインのディレクトリを置くと、:packadd myplugin とすることでプラグインをロードできる
>   - このディレクトリは起動時に runtimepath に追加されないので、任意のタイミングでロードできる

[Vim 8 時代のがんばらないプラグイン管理のすすめ](http://tyru.hatenablog.com/entry/2017/12/20/035142)より引用。

後者の「任意のタイミングでロードできる」がなぜ嬉しいかと言いますと、プラグインの中でも特定のファイル形式の場合にのみ使うものとかってありますよね。例えば、Vueファイルのシンタックス用のプラグインは、Vueファイルを編集するまで使わないです。

そういったものは必要になったタイミングで読み込むようにします。

というのも、Vim起動時に全てのプラグインを読み込もうとすると時間がかかり、必然的にVimの起動も遅くなってしまうからです。

なので、プラグインはできるだけ必要になるタイミングで読み込むようにします。

このようにパッケージ機能はプラグインを管理するための基本的な機能を提供してくれます。

簡単な使い方としては、指定のディレクトリ(~/.vim/pack/mypackage/start/)を作成し、そこに`git clone`でプラグインを配置しておくことで、Vim起動時にプラグインが読み込まれます。

例えば、[amatatsu](https://github.com/rintaronakamura/amatatsu)というプラグインをVim起動時に読み込みたいなら次のようになります。
```console
% mkdir -p ~/.vim/pack/rintaronakamura/start/
% cd ~/.vim/pack/rintaronakamura/start/
% git clone https://github.com/rintaronakamura/amatatsu.git
```

> そう、Vim 8 なら無理にプラグインマネージャーを使わずとも簡単にプラグインを読み込めるのです。 更新するのだって簡単で、プラグインのリポジトリで git pull すればいいだけです。

[Vim 8 時代のがんばらないプラグイン管理のすすめ](http://tyru.hatenablog.com/entry/2017/12/20/035142)より引用。

ただし、これまでの機能では以下の点においてプラグインを管理していくのに不十分です。

1. プラグインのバージョンを固定し、別PCで同じ環境を復元する
2. プラグインの一括アップデートを行う
3. プラグインの遅延読み込みを行う

以降では、これらの問題を順だって解消していきます。

## 1. プラグインのバージョンを固定し、別PCで同じ環境を復元する
まず、プラグインを特定のバージョンに固定するために次のことを行います。

> - 1-1. ~/.vim を Git で管理する
> - 1-2. プラグインを git submodule で管理する
> - 1-3. リポジトリを GitHub で公開する

[Vim 8 時代のがんばらないプラグイン管理のすすめ](http://tyru.hatenablog.com/entry/2017/12/20/035142)より引用。

### 1-1. ~/.vim を Git で管理する
`~/.vim`ディレクトリをGitで管理します。
```console
% cd ~/.vim
% git init
% git add .
% git commit -m 'initial commit.'
```
こちらに関しては既に[dotfilesで管理](https://qiita.com/okamos/items/7f5461814e8ed8916870)している方もいらっしゃるかと思います。

自身も[`dotfiles`](https://github.com/rintaronakamura/dotfiles)で`.vimrc`や`.zshrc`などの設定ファイルを管理しているので、今回も`dotfiles`配下に[`_vim`](https://github.com/rintaronakamura/dotfiles/tree/master/_vim)ディレクトリを作成し、`~/.vim`にシンボリックリンクを貼って運用しています。

### 1-2. プラグインを git submodule で管理する
「げ、出たsubmodule。。。よう分からん。」という感じだったので、[Git submodule の基礎](https://qiita.com/sotarok/items/0d525e568a6088f6f6bb)で勉強し直しました。

submoduleが何となく分かったので、プラグインをsubmoduleとして管理します。

例として、[amatatsu](https://github.com/rintaronakamura/amatatsu)というプラグインを管理してみます。**このプラグインはVim起動時に読み込みたいため、`start`ディレクトリ配下にインストールします。**

```console
% cd ~/.vim
% mkdir -p pack/rintaronakamura/start
% git submodule add https://github.com/rintaronakamura/amatatsu.git pack/rintaronakamura/start/amatatsu
% git commit -m 'add amatatsu plugin.'
```

すると、下記のようなディレクトリ構成になるかと思います。

```
~/.vim/
|- pack/
   |- rintaronakamura/
      |- start/
        |- amatatsu/
```

### 1-3. リポジトリをGitHubで公開する
終わりに、リポジトリをGitHubで公開します。

GitHubに`.vim`プロジェクトを作成し、`git push`します。

以後、このディレクトリを`clone`する際は、submoduleも一緒に`clone`します。
```
git clone --recursive { gitリポジトリurl }
```

ちなみに、`--recursive`オプションを忘れて`clone`してしまっても、後から落としてこるみたいです。

[git cloneしたあとに--recursiveを付け忘れたことに気づいた。あとからsubmoduleをcloneしたい。](http://karoten512.hatenablog.com/entry/2017/11/09/013845)

## 2. プラグインの一括アップデートを行う

次に、プラグインの一括アップデートを行いたいと思います。

上記の手順(1)でプラグインを`submodule`として管理しているため、一括アップデートは下記のように簡単に行えます。
```console
% git submodule foreach git pull
% git commit -a -m 'update all vim plugins.'
```

＊注意点
> git commit しないと次回 git clone --recursive でリポジトリを持ってきた時に古いバージョンのままなので、ちゃんとコミットしておきましょう。

[Vim 8 時代のがんばらないプラグイン管理のすすめ](http://tyru.hatenablog.com/entry/2017/12/20/035142)より引用。

## 3. プラグインの遅延読み込みを行う
はい、最後です。

まずは、Vim8の標準パッケージ機能で遅延読み込みを行うための手順を確認します。

> - 3-1. ~/.vim/pack/mypackage/opt/ ディレクトリに入れ、
> - 3-2. 任意の FileType イベントや CmdUndefined イベントで :packadd を実行します
>   - FileType イベント：ファイルタイプ設定時
>   - CmdUndefined イベント：Ex コマンド実行時にコマンドが見つからなかった場合
> すると :packadd によりプラグインがロードされます。

[Vim 8 時代のがんばらないプラグイン管理のすすめ](http://tyru.hatenablog.com/entry/2017/12/20/035142)より引用。

上記の手順に沿って、[vim-vue](https://github.com/posva/vim-vue)というプラグインをインストールし遅延読み込みしてみます。

[vim-vue](https://github.com/posva/vim-vue)は、Vueコンポーネントのシンタックスハイライトを行ってくれるプラグインです。

そのため、このプラグインは`.vue`形式のファイルを編集するタイミングで読み込んであげるのが良さそうです。

### 3-1. ~/.vim/pack/mypackage/opt/ ディレクトリにプラグインをインストールする

[vim-vue](https://github.com/posva/vim-vue)は遅延読み込みするため、**`opt`ディレクトリ配下にインストールします。**

```console
% cd ~/.vim
% mkdir -p pack/posva/opt
% git submodule add https://github.com/posva/vim-vue.git pack/posva/opt/vim-vue
% git commit -m 'add vim-vue plugin.'
```

すると、今度は下記のようなディレクトリ構成になるかと思います。

```
~/.vim/
|- pack/
   |- posva/
      |- opt/
        |- vim-vue/
```

### 3-2. 任意の FileType イベントや CmdUndefined イベントで :packadd を実行しまする
`.vue`形式のファイルが読み込まれた時に、[vim-vue](https://github.com/posva/vim-vue)が読み込まれるように`.vimrc`に設定を書いていきます。

```vim
# .vimrc
function! s:config_vue()
  packadd vim-vue
endfunction

augroup MyAutoCmd
  autocmd!
  autocmd FileType vue call s:config_vue()
augroup END
```

**豆知識**

ファイルタイプによっては定義されていないものもあります。

そういった時は、「この拡張子のファイルタイプはこれ」と明示的に設定しておいてあげると良さそうです。

例えば、下記は拡張子が`.toml`なら、ファイルタイプを`toml`として設定しています。

```vim
# .vimrc
augroup MyAutoCmd
  autocmd!
  autocmd MyAutoCmd BufRead,BufNewFile *.toml set filetype=toml
augroup END
```

ちなみに、現在編集しているファイルのfiletypeを確認するためには、下記のコマンドを実行します。

```
:set filetype?
```

[Vimメモ : filetypeの確認](https://wonderwall.hatenablog.com/entry/2016/03/20/222308)

### 最後に
まだ運用し始めたばかりで何とも言えませんが、プラグイン管理をVim8の標準機能で行えているのは、個人的にはスッキリして気持ちが良いです。少し気が遠いですが良い年を迎えられそうです笑

あとは、しばらく使って様子をみてみようかなと思います。
