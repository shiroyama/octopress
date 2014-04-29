---
layout: post
title: "Android エンジニアが DeployGate を知らないのは恥と心得よ"
date: 2014-03-18 23:43:44 +0900
comments: true
categories: Android, DeployGate
---

[DeployGate](https://deploygate.com/?locale=ja "DeployGate") は Android 開発においてもはや欠かすことが出来ない。  
ところが未だに DeployGate を使っていない Android エンジニアを非常に多く見かける。  
これは、デザイナが Photoshop を知らないに等しい勿体なさである。

### DeployGate とは何か

DeployGate とは Android の apk を爆速でテスト配布するためのサービスである。

{% img /images/deploygate/deploygate01.jpg 'deploygate preview' 'deploygate preview' %}

最初に申し上げておきたいのが、DeployGate は ** [Google Play Developer Console](https://play.google.com/apps/publish/) のアルファ配布／ベータ配布とは目的が異なる。**  
そしてこれも断言しておきたいが、多くの Android 開発者に必要なのは DeployGate の方である。 順番に見て行きたいと思う。

後半では DeployGate の使い方も実際に解説してあるので是非参考にしていただきたい。

### テスト配布とは何か

テスト配布と言っても大きく2種類あると考えている。

1. 開発中のアプリをチームメンバーやプロダクトオーナーに素早く配布し触ってもらう。部分的に確認してもらうだけなので未実装部分があっても問題ない。
1. もうほとんど完成形のアプリを一部の想定カスタマーに実際に使ってもらって本リリース前の最後の微調整をする。

端的に言うと、前者に対応するのが DeplyGate であり、後者に対応するのが Developer Console である。

### Developer Console のベータ配布ではダメなのか

Google 謹製の Developer Console を使ったアルファ／ベータ配布で何がダメなのだろうか？  
その理由は一度でも Developer Console でテスト配布したことのある人は痛いほどご存知のはずだ。

#### 1. 用意するものが多すぎる

Developer Console でベータ配布するには本番アプリとまったく同じ準備が必要である。  
アプリアイコンが1つ、スクリーンショットが1つ、プロモーション用の画像もたしか2枚、説明文、公開範囲等々…  
詳しくは当該サイトを参照して欲しいが、ちょっと同僚に試してもらうには準備が大げさすぎる。

#### 2. 配布されるまでに時間が掛かり過ぎる

Developer Console でのベータ配布は本番アプリのストア配布と公開範囲を除いては何ら変わりがない。  
従って apk ファイルをアップロードしてから Google の CDN に載り、実際にダウンロード出来るようになるまでゆうに3時間は掛かる。  
もし何かミスがあれば apk ファイルを上げ直し、また3時間待つハメになるのだ。とてもやっていられない。

#### 3. バージョン番号を上げないとアップロードし直せない

Developer Console で修正した apk をアップロードするには、Android が内部で持っている *versionCode* を上げてやらねばならない。  
本来はアプリの純然たる識別子としてのバージョンコードが、配布ミスや漏れのたびに数字が上がっていくという残念な状況になるのだ。

これらはすべて、ベータ配布が悪いというわけではなく、前述のとおり思想の違いである。**開発サイクルを短く回すためにベータ配布を使うのがそもそも使い方として間違っている**のだ。

### そこで DeployGate である

DeployGate では以下の様なことが出来る。

* 配布したいメンバーを追加し30秒で apk 配布。難しい操作なし！
* vesionCode 上げる必要なし！新しい apk を何も考えずにアップロードすると新しいビルドとしてすぐさま配布される
* テスト配布のバージョンアップが一瞬でプッシュ通知されてくるのでどこに居ても見逃さない
* 簡単に組み込める SDK を開発者が仕込んでおくと、**配布先のメンバーのクラッシュレポートが自動的に送られてくる！**
* Jenkins 等の CI ツールとの連携が容易
* 便利なコマンドラインツール群
* **無料で使える**（有料プランとの違いは後述）

DeployGate では小難しい操作や七面倒臭い準備は何一つ必要ない。  
誰にでもすぐに使い始められる。

DeployGate はユーザ体験も非常に良いので、チーム内の開発サイクルを速めるのみならず、実際にリリース直前のお客さんに触ってもらう時も実にスムーズだ。  
今まで使ってなかったなんて本当にもったいない！

### DeployGate を使ってみよう

早速 DeployGate を使ってみよう。もちろん無料だ。  
[https://deploygate.com/?locale=ja](https://deploygate.com/?locale=ja) にアクセスして「無料で試してみる」をクリックする。

{% img /images/deploygate/deploygate02.jpg 'deploygate sign-up' 'deploygate sign-up' %}

サインアップは30秒で終わる。  
望むならば GitHub アカウント等を使っても良い。

{% img /images/deploygate/deploygate03.jpg 'deploygate sign-up 02' 'deploygate sign-up 02' %}

早速 apk ファイルをアップロードしてみよう。  
「アップロード」ボタンを押して apk ファイルの場所を指定するだけだ。

{% img /images/deploygate/deploygate04.jpg 'deploygate upload' 'deploygate upload' %}

このバージョンの説明を簡単に記入し「アップロード」ボタンをクリック。

{% img /images/deploygate/deploygate05.jpg 'deploygate upload 02' 'deploygate upload 02' %}

アップロードが完了した。  
驚くべきことに、すぐさま読み取れる QR コードと、メールで共有したり出来る静的なリンクが生成されている。

{% img /images/deploygate/deploygate06.jpg 'deploygate upload complete' 'deploygate upload complete' %}

### DeployGate から apk をインストールしてみよう

では早速 apk をインストールしてみよう。  
まずは Play ストアから DeployGate をインストールする。

{% img /images/deploygate/deploygate07.jpg 'install deploygate' 'install deploygate' %}

DeployGate を起動しよう。  
ここでは試しに先ほどの画面の QR コードを読み取って見るとしよう。

{% img /images/deploygate/deploygate08.jpg 'deploygate QR code' 'deploygate QR code' %}

QR コードリーダーが立ち上がり、先ほどの画面にかざすだけでアプリの情報を取得できる。

{% img /images/deploygate/deploygate09.jpg 'deploygate QR code 02' 'deploygate QR code 02' %}

アカウントがなければここからも登録できるし、ログインも出来る。

{% img /images/deploygate/deploygate10.jpg 'deploygate login' 'deploygate login' %}

自分のアカウントでログインしよう。

{% img /images/deploygate/deploygate11.jpg 'deploygate login 02' 'deploygate login 02' %}

アプリが表示された！  
「インストール」をクリックしてインストールしてみよう。  
以下、インストール完了まで一本道なので補足は省略する。

{% img /images/deploygate/deploygate12.jpg 'deploygate install 01' 'deploygate install 01' %}
{% img /images/deploygate/deploygate13.jpg 'deploygate install 02' 'deploygate install 02' %}
{% img /images/deploygate/deploygate14.jpg 'deploygate install 03' 'deploygate install 03' %}
{% img /images/deploygate/deploygate15.jpg 'deploygate install 04' 'deploygate install 04' %}
{% img /images/deploygate/deploygate16.jpg 'deploygate install 05' 'deploygate install 05' %}
{% img /images/deploygate/deploygate17.jpg 'deploygate install 06' 'deploygate install 06' %}

見事インストールされた！

もしかしたら開発中のアプリは「設定」から「提供元不明のアプリ」にチェックを入れないとインストールできないかもしれないが、Android 開発者の皆さんには釈迦に説法だろう。

{% img /images/deploygate/deploygate18.jpg 'deploygate unknown app' 'deploygate unknown app' %}

### apk を更新してみよう

Android アプリに修正を加え、新しいバージョンをテスト配布してみよう。

これまでと同じように DeployGate のログイン後画面から最新の apk をアップロードする。

{% img /images/deploygate/deploygate19.jpg 'deploygate update app' 'deploygate update app' %}

アップロードすると即座に更新がプッシュ通知される。

{% img /images/deploygate/deploygate20.jpg 'deploygate push notification' 'deploygate push notification' %}

更新ボタンを押すだけで最新の配布を試すことが出来る。

{% img /images/deploygate/deploygate21.jpg 'deploygate update app 02' 'deploygate update app 02' %}

すぐさま新機能の更新を確認することが出来た！なんと簡単なのか！

{% img /images/deploygate/deploygate22.jpg 'deploygate update app 03' 'deploygate update app 03' %}

### 開発者やテスタを追加しよう

自分以外の開発者やテスターの方ににテスト配布してみよう。これもとても簡単だ。

招待したい人のメールアドレスを入力し、開発者かテスタか選ぶ。  
開発者として追加した場合には今自分がやっているのと同等の権限がメンバに与えられる。  
テスタとして追加した場合にはアプリのテストのみが行える。

{% img /images/deploygate/deploygate23.jpg 'deploygate test distribution 01' 'deploygate test distribution 01' %}

招待されるとメールが届くのでリンクをクリックする。

{% img /images/deploygate/deploygate24.jpg 'deploygate test distribution 02' 'deploygate test distribution 02' %}

メンバとして追加された！後はこれまでに見てきたのと同じようにテスト配布を受け取ることができる！説明の必要がないほど直感的で簡単である。

{% img /images/deploygate/deploygate25.jpg 'deploygate test distribution 03' 'deploygate test distribution 03' %}

### まとめ

以上が DeployGate の最も基本的な使い方である。  
DeployGate がどれだけ簡単でどれほど Android 開発に必要不可欠かお分かりいただけたと思う。

次エントリでは、Android アプリに DeployGate SDK を組み込んで、リモートからクラッシュレポートを収集する方法等を解説したい。
