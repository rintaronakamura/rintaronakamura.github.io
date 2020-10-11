---
layout: post
title:  "Amazon Web Services 基礎からのネットワーク＆サーバー構築"
date:   2019-08-31 01:00:00 +0900
categories: aws
---

最近仕事でAWSを弄る機会もあり、自力でAWS上にネットワークとサーバー構築くらいできるくらいの知識は欲しいなと思い書籍を1冊購入した。
それに、こういうのがさっとできるようになると個人開発の敷居もいくらか下がりそうな気もしたので。

購入した書籍: [Amazon Web Services 基礎からのネットワーク＆サーバー構築 改訂版 Kindle版](https://www.amazon.co.jp/gp/product/B06Y5ZSYY4/ref=ppx_yo_dt_b_d_asin_title_o00?ie=UTF8&psc=1)

ここには、書籍を進めていく中で参考にしたサイトのリンクや対応したエラーなどを載せてく。

## 参考サイト

### アカウントを取得後にやっておくこと

・[AWSアカウントを取得したら速攻でやっておくべき初期設定まとめ](https://qiita.com/tmknom/items/303db2d1d928db720888)
・[AWS初心者向けWebinar 利用者が実施するAWS上でのセキュリティ対策](https://www.slideshare.net/AmazonWebServicesJapan/awswebinar-aws-56260969)
・[クラウド破産しないように git-secrets を使う](https://qiita.com/pottava/items/4c602c97aacf10c058f1)

### サービスについて

・[AWS クラウド無料利用枠 | AWS ](https://aws.amazon.com/jp/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=categories%23featured%7Ccategories%23alwaysfree)
・[【AWS】S3まとめ](https://qiita.com/iron-breaker/items/f35c1d54887c434a321a)

### その他

・[GitHub に AWS キーペアを上げると抜かれるってほんと？？？試してみよー！](https://qiita.com/saitotak/items/813ac6c2057ac64d5fef)
・[AWSを使い始めて2日目に無料枠限度アラームがきた話](https://qiita.com/Ki2neudon/items/aefaa9edb435b4945c3a)
・[【AWS EC2 エラー】ssh port 22 Operation timed out](https://qiita.com/yokoto/items/338bd80262d9eefb152e)

## 一時的なタイトル

### SSHで接続する

インスタンスにインターネット側からアクセスするには、事前にインスタンスの「パブリックIPアドレス」を調べておく。
次に下記のコマンドを実行しインスタンスにSSHで接続する。

```
$ ssh -i my-key.pem ec2-user@{パブリックIPアドレス}
```

-i オプションでキーペアファイルを指定している。
初回の接続に限り色々聞かれる。「対応したエラー > 1. ssh: connect to host x.xxx.xxx.xxx port 22: Operation timed out」を参考に。

### Apachを起動/自動起動させる

`service`コマンドを使い指定したコマンドを「起動(start)」する。

```
$ sudo service httpd start
```

他にも「停止(stop)」「再起動(restart)」「状態確認(status)」とか。

これでApachが起動するがサーバーを再起動するとまた停止してしまうので、サーバーの起動に合わせてApachも起動するようにする。

```
$ sudo chkconfig httpd on
```

正しく設定されたかは次のコマンドで確認できる。

```
$ sudo chkconfig --list httpd
httpd          0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

0~6はランレベルと呼ばれるLinuxの概念で、カーネルがどのような状態であるかを表す用語。

0 : シャットダウンに向かう状態
1 : シングルユーザモード
2 : 使用されない
3 : 標準的な状態
4 : 使用されない
5 : GUIでログインする状態
6 : 再起動に向かう状態

一般的に自動起動を設定する必要があるのは3と5らしい。

ランレベルを指定してonまたはoffに設定した時は次のようになる。

```
# ランレベル3と5をonに設定する.
chkconfig --level 35 httpd on
```

## 対応したエラー

### 1. ssh: connect to host x.xxx.xxx.xxx port 22: Operation timed out

EC2インスタンスにsshしようとした時に起きたエラー。

```
$ ssh -i my-key.pem ec2-user@x.xxx.xxx.xxx
ssh: connect to host x.xxx.xxx.xxx port 22: Operation timed out
```

[ここ](https://qiita.com/yokoto/items/338bd80262d9eefb152e)を参考に、セキュリティグループのインバウンドで、sshの送信元をマイIP(自分のパソコンのIPアドレス)に設定したのち、インスタンスを再起動したらいけた。

`The authenticity of host`うんたらかんたらは初回接続時に聞かれるものなので`yes`で大丈夫。

```
$ ssh -i my-key.pem ec2-user@x.xxx.xxx.xxx
The authenticity of host 'x.xxx.xxx.xxx (x.xxx.xxx.xxx)' can't be established.
ECDSA key fingerprint is SHA256:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.
Are you sure you want to continue connecting (yes/no)? yes

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/
3 package(s) needed for security, out of 3 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-xx-x-x ~]$
```

ちなみに鍵ファイルのパーミッションが、他のユーザーも閲覧できるようになっていたため、次のようなエラーが出てsshが失敗した。
この場合は`chmod`コマンドで自分だけしか読み込めないように権限を変更したらいけた。

```
Warning: Permanently added 'x.xxx.xxx.xxx' (ECDSA) to the list of known hosts.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'my-key.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "my-key.pem": bad permissions
ec2-user@x.xxx.xxx.xxx: Permission denied (publickey).

$ ls -la my-key.pem
-rw-r--r--@ 1 residenti  staff  1692  6  9 19:38 my-key.pem

# 400の「4」は、読み込み属性しか付いていないことを示す
$ chmod 400 my-key.pem

$ ls -la my-key.pem
-r--------@ 1 residenti  staff  1692  6  9 19:38 my-key.pem
```

### 2. Your server is running PHP version 5.4.45 but WordPress 5.2.2 requires at least 5.6.20.

[こちら](https://homepage-reborn.com/2019/01/22/2019%E5%B9%B44%E6%9C%88%E3%81%8B%E3%82%89wordpress%E3%81%AEphp%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3%E3%81%AF5-6%E4%BB%A5%E4%B8%8A%E3%81%8C%E5%BF%85%E8%A6%81%E3%81%AB%E3%81%AA%E3%82%8B%E3%82%88/)の記事に書かれている通り、2019年4月からWordPressのphpバージョンは5.6以上が必要になった模様。

Amazon Linuxにphp7.3系をインストールする手順をまとめとく。
まずは、すでにインストールしてしまった古いバージョンのphp(また周辺のパッケージ)をアンインストールするし、yum自体をアップデートする。
```
$ sudo yum -y remove php-*
$ sudo yum -y update
```

今回yumリポジトリでは管理されていない新しいバージョンのパッケージをインストールしたいので、remiリポジトリ追加する。
＊remiリポジトリがAmazon Linux（Amazon Linux AMI release 2016.09）ではデフォルトでインストールされていなかった。
```
sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-6.rpm
```
/etc/yum.repos.d/remi.repoというファイルが生成されている。詳しくは[参考サイト](https://qiita.com/yamaguchi_takashi/items/d4b7b2693b42679dc3ae#%E5%90%84%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%81%AE%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88)を参照。

各リポジトリをアップデートする。
```
sudo yum -y update --enablerepo=epel,remi,remi-php73
```

php7.3系とその他に必要なパッケージをインストールする。
```
$ sudo yum -y install  --disablerepo=amzn-main --enablerepo=remi-php73 php php-mysql php-mbstring

$ php -v
PHP 7.3.9 (cli) (built: Aug 28 2019 12:08:07) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.9, Copyright (c) 1998-2018 Zend Technologies
```

[参考サイト](https://qiita.com/yamaguchi_takashi/items/d4b7b2693b42679dc3ae#%E5%90%84%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%81%AE%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88)
