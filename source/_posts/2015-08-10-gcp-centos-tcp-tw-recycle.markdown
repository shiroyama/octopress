---
layout: post
title: "GCPのCentOSでtcp_tw_recycleが有効になっていてハマった"
date: 2015-08-10 01:10:51 +0900
comments: true
categories: GCP GCE CentOS
---

Google Cloud Platform（以下GCP）を評価中であるが、Google Compute Engine（以下GCE）でCentOS 6をイメージとして選択してインスタンスを起動してSocket.IO環境を構築したところ、AndroidとGCE間で **時々** 原因不明のSSLハンドシェイク失敗に悩まされ非常にハマった。

結論から言うと、当該インスタンスでは `net.ipv4.tcp_tw_recycle` が有効になっていた。これを無効にすることで問題は解決した。

```bash
% sysctl net.ipv4.tcp_tw_recycle
net.ipv4.tcp_tw_recycle = 1

% sudo sysctl -w net.ipv4.tcp_tw_recycle=0
% sysctl net.ipv4.tcp_tw_recycle
net.ipv4.tcp_tw_recycle = 0
```

設定を永続化させたかったら `/etc/sysctl.conf` に書けば良い。

ちなみに `net.ipv4.tcp_tw_recycle` はTIME_WAIT状態のソケットを効率的に再利用するためのLinuxカーネル特有の仕組みとのこと。  
これが有効の場合、TCPパケットのタイムスタンプ情報を見て同一IPから新しいパケットが届くと古いソケットを開放するらしい。  
ただ、同一NAT下の複数端末が同時に接続しに来た場合に各ノードを区別できずに単純に古いものをドロップするような挙動をすることがあるようで、今回の僕のケースはそれに該当するようだ。

（※この辺り門外漢なので記述が正確でなかったらすみません。）

参考までに、AWSのAmazon Linuxの今日時点での最新である`Amazon Linux AMI 2015.03`で確認したところ、 `net.ipv4.tcp_tw_recycle = 0` となっており無効であるようだ。  
個人的にはこれに合わせておくと無難だろうと判断した。

なお、GCEのどのイメージでも同様かどうかは一切確認してない。CentOS 6なんていう古いのを選んだのも、Amazon Linuxからの移行コストが一番少なそうだからで、普通に新規構築するならCentOSみたいな保守的で新陳代謝の遅いディストロは特に選択するメリットも無い気がする。

ただ、この問題そのものは原因究明に非常〜〜〜に苦労した（WireSharkやらtcpdumpやら使いまくった。途中マジで死のうかと思った）ので、同じような人が居るかも知れないのでブログにまとめておく。

以下参考リンク

* [Why would a server not send a SYN/ACK packet in response to a SYN packet](http://serverfault.com/questions/235965/why-would-a-server-not-send-a-syn-ack-packet-in-response-to-a-syn-packet "Why would a server not send a SYN/ACK packet in response to a SYN packet")
* [「net.ipv4.tcp_tw_recycle」を有効にするのは（場合によっては）やめた方がいい](http://d.hatena.ne.jp/pullphone/20120511/1336722675 "「net.ipv4.tcp_tw_recycle」を有効にするのは（場合によっては）やめた方がいい")
* [NAT環境下では net.ipv4.tcp_timestamps = 0 する](http://blog.kamipo.net/entry/20110401/1301660084 "NAT環境下では net.ipv4.tcp_timestamps = 0 する")
