---
layout: post
title:  "開発用ブロックチェーンの構築"
date:   2019-09-22 01:00:00 +0900
categories: blockchain
---

Ethereumのブロックチェーンアプリケーションを使うためには、Ethereumブロックチェーンネットワークが必要になる。
そのため、まずは必要となるブロックチェーンネットワークを構築しいく。

ちなみに、ネットワークには下記の3種類が存在する。

・パブリックネット
・プライベートネット
・テストネット

## パブリックネット
メインネットとも呼ばれる。
いわゆる本番環境で、実世界で利用されているブロックチェーンネットワーク。
本ネット内のetherのみが法定通貨や他の仮想通貨と交換ができる。
先ほども書いた通り、実世界で利用されているため、発行したトランザクションがブロックに取り込まれると、消す事はできないので注意が必要。

## プライベートネット
独自に構築できるネットで、開発環境として利用される。
個人、組織内のみで良く利用される、全世界に公開されることは稀。
構築者自身がマイナーなため、マイニング報酬のETHは全て構築者のものとなり、初期設定であらかじめ決められた数量のETHを保持することができる。

## テストネット
プライベートでのテストが終わって、パブリックネットにリリースする前の最終テスト環境として利用され、 代表的なテストネットとしてRopsten, Rinkeby, Kovanの3種類が存在する。
テストネット上で必要なETHはマイニングで入手可能だが、ハードルが高いため配布サイトやコミュニティから送金してもらうのが良い。

go-ethereum(通称Geth)を利用し、開発に必要となるプライベートネットを作成する。
go-ethereumの初期化。

```
$ geth --datadir ~/private-net/ --nodiscover --maxpeers 0 init ~/private-net/genesis.json
INFO [09-18|21:31:23.931] Maximum peer count                       ETH=0 LES=0 total=0
INFO [09-18|21:31:23.956] Allocated cache and file handles         database=/Users/NRintaro/private-net/geth/chaindata cache=16.00MiB handles=16
INFO [09-18|21:31:23.976] Writing custom genesis block
INFO [09-18|21:31:23.978] Persisted trie from memory database      nodes=7 size=1.02KiB time=167.7µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [09-18|21:31:23.979] Successfully wrote genesis state         database=chaindata hash=0e1989…f9d68e
INFO [09-18|21:31:23.979] Allocated cache and file handles         database=/Users/NRintaro/private-net/geth/lightchaindata cache=16.00MiB handles=16
INFO [09-18|21:31:23.995] Writing custom genesis block
INFO [09-18|21:31:23.995] Persisted trie from memory database      nodes=7 size=1.02KiB time=212.566µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [09-18|21:31:23.996] Successfully wrote genesis state         database=lightchaindata hash=0e1989…f9d68e
```
[オプションについて]
・--datadir: データベースとキーストア用のデータディレクトリを指定
・--nodiscover: ピア発見メカニズムを無効 (手動でピアを追加)
・--maxpeers: ネットワークピアの最大数を指定 (0にセットするとネットワークは無効)

go-ethereumでプライベートネットの起動
各種オプションを毎回指定するのは大変なので、シェルスクリプトとしてまとめる。
```sh
#geth-start.sh
geth --datadir ~/private-net/ --networkid 15 \
  --nodiscover --maxpeers 0 --mine --minerthreads 1 \
  --rpc --rpcaddr "0.0.0.0" --rpccorsdomain "*" \
  --rpcvhosts "*" --rpcapi "eth,web3,personal,net" \
  --wsapi "eth,web3,personal,net" --wsorigins "*" \
  --unlock 0,1,2,3,4 --password ~/private-net/password
```

