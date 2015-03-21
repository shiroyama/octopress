---
layout: post
title: "とりあえずMacでDocker入門する"
date: 2015-03-21 23:36:52 +0900
comments: true
categories: Docker
---

とりあえずMacでDocker入門したのでメモ。  
後述するブリッジインタフェースの件とかは意外に情報がなくて苦労した。

#### Dockerとは

いま流行りのコンテナ型仮想化技術のひとつ。  
これについては長くなるので別のエントリに書いておく。  
とりあえずAWSのEC2とかとはちょっと違う仮想化技術だということが伝わればよい。

#### 事前準備

* [Homebrew](http://brew.sh/ "Homebrew")
* [VirtualBox](https://www.virtualbox.org/ "VirtualBox")

#### 下準備

深く考えずに下をコピペ

```bash
$ brew update
$ brew tap homebrew/binary
$ brew install docker
$ brew install boot2docker
$ boot2docker init
$ boot2docker up
```

`boot2docker up` したら docker server に接続するための情報が出力されるので `.zshrc` に書いとく。

```bash
$ cat<<'EOS'>~/.zshrc
> # boot2docker
> export DOCKER_HOST=tcp://192.168.59.103:2376
> export DOCKER_CERT_PATH=/Users/shiroyama/.boot2docker/certs/boot2docker-vm
> export DOCKER_TLS_VERIFY=1
EOS

$ source ~/.zshrc
```

こんなのが出たらOK。まだコンテナ追加してないのでエラーが出ないことが確認できれば充分。

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

余談だけど [boot2docker](http://boot2docker.io/ "boot2docker") は何者かというと、MacでDockerを使うための橋渡し。DockerはLinuxカーネルの機能を使うのでMacにはネイティブ対応していない。

実はboot2dockerはDockerが使えるLinuxインスタンスのVMインスタンス+周辺スクリプト群にすぎない。仮想化技術を使うために仮想化技術を使うのだ。まあMacでちょっと試す分には大変ありがたい。

#### boot2dockerにブリッジインタフェースを追加

このあと実際にDockerコンテナを作っていくんだけど、その前に重要なステップがあって、以下の手順でboot2dockerにブリッジインタフェースを追加しておく。これをしておかないとDockerコンテナとMacとで通信ができないので不便なことこの上ない。

1. `boot2docker down`
1. VirtualBoxを起動し`boot2docker-vm`を選択
1. 設定からネットワークを選んでアダプター3を有効化しブリッジアダプターを選択する。
1. `boot2docker up`

{% img /images/docker/boot2docker.jpg 'boot2docker bridge' 'boot2docker bridge' %}

[boot2docker上のコンテナに別ホストからアクセスする](http://qiita.com/nyamage/items/fad845bc5e4ce3cf33eb "boot2docker上のコンテナに別ホストからアクセスする") とかが参考になるよ。

#### Dockerコンテナ is 何

Dockerコンテナは雑に言うとVMインスタンスに相当するもの。  
ただ、DockerコンテナをXenとかKVMみたいな、いわゆる普通のLinuxのVMインスタンスだと思って使うと大変な目に遭う（つくりも思想もまったく別物）ので、これも詳しくは別エントリに譲りたい。

#### Dockerfile

DockerコンテナはDockerfileという設定ファイルにしたがってセットアップされて起動する。

Dockerfileの中に、

* ベースとなるイメージはUbuntuで...
* Nginxインストールして...
* 80番でLISTENして...
* 起動！

みたいなことを順番に書いていくというわけ。

#### Dockerfile書いてみる

なんか適当に `test` みたいなディレクトリ掘ってそこにDockerfileというファイル名で作成する。

```vim
FROM ubuntu:latest

RUN apt-get update && apt-get install -y nginx

EXPOSE 80
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
```

なんとなく意味分かるんじゃなかろうか。

Dockerfileはそれだけで1エントリ書くつもりなのでそこで詳しく書く。

#### Dockerコンテナを起動

```bash
$ docker build -t firstdocker/nginx .

$ docker images
REPOSITORY           TAG                  IMAGE ID            CREATED             VIRTUAL SIZE
firstdocker/nginx    latest               d9fcf4f72bcd        5 seconds ago       227 MB

$ docker run -d -p 80:80 firstdocker/nginx
```

##### docker build
まず、1行目で `docker build` してDockerfileからDockerイメージを作る。DockerイメージはDockerコンテナのもとになるものである。

`-t` オプションで後から参照しやすいようにタグをつけている。`レポジトリ名/バージョン`が慣習っぽいけど僕は`サービス名/コンテナの役割`みたいな感じで付けてる。まあ公式サイト見てください。

最後の `.` でカレントディレクトリにあるDockerfileをビルドする。

エラーなくビルドできたら `docker images` で確認。

##### docker run

最後に `docker run -d -p 80:80 firstdocker/nginx` でDockerコンテナを起動している。

`-d` オプションでDockerコンテナをバックグラウンドで起動。

`-p` オプションで `dockerサーバのポート:dockerコンテナのポート` のようにして、ポートのバインディングをする。今回のケースだと、NginxがインストールされたDockerコンテナの80番ポートをDockerサーバ、つまりboot2dockerの80番ポートに割り当てるというわけだ。

最後の `firstdocker/nginx` で今から起動するDockerコンテナの元になるイメージを指定している。もちろん最前こしらえたものを指定している。

`docker run` は `-d` オプションの代わりに

```bash
$ docker run -i -t -p 80:80 firstdocker/nginx bash
```

こんな風にして、Dockerコンテナを立ち上げつつコンテナにBASHをログインシェルとしてログインするというようなこともできる。ただしこの場合はDockerfileに書いた `CMD ["/usr/sbin/nginx", "-g", "daemon off;"]` が実行されないのでNginxが起動しない。

この辺りはDockerfileだけを深く掘り下げたエントリで詳しく書くのでいまはあまり気にしないで欲しい。

##### 確認してみよう！

`boot2docker ssh` して `ifconfig` してみよう。  
上の方でboot2dockerのネットワークアダプター3をブリッジにしたので `eth2` の情報をメモしておく。

```bash
$ ifconfig
...
eth2      Link encap:Ethernet  HWaddr 08:00:27:D0:9B:9F
          inet addr:192.168.11.111  Bcast:192.168.11.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fed0:9b9f/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:76 errors:0 dropped:0 overruns:0 frame:0
          TX packets:13 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:42936 (41.9 KiB)  TX bytes:2559 (2.4 KiB)
          Interrupt:17 Base address:0xd060
```

僕の環境では `192.168.11.111` がboot2dockerの（Macから到達可能な）IPアドレスだ。メモしておく。

Macで任意のブラウザを立ち上げて、`192.168.11.111` にアクセスしてNginxのデフォルト画面が表示されたらOKだ！

##### dockerコンテナに接続

`docker ps` で起動中のコンテナを確認してみる。

```bash
$ docker ps
CONTAINER ID        IMAGE                      COMMAND                CREATED             STATUS              PORTS                NAMES
703409c1d96c        firstdocker/nginx:latest   "/usr/sbin/nginx -g    46 minutes ago      Up 7 seconds        0.0.0.0:80->80/tcp   trusting_cori
```

先ほどのコンテナが起動している。

停まっているコンテナは `docker start [NAME]` で起動できる。他にも `start/stop/restart/kill` など色々あるが、だいたい名前をみて想像がつくと思う。詳しくは `--help` してみると分かる。

で、起動しているインスタンスには以下のようにして接続する。

```bash
$ docker exec -it trusting_cori bash
```

`docker exec -it` まではとりあえずおまじないでいいです。

`trusting_cori` は対象のコンテナ名を指定している。`docker ps` の結果を参照のこと。

最後の `bash` でBASHをログインシェルにしてログインすることを意味している。

実行したら普通のLinuxのようにコンテナ内をウロウロできたはずだ。

余談だが、`docker exec` はログインのためのものではなく、コンテナの任意のコマンドを実行できる命令である。つまり

```bash
$ docker exec -it trusting_cori cat /var/log/nginx/access.log

192.168.11.108 - - [21/Mar/2015:15:22:54 +0000] "GET /favicon.ico HTTP/1.1" 404 208 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.101 Safari/537.36"
192.168.11.108 - - [21/Mar/2015:16:09:20 +0000] "GET / HTTP/1.1" 200 396 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.101 Safari/537.36"
192.168.11.108 - - [21/Mar/2015:16:09:20 +0000] "GET /favicon.ico HTTP/1.1" 404 208 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.101 Safari/537.36"
```

こんな風に外からログをみたりするのも簡単にできる。

#### まとめ

ここまでで、多分コピペするだけなら15分ぐらいでNginxをDockerコンテナ上に立てられたはずだ。なんて便利で簡単なんだと思った方もいるかも知れないが、**甘い**。

入門記事はこのエントリと同じぐらいの内容をサラッと書いてあることが多いと思うが、Dockerの本当の素晴らしさと苦しみはこの先にある。僕が本当に書きたかったエントリはそれである。そのエントリを書きたいがために、わざわざQiitaでも見りゃなんぼでも載っているような内容を書いたわけだ。

次のエントリは**「Dockerは何であって何でないか」**というようなエントリになるはずだ。

そのエントリでNginxの起動がなんで `CMD ["/usr/sbin/nginx", "-g", "daemon off;"]` みたいなけったいな方法なのか、Dockerコンテナにログインして `ps aux | grep nginx` とかしてみたら、なんでNginxのプロセスIDが1なのか、そういうことを書いていきたいと思う。

#### 参考書籍

[Docker入門 Immutable Infrastructureを実現する](http://www.amazon.co.jp/Docker%E5%85%A5%E9%96%80-Immutable-Infrastructure%E3%82%92%E5%AE%9F%E7%8F%BE%E3%81%99%E3%82%8B-%E6%9D%BE%E5%8E%9F%E8%B1%8A-ebook/dp/B00JWM4W2E)

少し情報が古くなってる部分があるけどこのエントリよりは200倍ぐらい網羅的に色んな事が解説されてるので600円の投資対効果はかなり高いと思う。何も知識のない人はひと通り読むといいかも。
