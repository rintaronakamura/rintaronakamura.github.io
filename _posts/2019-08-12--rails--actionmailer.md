---
layout: post
title:  "Railsアプリケーションでメールを送信する"
date:   2019-08-12 01:00:00 +0900
categories: rails
---

気付けば久方ぶりのブログ更新です。
今回は最近開発している[APIサービス](https://github.com/rintaronakamura/lalalalunch-api)で、アカウント登録完了時にアクセストークンをメールで送信する機能を実装したので、そのメモです。

## Action Mailer

Action Mailerを使うと簡単にメールを送ることができました。
実装したのは、メイラークラスとビュー、それからメール送信用の設定です。
メイラークラス?となりましたが、コントローラーの役割に近く感じたので、一旦そういうものだと理解しています。

ちなみに、メイラーはActionMailer::Baseを継承していて、app/mailerに配置され、app/viewsにあるビューと結び付けられます。

## メイラーを生成する

gnerateコマンドでメイラーを生成すると、ビューのディレクトリとテストも同時に作成されます。

```
$ bin/rails g mailer ConfirmationMailer
Running via Spring preloader in process 57787
  create  app/mailers/confirmation_mailer.rb
  invoke  erb
  create    app/views/confirmation_mailer
  invoke  test_unit
  create    test/mailers/confirmation_mailer_test.rb
  create    test/mailers/previews/confirmation_mailer_preview.rb
```
＊application_mailer.rb がない場合は、その辺りの必要なファイルも生成されます。

```ruby
# app/mailers/confirmation_mailer.rb
class ConfirmationMailer < ApplicationMailer
end
```

## メイラーにアクションを定義する

まずはメイラーに「アクション」と呼ばれるメソッドを定義します。
メールを送信したいコントローラーで、定義したアクションを呼び出すことでメールを送信できます。また、メールの本文は、先ほどのgenerateコマンドで生成された`app/views/confirmation_mailer`ディレクトリ配下で定義します。

今回は、アカウント登録完了時に呼び出すのでregistration_completedアクションを定義してみます。

```ruby
# app/mailers/confirmation_mailer.rb
class ConfirmationMailer < ApplicationMailer
  default from: 'lalalalunch.api@gmail.com'

  def registration_completed
    @user = params[:user]
    mail(to: @user.email, subject: '【らららランチAPI】ユーザー登録手続き完了')
  end
end
```

アクションで使える全てのオプションは[Action Mailerの全メソッド](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%81%AE%E5%85%A8%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89)に載ってます。

コントローラー同様、アクションで定義したインスタンス変数(`@user`)はビューで利用できます。

## メールの本文を定義する

`app/views/confirmation_mailer`配下に、メール本文を定義するビューファイルを作成し、本文を定義します。

```html
# app/views/confirmation_mailer/registration_completed.html.erb
<p><%= @user.userid %> 様</p>

<p>らららランチAPIをご利用いただきありがとうございます。</p>

<p>ユーザー登録が完了しました。</p>
<p>アクセスキーは</p>
<p><%= @user.access_token %></p>
<p>になります。</p>
<p>利用期限はマイページからご確認ください。</p>
```

＊ このビューファイルは`app/views/layouts/mailer.html.erb`にレンダリングされます。

```html
# app/views/layouts/mailer.html.erb
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <style>
      /* Email styles need to be inline */
    </style>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```

このレイアウトの指定は`app/mailer/application_mailer.rb`で定義されています。

```ruby
# app/views/layouts/mailer.html.erb
class ApplicationMailer < ActionMailer::Base
  default from: 'from@example.com'
  layout 'mailer'
end
```

また、HTMLに対応していない人でもメールが見れるように、テキストメールも作成しておきます。

```txt
# app/views/confirmation_mailer/registration_completed.text.erb
<%= @user.userid %> 様

らららランチAPIをご利用いただきありがとうございます。

ユーザー登録が完了しました。
アクセスキーは
<%= @user.access_token %>
になります。
利用期限はマイページからご確認ください。
```

HTML形式のメールに加え、テキスト形式のメールも作成しておくことで、mailerメソッドが2種類のテンプレートがあるか確認し[multipart/alternative形式](http://fuji3.main.jp/common/tips/mail_m_p.html)のメールを自動生成してくれる。

## メールを送信する

定義したメイラーを呼び出してメールを送信します。

```ruby
class Users::ConfirmationsController < Devise::ConfirmationsController

  ~省略~

  def after_confirmation_path_for(resource_name, resource)
    ConfirmationMailer.with(user: resource).registration_completed.deliver_later
    super(resource_name, resource)
  end
end
```

`with`で渡した値はメイラーアクション側でparams[:user]のような形で受け取ることができます。

## プレビュー確認する

Action Mailerにはプレビュー機能があり、レンダリング用のURLを開くことでメールの外観を確認することができます。
対象ファイルやメイラー自身に何らかの変更を加えると自動的に再読み込みしてレンダリングされるので、スタイル変更を画面ですぐ確認できます。
registration_completedのプレビューを確認するには、`test/mailers/previews/confirmation_mailer_preview.rb`で同じ名前のメソッドを呼び出します。
＊confirmation_mailer_preview.rb はメイラーと同時に生成されています。

```ruby
# test/mailers/previews/confirmation_mailer_preview.rb

# Preview all emails at http://localhost:3000/rails/mailers/confirmation_mailer
class ConfirmationMailerPreview < ActionMailer::Preview
  def registration_completed
    ConfirmationMailer.with(user: User.first).registration_completed
  end
end
```

registration_completedのプレビューを表示するには、同じ名前のメソッドを定義し、その中でConfirmationMailer.registration_completedを呼び出します。
http://localhost:3000/rails/mailers/confirmation_mailer にアクセスしてプレビューが確認できます。
また、利用可能なプレビューは http://localhost:3000/rails/mailers に一覧で表示されます。

## 参考サイト

・[Rasil Guide | Action Mailer の基礎](https://railsguides.jp/action_mailer_basics.html)
