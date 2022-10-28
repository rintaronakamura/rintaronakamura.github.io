---
layout:      post
title:       Crafting Rails4 Applications 覚え書き
date:        2021-06-09
update_date: 2021-06-13
categories:  book
---

![書籍カバー]({{site.baseurl}}/assets/images/crafting_rails4_applications.png)

## この記事に書いてあること

[Crafting Rails 4 Applications Expert Practices for Everyday Rails Development](https://pragprog.com/titles/jvrails2/crafting-rails-4-applications/)を読み次のことを記載している。

1. Outline・・・各章で学んだことをざっくりとまとめている。
2. Diff・・・本書で扱われているバージョンと異なるもの(Rails6)で進めているため、参照先のソースコードが本に記載されている内容と異なることがある。そこについて記載している。
3. DevTips・・・実際に手を動かしている中で気になったことを記載している。
4. Question・・・本書を読んでいて心配なことや気になったことを記載している。

## 1. Creating Our Owner Render

Rails のレンダリング処理を読み解いていく章。

### 1.1 Creating Your First Rails Plug-in ~ 1.2 Writing the Renderer

#### Outline

[`pdf_renderer`](https://github.com/rintaronakamura/pdf_renderer) という Gemを実装することで、`render` メソッドが `:pdf` のようなオプションを渡された時に、どのような処理が行われているのかをRailsのソースコード(正確には、`actionpack`)を見ながら理解した。
具体的には、`rails/actionpack/lib/action_dispatch/http/mine_type.rb` にContentTypeに対応するMINETypeをセットにしたコードがあり、それを元に `:pdf` の対になるMINEType(application/pdf)を決めていた。

####  Diff

TODO

#### DevTips

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

### 1.3 Understanding the Rails Rendering Stack

#### Outline

[`AbstractController::Rendering#render`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/abstract_controller/rendering.rb#L21-L33) を呼び出した時のレンダリング処理がどのように走っていくのかを読み解いていく。

Rails2.3では、Viewが自動的にControllerに定義されているインスタンス変数を取得していたが、Rails3.0からは、ControllerでViewに渡すインスタンス変数をハンドリングできるようになった。
例えば、Rails2.3だとViewにインスタンス変数を取得させたくない場合、それを定義しないかレンダリングの前にそれを削除する必要があったが、Rails3.0からは [`view_assigns()`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/abstract_controller/rendering.rb#L63-L69) をオーバーライドすることで、これをハンドリングできるようになった。

```ruby
class UsersController < ApplicationController
  protected
  def view_assings
    {}
  end
end
```

レンダリング処理にフックし機能を拡張してくれるモジュールたち。
- `AbstractController::Layouts` ・・・[`ActionController::Rendering#_normalize_options`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/rendering.rb#L93-L106)をオーバーライドすることで、`:layout` オプションをサポートする(＊Diffの1を参照)
- [`ActionController::Rendering`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/rendering.rb)・・・[`AbstractController::Rendering#render`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/abstract_controller/rendering.rb#L21-L33)を[オーバーライドする](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/rendering.rb#L27-L31)ことで、[`DoubleRenderError`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/abstract_controller/rendering.rb#L9-L15) が発生するようにしている。また、[`AbstractController::Rendering#_process_options`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/abstract_controller/rendering.rb#L94-L97) を[オーバーライドする](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/rendering.rb#L116-L125)ことで `:location`,`:status`,`content_type` オプションをハンドリングできるようになる
- [`ActionController::Renderers`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/renderers.rb)・・・`:pdf` のようなキーに対応できるようにする
- [`ActionController::Instrumentation`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/instrumentation.rb)・・・[`AbstractController::Rendering#render`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/abstract_controller/rendering.rb#L21-L33)を[オーバーライドする]()ことで、レンダリング処理にかかった時間を計測できるようになる
- [`ActionController::Streaming`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/streaming.rb)・・・[`AbstractController::Rendering#_process_options`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/abstract_controller/rendering.rb#L94-L97)を[オーバーライドする](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/streaming.rb#L197-L209)ことで、適切なHTTPヘッダを設定して `:stream` オプションを処理できるようになり、また、[`ActionController::Streaming#_render_template`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/streaming.rb#L211-L218) によってテンプレートがストリームを処理できるようになる。

#### Diff

1. `AbstractController::Layouts` が存在しない。

Rails6では、`AbstractController::Layouts#_normalize_options` は  [`ActionController::Rendering#_normalize_options`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/rendering.rb#L93-L106) に置き換わっている？

> `AbstractController::Layouts` simply overrides `_normalize_options` to support the `:layout` option.

もし置き換わっているなら `ActionController::Rendering#_normalize_options` には、`:layout` オプションに関する処理が書いてありそうだが、そのような処理はないので**別物っぽい**。

#### DevTips

1. Gemのソースコードにデバッガーを仕込んでデバッグする

本書の内容ではないが、デバッグしならがらの方が理解度上がると思ったので、[pry-byebugをインストールした](https://github.com/rintaronakamura/pdf_renderer/commit/bd684f3f5e6fabcaba2781a6d270dd4799e11d98)。
その他、デバッグしたいGemのソースコードは `bundle open` コマンドで開いて、`bundle pristine` で変更を戻していた。

#### Question

1. `AbstractController::Rendering#_process_options` は空実装で、`ActionController::Rendering#_process_options` をどのようにして呼び出しているのか。
2. [`ActionController::Streaming#_render_template`](https://github.com/rails/rails/blob/3b1f87aded6d42124e4272428447a642564c6677/actionpack/lib/action_controller/metal/streaming.rb#L211-L218) はどこから呼ばれているか。

### 1.4 Taking It to the Next Level

#### Outline

この章で実装したGem(`pdf_renderer`)は、`render_to_string()` を呼び出しいるので、`template:` を渡すことで明示的にテンプレートを指定することができることがわかった。
そこで、[実際にテスト用アプリケーションのコントローラーにテンプレートを指定したアクション(`another`)と、それ用のテストケースを追加して](https://github.com/rintaronakamura/pdf_renderer/commit/cfb3e9e8e4c683695e915f1cf9b0e8d4b8db93d8)、`:pdf` が `template:` を引数に取れることを証明した。

#### Diff

なし。

#### DevTips

なし。

#### Question

なし。

### 1.5 Wrapping Up

#### Outline

- この章でPDF用のレンダラーGemを実装した
  - この実装方法に沿えば、CSVやATOMなどもレンダリングできるようになるし、HTMLをPDFに変換するようなライブラリをラッパーしたものも実装できる。
- Railsのレンダリング処理とモジュール性の説明

#### Diff

なし。

#### DevTips

なし。

#### Question

なし。

## 参考記事

- [【勉強会報告】Crafting Rails 4 Applications読み会をはじめました＋第一回資料｜TechRacho（テックラッチョ）〜エンジニアの「？」を「！」に〜｜BPS株式会社](https://techracho.bpsinc.jp/morimorihoge/2014_03_29/16247)

## 2. Building Models with Active Model

この章では、次のことを見ていく。
- ActiveModelとそのモジュール
- オブジェクトをRailsが要求するActive Model APIに適合させる方法
- バリデータとRuby定数

### 1.1

#### Outline

####  Diff

#### DevTips

#### Question