[オプションについて]
・--networkid: ネットワークIDを指定
・--mine: マイニングを有効
・--minerthreads: マイニングに使うCPUスレッド数を指定
・--rpc: HTTP-RPC サーバを有効
・--rpcaddr: HTTP-RPC サーバのリスニングインターフェイスを指定
・--rpccorsdomain: クロスoriginリクエストを受け付けるドメインのカンマ区切りリストを指定(ブラウザ強制
・--rpcvhosts: リクエストを受け付ける仮想ホスト名のコンマ区切りリストを指定 (サーバー強制/ワイルドカード「\*」の指定も可能)
・--rpcapi: HTTP-RPCインターフェースを介して提供されるAPIを指定
・--wsapi: WS-RPC インターフェースを介して提供されるAPIを指定
・--wsorigins: WebSocketリクエストを受け付けるoriginを指定
・--unlock: unlockするアカウントのリストをカンマ区切りで指定
・--password: パスワードファイルのパスを指定(対話型のパスワード入力ではない)


実行。
```
$ ./geth-start.sh
INFO [09-18|21:46:57.397] Maximum peer count                       ETH=0 LES=0 total=0
INFO [09-18|21:46:57.419] Starting peer-to-peer node               instance=Geth/v1.9.3-stable/darwin-amd64/go1.12.9
INFO [09-18|21:46:57.419] Allocated trie memory caches             clean=256.00MiB dirty=256.00MiB
INFO [09-18|21:46:57.419] Allocated cache and file handles         database=/Users/NRintaro/private-net/geth/chaindata cache=512.00MiB handles=5120
INFO [09-18|21:46:57.471] Opened ancient database                  database=/Users/NRintaro/private-net/geth/chaindata/ancient
INFO [09-18|21:46:57.472] Initialised chain configuration          config="{ChainID: 15 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Byzantium: 0 Constantinople: <nil> Petersburg: <nil> Istanbul: <nil> Engine: unknown}"
INFO [09-18|21:46:57.472] Disk storage enabled for ethash caches   dir=/Users/NRintaro/private-net/geth/ethash count=3
INFO [09-18|21:46:57.472] Disk storage enabled for ethash DAGs     dir=/Users/NRintaro/Library/Ethash          count=2
INFO [09-18|21:46:57.473] Initialising Ethereum protocol           versions=[63] network=15 dbversion=<nil>
WARN [09-18|21:46:57.473] Upgrade blockchain database version      from=<nil> to=7
INFO [09-18|21:46:57.525] Loaded most recent local header          number=0 hash=0e1989…f9d68e td=1 age=50y5mo1w
INFO [09-18|21:46:57.525] Loaded most recent local full block      number=0 hash=0e1989…f9d68e td=1 age=50y5mo1w
INFO [09-18|21:46:57.525] Loaded most recent local fast block      number=0 hash=0e1989…f9d68e td=1 age=50y5mo1w
INFO [09-18|21:46:57.526] Regenerated local transaction journal    transactions=0 accounts=0
INFO [09-18|21:46:57.536] Allocated fast sync bloom                size=512.00MiB
INFO [09-18|21:46:57.538] Initialized fast sync bloom              items=7 errorrate=0.000 elapsed=240.921µs
INFO [09-18|21:46:57.573] New local node record                    seq=1 id=98e0d0e5b5c17d42 ip=127.0.0.1 udp=0 tcp=30303
INFO [09-18|21:46:57.573] Started P2P networking                   self="enode://b3e8cb3f279e57056563f650e94023c7017e94b678c190a0826ccb28acae0545383380a7da5422209d80e499978ba0eff9258aca5506f51dbc95592436ddb310@127.0.0.1:30303?discport=0"
INFO [09-18|21:46:57.576] IPC endpoint opened                      url=/Users/NRintaro/private-net/geth.ipc
INFO [09-18|21:46:57.577] HTTP endpoint opened                     url=http://0.0.0.0:8545                  cors=* vhosts=*
Fatal: Account unlock with HTTP access is forbidden!
```

Fatal!

調べたところ、バージョン1.9.0からRPCでアカウントをアンロック状態にするには、--allow-insecure-unlock を指定しないとダメだったみたい。  
geth-start.shを修正。

[参考サイト]
[Go-Ethereum変更点まとめ (1.8.23 -> 1.9.0)](https://qiita.com/KKZ@github/items/f7f358bc3f017960a44d)
[Error: account unlock with HTTP access is forbidden when unlock an account in Geth console](https://ethereum.stackexchange.com/questions/69435/error-account-unlock-with-http-access-is-forbidden-when-unlock-an-account-in-ge)
```sh
geth --datadir ~/private-net --networkid 15 \
  --nodiscover --maxpeers 0 --mine --minerthreads 1 \
  --rpc --rpcaddr "0.0.0.0" --rpccorsdomain "*" \
  --rpcvhosts "*" --rpcapi "eth,web3,personal,net" \
  --ipcpath ~/private-net/geth.ipc --ws --wsaddr "0.0.0.0" \
  --wsapi "eth,web3,personal,net" --wsorigins "*" \
  --unlock 0,1,2,3,4 --password ~/private-net/password \
  --allow-insecure-unlock
```

今度こそ。
```
$ ./geth-start.sh
INFO [09-18|22:44:37.482] Maximum peer count                       ETH=0 LES=0 total=0
INFO [09-18|22:44:37.503] Starting peer-to-peer node               instance=Geth/v1.9.3-stable/darwin-amd64/go1.12.9
INFO [09-18|22:44:37.503] Allocated trie memory caches             clean=256.00MiB dirty=256.00MiB
INFO [09-18|22:44:37.503] Allocated cache and file handles         database=/Users/NRintaro/private-net/geth/chaindata cache=512.00MiB handles=5120
INFO [09-18|22:44:37.562] Opened ancient database                  database=/Users/NRintaro/private-net/geth/chaindata/ancient
INFO [09-18|22:44:37.564] Initialised chain configuration          config="{ChainID: 15 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Byzantium: 0 Constantinople: <nil> Petersburg: <nil> Istanbul: <nil> Engine: unknown}"
INFO [09-18|22:44:37.564] Disk storage enabled for ethash caches   dir=/Users/NRintaro/private-net/geth/ethash count=3
INFO [09-18|22:44:37.564] Disk storage enabled for ethash DAGs     dir=/Users/NRintaro/Library/Ethash          count=2
INFO [09-18|22:44:37.564] Initialising Ethereum protocol           versions=[63] network=15 dbversion=7
INFO [09-18|22:44:37.617] Loaded most recent local header          number=0 hash=0e1989…f9d68e td=1 age=50y5mo1w
INFO [09-18|22:44:37.617] Loaded most recent local full block      number=0 hash=0e1989…f9d68e td=1 age=50y5mo1w
INFO [09-18|22:44:37.617] Loaded most recent local fast block      number=0 hash=0e1989…f9d68e td=1 age=50y5mo1w
INFO [09-18|22:44:37.617] Loaded local transaction journal         transactions=0 dropped=0
INFO [09-18|22:44:37.617] Regenerated local transaction journal    transactions=0 accounts=0
INFO [09-18|22:44:37.627] Allocated fast sync bloom                size=512.00MiB
INFO [09-18|22:44:37.628] Initialized fast sync bloom              items=7 errorrate=0.000 elapsed=119.414µs
INFO [09-18|22:44:37.660] New local node record                    seq=4 id=98e0d0e5b5c17d42 ip=127.0.0.1 udp=0 tcp=30303
INFO [09-18|22:44:37.660] Started P2P networking                   self="enode://b3e8cb3f279e57056563f650e94023c7017e94b678c190a0826ccb28acae0545383380a7da5422209d80e499978ba0eff9258aca5506f51dbc95592436ddb310@127.0.0.1:30303?discport=0"
INFO [09-18|22:44:37.661] IPC endpoint opened                      url=/Users/NRintaro/private-net/geth.ipc
INFO [09-18|22:44:37.662] HTTP endpoint opened                     url=http://0.0.0.0:8545                  cors=* vhosts=*
INFO [09-18|22:44:37.662] WebSocket endpoint opened                url=ws://[::]:8546
WARN [09-18|22:44:37.663] -------------------------------------------------------------------
WARN [09-18|22:44:37.663] Referring to accounts by order in the keystore folder is dangerous!
WARN [09-18|22:44:37.663] This functionality is deprecated and will be removed in the future!
WARN [09-18|22:44:37.663] Please use explicit addresses! (can search via `geth account list`)
WARN [09-18|22:44:37.663] -------------------------------------------------------------------
```

SUCCESS!

Go-Ethereumのコンソールにアクセスして、eth.blockNumberコマンドでブロック高を確認する。
```
$ geth attach http://localhost:8545
Welcome to the Geth JavaScript console!

instance: Geth/v1.9.3-stable/darwin-amd64/go1.12.9
coinbase: 0x945cd603a6754cb13c3d61d8fe240990f86f9f8a
at block: 246 (Wed, 18 Sep 2019 23:04:43 JST)
 modules: eth:1.0 net:1.0 personal:1.0 rpc:1.0 web3:1.0

> eth.blockNumber
247
> exit
```

うん、大丈夫そう。  
今日(2019/09/18)はここまで。
