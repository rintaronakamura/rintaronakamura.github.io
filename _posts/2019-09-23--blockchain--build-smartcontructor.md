---
layout: post
title:  "スマートコントラクト開発環境の構築"
date:   2019-09-23 01:00:00 +0900
categories: blockchain
---

この記事は[堅牢なスマートコントラクト開発のためのブロックチェーン[技術]入門](https://gihyo.jp/book/2017/978-4-7741-9353-3)のChapter8、「スマートコントラクト開発の準備とSolidityの基本文法」を進めながら記載している。Chapter8では、タイトルの通り開発準備とSolidityの基本文法が説明されていたが、ここに記載しているのは開発準備のみ。

## Gethのインストール
GethとはEthereum（イーサリアム）が提供しているクライアントソフトで、Go Ethereumの略称。
できること
- マイニング
- 資金移動
- スマートコントラクト
- ブロック履歴の表示

インストール方法はいくつかあるが、今回はHomebrewでインストールする。
まずは、Homebrewのパッケージカタログを最新版に更新して、Ethereumリポジトリをパッケージカタログに追加する。

```
$ brew update
Alerady up-to-date.

$ brew tap ethereum/ethereum
==> Tapping ethereum/ethereum
Cloning into '/usr/local/Homebrew/Library/Taps/ethereum/homebrew-ethereum'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 9 (delta 2), reused 1 (delta 0), pack-reused 0
Unpacking objects: 100% (9/9), done.
Warning: Aleth (formerly cpp-ethereum) has been removed from Homebrew. Please install binary releases from https://github.com/ethereum/aleth/releases.
Tapped 4 formulae (28 files, 18.0KB).
```

次に、Gethをインストールする。

```
$ brew install ethereum
==> Downloading https://homebrew.bintray.com/bottles/ethereum-1.9.3.mojave.bottle.tar.gz
==> Downloading from https://akamai.bintray.com/ff/ff6330010cc5feb2dbe25643d40f0583492d505aed2ffc48f7cfcacb41960479?__gda__=exp=1568598835~hmac=de6fb6c72464cf935d57703c6d31254d7a
######################################################################## 100.0%
==> Pouring ethereum-1.9.3.mojave.bottle.tar.gz
  /usr/local/Cellar/ethereum/1.9.3: 19 files, 327.2MB
==> `brew cleanup` has not been run in 30 days, running now...
Removing: /Users/residenti/Library/Caches/Homebrew/libevent--2.1.10.mojave.bottle.tar.gz... (782.3KB)
Removing: /Users/residenti/Library/Caches/Homebrew/oniguruma--6.9.2.mojave.bottle.tar.gz... (422.2KB)
Removing: /usr/local/Cellar/openssl/1.0.2r... (1,795 files, 12.1MB)
Removing: /Users/residenti/Library/Caches/Homebrew/wget--1.20.3_1.mojave.bottle.tar.gz... (1.4MB)
Removing: /Users/residenti/Library/Logs/Homebrew/ncurses... (64B)
Removing: /Users/residenti/Library/Logs/Homebrew/tmux... (64B)
Removing: /Users/residenti/Library/Logs/Homebrew/oniguruma... (64B)
Removing: /Users/residenti/Library/Logs/Homebrew/libevent... (64B)
Removing: /Users/residenti/Library/Logs/Homebrew/jq... (64B)
Removing: /Users/residenti/Library/Logs/Homebrew/openssl... (64B)
Removing: /Users/residenti/Library/Logs/Homebrew/telnet... (64B)
Pruned 0 symbolic links and 2 directories from /usr/local
```

最後に、Gethが正しくインストールされたかを確認する。
```
$ geth version
Geth
Version: 1.9.3-stable
Architecture: amd64
Protocol Versions: [63]
Network Id: 1
Go Version: go1.12.9
Operating System: darwin
GOPATH=
GOROOT=/usr/local/Cellar/go/1.12.9/libexec[]
```

## Genesisブロックの作成とgethの起動

```
# genesis.json
{
  "config": {
    "chainId": 15,
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0
  },
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "difficulty" : "0x00",
  "alloc": {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "timestamp"  : "0x00",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "extraData"  : "0x00",
  "gasLimit"   : "0x1312d00"
}
```

初期化処理
```
eth_private_net$ geth --datadir ~/eth_private_net init genesis.json
INFO [09-22|18:47:33.874] Maximum peer count                       ETH=50 LES=0 total=50
INFO [09-22|18:47:33.914] Allocated cache and file handles         database=/Users/residenti/eth_private_net/geth/chaindata cache=16.00MiB handles=16
INFO [09-22|18:47:33.930] Writing custom genesis block
INFO [09-22|18:47:33.931] Persisted trie from memory database      nodes=0 size=0.00B time=10.053µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [09-22|18:47:33.932] Successfully wrote genesis state         database=chaindata hash=76d747…1a5e65
INFO [09-22|18:47:33.932] Allocated cache and file handles         database=/Users/residenti/eth_private_net/geth/lightchaindata cache=16.00MiB handles=16
INFO [09-22|18:47:33.945] Writing custom genesis block
INFO [09-22|18:47:33.945] Persisted trie from memory database      nodes=0 size=0.00B time=4.841µs  gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [09-22|18:47:33.946] Successfully wrote genesis state         database=lightchaindata hash=76d747…1a5e65
```

geth起動
```
eth_private_net$ geth --networkid "10" \
                   --nodiscover \
                   --datadir ~/eth_private_net \
                   --rpc --rpcaddr "localhost" \
                   --rpcport "8545" \
                   --rpccorsdomain "*" \
                   --rpcapi "eth,web3,personal" \
                   --targetgaslimit "20000000" \
                   console 2 >> ~/eth_private_net/geth_err.log

INFO [09-22|19:08:02.762] Maximum peer count                       ETH=50 LES=0 total=50
INFO [09-22|19:08:02.787] Starting peer-to-peer node               instance=Geth/v1.9.3-stable/darwin-amd64/go1.12.9
INFO [09-22|19:08:02.788] Allocated trie memory caches             clean=256.00MiB dirty=256.00MiB
INFO [09-22|19:08:02.788] Allocated cache and file handles         database=/Users/residenti/eth_private_net/geth/chaindata cache=512.00MiB handles=5120
INFO [09-22|19:08:02.822] Opened ancient database                  database=/Users/residenti/eth_private_net/geth/chaindata/ancient
INFO [09-22|19:08:02.823] Initialised chain configuration          config="{ChainID: 15 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Byzantium: <nil> Constantinople: <nil> Petersburg: <nil> Istanbul: <nil> Engine: unknown}"
INFO [09-22|19:08:02.823] Disk storage enabled for ethash caches   dir=/Users/residenti/eth_private_net/geth/ethash count=3
INFO [09-22|19:08:02.823] Disk storage enabled for ethash DAGs     dir=/Users/residenti/Library/Ethash count=2
INFO [09-22|19:08:02.823] Initialising Ethereum protocol           versions=[63] network=10 dbversion=7
INFO [09-22|19:08:02.877] Loaded most recent local header          number=0 hash=76d747…1a5e65 td=0 age=50y5mo1w
INFO [09-22|19:08:02.877] Loaded most recent local full block      number=0 hash=76d747…1a5e65 td=0 age=50y5mo1w
INFO [09-22|19:08:02.877] Loaded most recent local fast block      number=0 hash=76d747…1a5e65 td=0 age=50y5mo1w
INFO [09-22|19:08:02.878] Loaded local transaction journal         transactions=0 dropped=0
INFO [09-22|19:08:02.878] Regenerated local transaction journal    transactions=0 accounts=0
INFO [09-22|19:08:02.893] Allocated fast sync bloom                size=512.00MiB
INFO [09-22|19:08:02.894] Initialized fast sync bloom              items=0 errorrate=0.000 elapsed=89.152µs
INFO [09-22|19:08:02.926] IPC endpoint opened                      url=/Users/residenti/eth_private_net/geth.ipc
INFO [09-22|19:08:02.928] New local node record                    seq=5 id=f96ef41c9ab3040e ip=127.0.0.1 udp=0 tcp=30303
INFO [09-22|19:08:02.928] Started P2P networking                   self="enode://32c08a3794125cf63b9c6355dde6dae5ec7fb51d09836f7a6e4b0c4c9b99897c7c33e802aed9723b0c61fe45e6d9d436d48b334e3e803191a824165fd5b20760@127.0.0.1:30303?discport=0"
INFO [09-22|19:08:02.929] HTTP endpoint opened                     url=http://localhost:8545                    cors= vhosts=localhost
WARN [09-22|19:08:03.018] Served eth_coinbase                      reqid=3 t=20.136µs err="etherbase must be explicitly specified"
```

## アカウントの作成

gethの対話型コンソールに接続し、アカウントを4つ作成する。
作成するとEOAのアドレスが出力される。
```
eth_private_net$ geth attach http://localhost:8545
Welcome to the Geth JavaScript console!

instance: Geth/v1.9.3-stable/darwin-amd64/go1.12.9
at block: 0 (Thu, 01 Jan 1970 09:00:00 JST)
 modules: eth:1.0 personal:1.0 rpc:1.0 web3:1.0

> personal.newAccount("password")
"0x2c57107071248cd578083f57ec558744e0fd9faf"
> personal.newAccount("password")
"0x2cf7c17e5cc600c4d0f351aa1aa5aef039abc9f5"
> personal.newAccount("password")
"0x308340c825cc97c6ee0c61d12e7c04ad815f6e29"
> personal.newAccount("password")
"0x095af36007331617d1270098f4e27d605deff841"
> eth.accounts
["0x2c57107071248cd578083f57ec558744e0fd9faf", "0x2cf7c17e5cc600c4d0f351aa1aa5aef039abc9f5", "0x308340c825cc97c6ee0c61d12e7c04ad815f6e29", "0x095af36007331617d1270098f4e27d605deff841"]
```

インデックス指定でアカウントの確認。
coinbaseアカウントの確認と変更。
```
eth_private_net$ geth attach http://localhost:8545
Welcome to the Geth JavaScript console!

instance: Geth/v1.9.3-stable/darwin-amd64/go1.12.9
at block: 0 (Thu, 01 Jan 1970 09:00:00 JST)
 modules: eth:1.0 personal:1.0 rpc:1.0 web3:1.0

> eth.accounts[0]
"0x2c57107071248cd578083f57ec558744e0fd9faf"

> eth.coinbase
"0x2c57107071248cd578083f57ec558744e0fd9faf"

> miner.setEtherbase(eth.accounts[1])
true

> eth.coinbase
"0x2cf7c17e5cc600c4d0f351aa1aa5aef039abc9f5"
```

## gethコンソールでよく使うコマンド

以降で登場する「`eth`, `personal`, `miner`」から始まるコマンドは、web3というオプジェクトに含まれるもので、省略しないで書くと「`Web3.eth`, `Web3.personal`, `Web3.miner`」となる。
また、web3が持つオブジェクトや関数はgethコンソール上で「web3」と入力すると確認できる。

[https://github.com/ethereum/go-ethereum/wiki](https://github.com/ethereum/go-ethereum/wiki)

### genesisブロックの内容を確認する

```
> eth.getBlock(0)
{
  difficulty: 0,
  extraData: "0x00",
  gasLimit: 20000000,
  gasUsed: 0,
  hash: "0x76d747ec34337ec5677b1aba554769485e160663eee3c63486400bddc21a5e65",
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  miner: "0x0000000000000000000000000000000000000000",
  mixHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
  nonce: "0x0000000000000042",
  number: 0,
  parentHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
  receiptsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 505,
  stateRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  timestamp: 0,
  totalDifficulty: 0,
  transactions: [],
  transactionsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  uncles: []
}
```

### マイニングの実行と確認
引数にはマイニングに利用するスレッド数を指定する。
「null/true」が出力されればOK。
```
> miner.start(2)
null
```

「true」であればマイニング中。
```
> eth.mining
true
```

### conibaseの残高を確認する
上がwei表記。下が`web3.fromWei`関数を使いether表記にしたやつ。
```
> eth.getBalance(eth.accounts[0])
1.6e+21

> web3.fromWei(eth.getBalance(eth.accounts[0]),"ether")
1520
```

### 送金する
`from`: 送金元, `to`: 送金先アドレス, `value`: 送金額を指定している。
ちなみに、送金額はwei単位で指定する必要があるため、`web3.toWei`関数で5etherをweiに変換している。
```
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[2], value: web3.toWei(5,"ether")})
Error: authentication needed: password or unlock
    at web3.js:3143:20
    at web3.js:6347:15
    at web3.js:5081:36
    at <anonymous>:1:1
```

eth.accounts[0]のロックを解除していないために発生するエラーのため、次のコマンドでロックを解除する。
```
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x2c57107071248cd578083f57ec558744e0fd9faf
Password:
Error: account unlock with HTTP access is forbidden
```
エラー出たので調べたところ、バージョン1.9.0からRPCでアカウントをアンロック状態にするには、`--allow-insecure-unlock`を指定しないとダメだったみたい。
[参考サイト]
[Go-Ethereum変更点まとめ (1.8.23 -> 1.9.0)](https://qiita.com/KKZ@github/items/f7f358bc3f017960a44d)
[Error: account unlock with HTTP access is forbidden when unlock an account in Geth console](https://ethereum.stackexchange.com/questions/69435/error-account-unlock-with-http-access-is-forbidden-when-unlock-an-account-in-ge)

ということで、`--allow-insecure-unlock`オプション追加してgethを再起動する。
```
$ geth --networkid "10" \
     --nodiscover \
     --datadir ~/eth_private_net \
     --targetgaslimit "20000000" \
     --rpc --rpcaddr "localhost" --rpcport "8545" \
     --rpccorsdomain "*" \
     --rpcapi "eth,web3,personal,miner" \
     --allow-insecure-unlock
```

もう一度アンロックするとうまくいった。
```sh
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x2c57107071248cd578083f57ec558744e0fd9faf
Password:
true
```

ただし、本番環境ではセキュリティーの観点からロック解除はしない方が望ましい。
今回はプライベートネットなので、都度パスワードを聞かれるのがめんどいので、geth起動時に`geth`オプションを使いアンロックするようにする。
(＊下記手順でアンロックできなかった。2019/09/23 実施)
まずは、`password.txt`というファイルを新規作成し、`eth.accounts[0]~eth.accounts[3]`のパスワードを改行区切りで記載する。
```text
# password.txt
password

password

password

password
```

そして、下記2つのアンロックオプションを追加しgethを起動する。
```
# 追加するアンロックオプション
--unlock・・・アンロックするアカウントアドレス
--password・・パスワードファイルのパス
```

geth起動。
```sh
$ geth --networkid "10" \
     --nodiscover \
     --datadir ~/eth_private_net \
     --targetgaslimit "20000000" \
     --rpc --rpcaddr "localhost" --rpcport "8545" \
     --rpccorsdomain "*" \
     --rpcapi "eth,web3,personal,miner" \
     --allow-insecure-unlock
     --password ~/eth_private_net/password.txt
     --unlock 0x2c57107071248cd578083f57ec558744e0fd9faf, 0x2cf7c17e5cc600c4d0f351aa1aa5aef039abc9f5, 0x308340c825cc97c6ee0c61d12e7c04ad815f6e29, 0x095af36007331617d1270098f4e27d605deff841
```

参考書だとこれでアンロックできるらしんだけど、発行されてから8ヶ月経ってるし、gethのバージョンも違うの使ってるからか、アンロックできなかった。
なので、上記の`personal.unlockAccount(eth.accounts[0])`で都度ロックを解除して進めることにした。
```sh
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[2], value: web3.toWei(5,"ether")})
"0x3bbf4c0c5a24268a7428c37d70a1416ef97f4b9e09be97230ba4ee57059f2569"
```
発行された16進数のトランザクションハッシュを`eth.getTransaction`関数の引数に渡すことで、そのトランザクションの中身が確認できる。
```sh
> eth.getTransaction('0x3bbf4c0c5a24268a7428c37d70a1416ef97f4b9e09be97230ba4ee57059f2569')
{
  blockHash: "0x2fc5a10923cd08628b7d3cdf8de37e951aa7ce10033e9eacea829a0de0e832f6",
  blockNumber: 491,
  from: "0x2c57107071248cd578083f57ec558744e0fd9faf",
  gas: 21000,
  gasPrice: 1000000000,
  hash: "0x3bbf4c0c5a24268a7428c37d70a1416ef97f4b9e09be97230ba4ee57059f2569",
  input: "0x",
  nonce: 1,
  r: "0xb20dab4c85af91cc3a045d0f2aa5b072f5d137b0f012bf84addebae098746be3",
  s: "0x4df068854935dbcd126305779d4e628567058c5efd8d00d6e88cb18b464e56f0",
  to: "0x308340c825cc97c6ee0c61d12e7c04ad815f6e29",
  transactionIndex: 1,
  v: "0x42",
  value: 5000000000000000000
}
```
続いて`eth.getTransactionReception`関数にトランザクションハッシュを渡して、トランザクションのレシートを表示してみる。
```sh
> eth.getTransactionReceipt('0x3bbf4c0c5a24268a7428c37d70a1416ef97f4b9e09be97230ba4ee57059f2569')
null # nullが返ってくるのは、まだブロックに取り込まれていないので少しおいてから再度実行する.

> eth.getTransactionReceipt('0x3bbf4c0c5a24268a7428c37d70a1416ef97f4b9e09be97230ba4ee57059f2569')
{
  blockHash: "0x2fc5a10923cd08628b7d3cdf8de37e951aa7ce10033e9eacea829a0de0e832f6",
  blockNumber: 491, # 491ブロックに取り込まれている.
  contractAddress: null,
  cumulativeGasUsed: 42000,
  from: "0x2c57107071248cd578083f57ec558744e0fd9faf",
  gasUsed: 21000,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  root: "0x6a01c9e5dc7ee6b386b98f4efc4313686cb24a0bc96c9cc99c14d4c4605d49ba",
  to: "0x308340c825cc97c6ee0c61d12e7c04ad815f6e29",
  transactionHash: "0x3bbf4c0c5a24268a7428c37d70a1416ef97f4b9e09be97230ba4ee57059f2569",
  transactionIndex: 1
}
```

ちゃんと送金できているかを確認する。
```
> web3.fromWei(eth.getBalance(eth.accounts[2]), "ether")
5
```

`eth.getBalance`関数には、ブロック高を指定することでそのブロック時点での残高を確認することもできる。
```
> web3.fromWei(eth.getBalance(eth.accounts[2], 491), "ether")
5
> web3.fromWei(eth.getBalance(eth.accounts[2], 490), "ether")
0
```

### 文字列変換

上から「16進数の文字列 => ASCIIの文字列」、「16進数の文字列 => UTF8の文字列」へと変換している。
```
> web3.toAscii("0x68696d69747573")
"himitus"
> web3.toUtf8("0xe382a4e383bce382b5e383aae382a2e383a0")
"イーサリアム"
```

### マイニングを終了する

```
> miner.stop()
true
```

## Mist Wallet
(＊2019/09/23時点で非奨励となっている。)
- [Sunsetting Mist](https://medium.com/@avsa/sunsetting-mist-da21c8e943d2)
- [Mist Migration Patterns](https://medium.com/@wolovim/mist-migration-patterns-6bcf066ac383)

[Mist Wallet](https://github.com/ethereum/mist)はEthereumの公式ウォレットで、次の機能を備えている。

- アカウントの生成
- etherの残高確認や送金
- スマートコントラクトの生成/関数呼び出し
- イベントの確認

基本はgethコンソールで操作するが、たまにMist Walletを使っていく。
> gethはリモートから呼び出せるJSONベースのAPI(JSON PRCと呼ばれます)も提供しており、HTTPで呼び出せ、Mist WalletはgethのAPIを利用していますので、Mist Walletを利用する際はgethも起動してください。

[github.com/ethereum/mist/releases](https://github.com/ethereum/mist/releases)よりdmgをダウンロードした。

### インストールと起動
geth --datadir =。/ chaindata / --rpc --ipcpath〜/ライブラリ/ Ethereum / geth.ipc console

Ethereum WalletをApplicationsに追加したのち、gethを起動した状態で次のコマンドを実行すると「Insecure RPC connection」というコンソールが表示されるのでokを選択する。
エラーがちらほら出ていて気になるのだが、解決できなかった。。エラーは出ているもののMist Walletアプリは起動でき、アカウントの選択からアドレス表示用のQRコードの表示まで実施できたので一旦よしとして先に進む。
```sh
eth_private_net$ /Applications/Ethereum\ Wallet.app/Contents/MacOS/Ethereum\ Wallet --rpc http://localhost:8545 ./geth.ipc
[2019-09-23T15:38:47.342] [INFO] Settings - Running in production mode: true
[2019-09-23T15:38:47.346] [WARN] Settings - CONNECTING TO A NODE VIA HTTP INSTEAD OF IPCMAIN. THIS IS LESS SECURE!!!!
[2019-09-23T15:38:47.382] [INFO] EthereumNode - undefined null 'light'
[2019-09-23T15:38:47.384] [INFO] EthereumNode - Defaults loaded: geth main light
[2019-09-23T15:38:47.454] [INFO] main - Starting in Wallet mode
[2019-09-23T15:38:53.364] [INFO] Db - Loading db: /Users/residenti/Library/Application Support/Ethereum Wallet/mist.lokidb
[2019-09-23T15:38:53.379] [INFO] Windows - Creating commonly-used windows
[2019-09-23T15:38:53.380] [INFO] Windows - Create secondary window: loading, owner: notset
2019-09-23 15:38:53.389 Ethereum Wallet[11036:1015903] *** WARNING: Textured window <AtomNSWindow: 0x7fca848db000> is getting an implicitly transparent titlebar. This will break when linking against newer SDKs. Use NSWindow's -titlebarAppearsTransparent=YES instead.
[2019-09-23T15:38:53.432] [INFO] updateChecker - Check for update...
[2019-09-23T15:38:57.639] [INFO] Sockets/node-ipc - Connect to {"hostPort":"http://localhost:8545"}
[2019-09-23T15:38:57.672] [INFO] Windows - Create primary window: main, owner: notset
[2019-09-23T15:38:57.682] [INFO] ClientBinaryManager - Initializing...
[2019-09-23T15:38:57.683] [INFO] ClientBinaryManager - Checking for new client binaries config from: https://raw.githubusercontent.com/ethereum/mist/master/clientBinaries.json
[2019-09-23T15:38:57.699] [INFO] main - Loading Interface at file:///Applications/Ethereum Wallet.app/Contents/Resources/app.asar/interface/wallet/index.html
[2019-09-23T15:38:57.705] [ERROR] main - Couldn't infer if computer automatically syncs time. Error: checkEnabled is not supported on this operating system
    at Object.checkEnabled (/Applications/Ethereum Wallet.app/Contents/Resources/app.asar/node_modules/os-timesync/index.js:97:30)
    at checkTimeSync (/Applications/Ethereum Wallet.app/Contents/Resources/app.asar/main.js:283:14)
    at onReady (/Applications/Ethereum Wallet.app/Contents/Resources/app.asar/main.js:207:3)
    at App.<anonymous> (/Applications/Ethereum Wallet.app/Contents/Resources/app.asar/main.js:168:5)
    at Generator.next (<anonymous>)
    at step (/Applications/Ethereum Wallet.app/Contents/Resources/app.asar/main.js:25:191)
    at /Applications/Ethereum Wallet.app/Contents/Resources/app.asar/main.js:25:361
    at <anonymous>
[2019-09-23T15:38:57.860] [INFO] Sockets/node-ipc - Connected!
[2019-09-23T15:38:58.352] [INFO] Sockets/3 - Connect to {"hostPort":"http://localhost:8545"}
[2019-09-23T15:38:58.484] [INFO] ClientBinaryManager - No "skippedNodeVersion.json" found.
[2019-09-23T15:38:58.486] [INFO] ClientBinaryManager - Initializing...
[2019-09-23T15:38:58.487] [INFO] ClientBinaryManager - Resolving platform...
[2019-09-23T15:38:58.487] [INFO] ClientBinaryManager - Calculating possible clients...
[2019-09-23T15:38:58.490] [INFO] ClientBinaryManager - 1 possible clients.
[2019-09-23T15:38:58.490] [INFO] ClientBinaryManager - Verifying status of all 1 possible clients...
[2019-09-23T15:38:58.492] [INFO] ClientBinaryManager - Verify Geth status ...
[2019-09-23T15:38:58.514] [INFO] Sockets/3 - Connected!
[2019-09-23T15:38:58.537] [INFO] ClientBinaryManager - Checking for Geth sanity check ...
[2019-09-23T15:38:58.538] [INFO] ClientBinaryManager - Checking for Geth sanity check ...
[2019-09-23T15:38:58.539] [INFO] ClientBinaryManager - Checking sanity for Geth ...
[2019-09-23T15:38:58.550] [INFO] ClientBinaryManager - Checking sanity for Geth ...
[2019-09-23T15:38:58.678] [ERROR] ClientBinaryManager - Sanity check failed for Geth Error: Unable to find "1.8.23" in Geth output
    at Promise.resolve.then.then.then (/Applications/Ethereum Wallet.app/Contents/Resources/app.asar/node_modules/ethereum-client-binaries/src/index.js:632:17)
    at <anonymous>
[2019-09-23T15:38:58.701] [INFO] updateChecker - App is up-to-date.
[2019-09-23T15:39:01.036] [INFO] Sockets/2 - Connect to {"hostPort":"http://localhost:8545"}
[2019-09-23T15:39:01.040] [INFO] Sockets/2 - Connected!
[2019-09-23T15:39:01.196] [INFO] (ui: popupWindow) - Meteor starting up...
[2019-09-23T15:39:02.306] [INFO] (ui: popupWindow) - Meteor starting up...
[2019-09-23T15:39:03.285] [INFO] Sockets/3 - Disconnecting...
[2019-09-23T15:39:03.290] [INFO] Sockets/3 - Connect to {"hostPort":"http://localhost:8545"}
[2019-09-23T15:39:03.293] [INFO] Sockets/3 - Connected!
[2019-09-23T15:39:04.279] [INFO] (ui: browser) - Exception in template helper: Error: Error: [number-to-bn] while converting number 2.9749999999999997e+21 to BN.js instance, error: invalid number value. Value must be an integer, hex string, BN or BigNumber instance. Note, decimals are not supported. Given value: "2.9749999999999997e+21"
    at Object.t [as toBN] (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:171:582537)
    at Object.k.formatBalance (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:183:2476)
    at updateMistBadge (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:396:365)
    at Object.totalBalance (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:382:560)
    at file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:31312
    at file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:17158
    at file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:31945
    at Function.e._withTemplateInstanceFunc (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:37654)
    at file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:31852
    at Object.p.call (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:95:1925)
    at file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:271:8362
    at i (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:30496)
    at f.View.<anonymous> (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:27388)
    at file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:18998
    at Function.e._withTemplateInstanceFunc (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:37654)
    at file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:93:18950
[2019-09-23T15:39:07.186] [ERROR] ipcProviderBackend - Send request failed { code: -32601,
  message: 'the method net_listening does not exist/is not available' }
[2019-09-23T15:39:07.191] [INFO] (ui: browser) - Restart app operation again
[2019-09-23T15:39:07.192] [INFO] (ui: browser) - Connect to node...
[2019-09-23T15:39:07.203] [INFO] (ui: browser) - Error:  {}
[2019-09-23T15:39:07.221] [ERROR] ipcProviderBackend - Send request failed { code: -32000, message: 'notifications not supported' }
[2019-09-23T15:39:07.224] [ERROR] ipcProviderBackend - Send request failed { code: -32000, message: 'notifications not supported' }
[2019-09-23T15:39:07.225] [ERROR] ipcProviderBackend - Send request failed { code: -32601,
  message: 'the method net_listening does not exist/is not available' }
[2019-09-23T15:39:07.226] [ERROR] ipcProviderBackend - Send request failed { code: -32601,
  message: 'the method net_peerCount does not exist/is not available' }
[2019-09-23T15:39:07.226] [ERROR] ipcProviderBackend - Send request failed { code: -32000, message: 'notifications not supported' }
[2019-09-23T15:39:07.241] [INFO] (ui: browser) - Error:  {}
[2019-09-23T15:39:07.242] [WARN] (ui: browser) - %cUnhandled rejection Error: Returned error: the method net_peerCount does not exist/is not available
    at Object.t [as ErrorResponse] (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:171:326449)
    at Object.<anonymous> (file:///Applications/Ethereum%20Wallet.app/Contents/Resources/app.asar/interface/wallet/90416715bed3e5b86a6da902e26b2961d182f1fe.js:171:346403)
    at <anonymous>:1264:38 color: red
[2019-09-23T15:39:07.248] [ERROR] ipcProviderBackend - Send request failed { code: -32601,
  message: 'the method net_listening does not exist/is not available' }
[2019-09-23T15:39:07.267] [INFO] (ui: browser) - Error:  {}
```

以上でスマートコントラクトの開発環境が構築できた。
