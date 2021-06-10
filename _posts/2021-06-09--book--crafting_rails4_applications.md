---
layout: post
title:  "Crafting Rails4 Applications 覚え書き"
date:   2021-06-09 22:00:00 +0900
categories: book
---

## この記事に書いてあること

[Crafting Rails 4 Applications Expert Practices for Everyday Rails Development](https://pragprog.com/titles/jvrails2/crafting-rails-4-applications/)を読み次のことを記載している。

1. ざっくりまとめ・・・各章で学んだことをざっくりとまとめている。
2. Tips・・・本書ではタイトルの通りRails4が使われているが、Rails6を使って開発を行うことにした。そのため、`Rails plugin new` などで自動生成されるコードに若干の違いがあったり、記載の通りではすんなり実行が通らないコマンドもあるので、そういったものをどのように解決したかを記録している。
3. Question・・・本書を読んでいて気になったことを記録している。

## 1. Creating Our Owner Render

Rails のレンダリング処理を読み解いていく章。

### 1.1 ~ 1.2

#### ざっくりまとめ


[`pdf_renderer`](https://github.com/residenti/pdf_renderer) という Gemを実装することで、`render` メソッドが `:pdf` のようなオプションを渡された時に、どのような処理が行われているのかをRailsのソースコード(正確には、`actionpack`)を見ながら理解した。
具体的には、`rails/actionpack/lib/action_dispatch/http/mine_type.rb` にContentTypeに対応するMINETypeをセットにしたコードがあり、それを元に `:pdf` の対になるMINEType(application/pdf)を決めていた。

#### Tips

1. [未解決] ダミーアプリを起動する

`test/dummy` 配下で `rails server` を実行することでサーバーが起動され、`http://localhost:3000/home.pdf` にアクセスができると記載があるが、サーバーが無いと怒られる。

```
% rails s
Could not find server "".
Run `bin/rails server --help` for more options.
% rails s -u puma
Could not load server "puma". Maybe you need to the add it to the Gemfile?

  gem "puma"

Run `bin/rails server --help` for more options.
```

#### Question

なし。

### 1.3 ~

#### ざっくりまとめ

`AbstractController::Rendering#render` を呼び出した時のレンダリング処理がどのように走っていくのかを読み解いていく。

#### Tips

1. Gemのソースコードにデバッガーを仕込んでデバッグする

本書の内容ではないが、デバッグしならがらの方が理解度上がると思ったので、[pry-byebugをインストールした](https://github.com/residenti/pdf_renderer/commit/bd684f3f5e6fabcaba2781a6d270dd4799e11d98)。
その他、デバッグしたいGemのソースコードは `bundle open` コマンドで開いて、`bundle pristine` で変更を戻していた。

#### Question

1. `AbstractController::Rendering#_process_options` は空実装で、`ActionController::Rendering#_process_options` をどのようにして呼び出しているのか。
