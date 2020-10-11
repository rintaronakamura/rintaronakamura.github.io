---
layout: post
title:  "[Go version 1.13] パッケージ管理で悩んだ話し"
date:   2019-12-28 01:00:00 +0900
categories: go
---

golangを始めてからおよそひと月が経った。  
公式のチュートリアルを参考に基礎文法と習得と簡単なウェブアプリの実装を経験し、なんとなくgolangに慣れてきた。  
そこでこの1ヶ月間の集大成として、golangを使い自前でAPIの開発をしてみることに。  
ものとしては特定の時間ごとの仮想通貨の値段をDBに保存しつつ、データをJSON形式で返却するようなものを考えている。  

環境はDockerを使って構築することに。  
また今回はgolangとmysqlの複数コンテナを利用するためdocker-composeも利用している。  

ひとまずコンテナ内でgo run main.goするとwebサーバーが起動し、ブラウザで8080を開くとhello worldが表示されるとこまで構築できた辺りで、ファイルを修正する度に手動でビルドするのが面倒くさいことに気がついた。  
きっとホットリロードができるなんかいい感じのがあるはず。探しがしてみると[realize](https://github.com/oxequa/realize)なるものがよく利用されているらしく、関連資料もたくさん出てくるのでこれを導入することにした。  

さっそく導入しようとして思ったのがパッケージをプロジェクトごとで管理したいなということ。  
異なるプロジェクトでインストールしたパッケージは、どれも一律$GOPATH/bin、src、pkg配下にファイルが配置されるのを認識していたので、これを各プロジェクト配下に配置して管理したいと思った。  

RailsだとGemfileに記載されたGemをプロジェクトごとに管理したい時は、bundle install時にインストール先を「プロジェクト/vendor/bundle」みたいにしていた。  
今回もこんな感じのことがしたくて調べると次の文を見つけた。

> Go では GOPATH という特殊な概念があって、Go のコードはライブラリも含めてすべて $GOPATH/src 以下に置くという約束になっている。そうするとプロジェクトごとにライブラリのバージョンを変えたい場合に困るので、プロジェクト固有のライブラリは $GOPATH/src/プロジェクト/vendor の下に入れても良いことになっている。そうすると go コマンドは vendor 以下を優先して見てくれる。面白い事に、vendor ディレクトリの管理は govendor 等色々なサードパーティツールを使う。
>
> 将来の Go 1.12 からこのやり方を改め、$GOPATH/src や vendor は廃止となり Go Modules という物を採用する事になった。現在の Go 1.11 は移行期だが、環境変数 GO111MODULE により Go Modules を試す事が出来る。Go 1.11 で Go Module mode にするには次のようにする。

[Qiita | Go Modules](https://qiita.com/propella/items/e49bccc88f3cc2407745)より引用。

どうやらgoでもサードパーティツールを導入することで/vendorでプロジェクトごとに管理することはできるみたいだが、バージョン1.12からはこのやり方は非奨励となり、1.11よりGo本体に導入されたGo Modulesという仕組みを利用することを勧めている。

そしてGo modulesの大きな特徴は以下のようなものになるらしい。

> ・リポジトリのモジュール化  
> ・セマンティックバージョニングによるモジュールのバージョニング  
> ・go.mod による依存関係の管理  
> ・go.sum による依存モジュールのチェックサムの管理  
> 
> リポジトリに含まれる複数のパッケージをまとめてモジュール化し、モジュール単位でセマンティックバージョニングを使ってバージョニングをします。  
> モジュールパスや、go.mod を作成した Go のバージョン、そのモジュールが依存するモジュール群などは go.mod に記述されます。  
> go.sum には、依存しているモジュールのファイルツリーのハッシュ値が記録され、ビルド時に verify されます。  
> go.mod、go.sum ともに手動で書き換えることはなく、 go get、go mod tidy、go mod edit などでプログラム的に書き換えられます。 

[Go 1.13 に向けて知っておきたい Go Modules とそれを取り巻くエコシステム](https://syfm.hatenablog.com/entry/2019/08/10/170730)より引用。

そしてこのGo modules を使うかどうかの制御は$GO111MODULEという環境変数で設定することができる。

> ・on にされていれば Go modules を使う (module-aware mode)  
> ・off にされていれば従来どおり $GOPATH を使う (GOPATH mode)  
> ・auto の場合 $GOPATH/src の外に対象のリポジトリがあり、 go.mod が存在する場合は module-aware mode、そうでない場合は GOPATH mode  
>
> Go 1.11 時点では、 $GO111MODULE=auto がデフォルトとして設定されていたため、従来どおり $GOPATH 内にリポジトリを配置していた場合は明示的に on にしない限り Go modules が有効化されることはありませんでした。  
> 以前は Go 1.13 から Go modules がデフォルト ($GO111MODULE=on) になるとアナウンスされていましたが、[Issue#31857](https://github.com/golang/go/issues/31857) にて auto をデフォルトのままとする決定が行われました。  
>
> ただ、Issue にもある通り、 auto の定義が若干変わっており、 $GOPATH/src 内であっても go.mod が存在する場合は module-aware mode と認識されるようになります。  

[Go 1.13 に向けて知っておきたい Go Modules とそれを取り巻くエコシステム](https://syfm.hatenablog.com/entry/2019/08/10/170730)より引用。

一時期は1.13よりmodule-aware modeがデフォルトにする予定だったとのことで、今後もこの仕組みが使われていく気もするので導入しといて損はなさそう。  
さっそく公式の[クイックスタート](https://github.com/golang/go/wiki/Modules#quick-start)を試してみた。  
ざっくりと手順を下記にまとめとく。  

・GO111MODULEの値をautoもしくは、onにする(デフォルトauto)  
・GOPATHの外にプロジェクトディレクトリを作成しgithubにリポジトリ登録(上述の通りautoでGOPATH内でもgo.modが存在すればmodule-aware modeと認識されるはず。試してないから分からん)  
・go mod init github.com/my/repo でgo.mod、go.sumファイルが生成される  
・importでパッケージを指定した状態でビルドすると、Go のバージョン、そのモジュールが依存するモジュール群などが go.modに記録され、go.sum には、依存しているモジュールのファイルツリーのハッシュ値が記録される  


[go.mod]
```go
module github.com/residenti/go_module_sample

go 1.13

require rsc.io/quote v1.5.2
```

[go.sum]
```go
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZOaTkIIMiVjBQcw93ERBE4m30iBm00nkL0i8=
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3Y=
rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPXsUe+TKr0=
rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/QiW4=
rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9JXDnKaTXpA=
```

注意点として、module-aware modeで[go get](http://cuto.unirita.co.jp/gostudy/post/command-go-get/)を行うとGOPATH modeとは異なる挙動をすること。  
GOPATH mode で go get すると $GOPATH/src 以下に配置されるが、module-aware mode go get をすると $GOPATH/pkg/mod 配下にバージョン毎に配置され、また  go.mod と go.sum に go get したパッケージの情報を記録される。

ちなみに realize というツールをmodule-aware mode で go get で取得したところ、上述の通りの挙動となった。  
しかし、必要なツールが導入でき管理ファイルにもパッケージ情報が記載されたので、今回はこれで大丈夫かなと思っていたがgo mod tidy のようにパッケージの依存関係を整理するときにプロジェクトないで利用されていないパッケージ(importされてない)情報は、go.modとgo.sumより削除されてしまうことを知った。  
それと、そもそもこのままだとインストールした realizeが実行ファイルとして展開されておらず(おそらくバージョン毎に管理するので、勝手に$GOPATH/bin配下に展開されると困る)実行すらできない。  
[先ほどの記事](https://syfm.hatenablog.com/entry/2019/08/10/170730)を読み直すと「プロジェクトで使うツールの依存管理」という項目にプロジェクトで使用するツールのバージョン管理方法についても記載があった。  

より詳しい説明が欲しいかったので調べると下記のサイトが見つかった。  
・[Manage Go tools via Go modules](https://marcofranssen.nl/manage-go-tools-via-go-modules/)
・[go-modules-by-example/index](https://github.com/go-modules-by-example/index/blob/master/010_tools/README.md)

簡単にまとめると、  
・tools.goというファイルにblank importでツールとして利用するパッケージを記載しといてあげることで、go mod tidy したときに消されなくなる  
・go install を行うとGOBINという環境変数に指定されたパス配下に実行ファイルを吐き出しくれるので、この値を GOBIN=プロジェクト/bin にしておくとプロジェクト配下に配置することができる

以上のことを踏まえて、作成したプロジェクトが[trading_bitcoin_api](https://github.com/residenti/trading_bitcoin_api)になる。

最後に作業中の状況を思い出すために、その時の[ツイート](https://twitter.com/v_residenti/status/1210214637980934144?s=20)を載せとく。
