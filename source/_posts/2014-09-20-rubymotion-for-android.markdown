---
layout: post
title: "RubyMotion for Android を使ってみる"
date: 2014-09-20 15:53:05 +0900
comments: true
categories: Android RubyMotion Ruby RubyKaigi
---

9/16日、[RubyMotion が Android に対応し public Beta として公開された。](http://blog.rubymotion.com/post/97668474211/announcing-the-public-beta-of-rubymotion-for-android "公式ブログ")

9/18〜20まで開催された [RubyKaigi 2014](http://rubykaigi.org/2014/ "RubyKaigi 2014") でも [RubyMotion for Android に関するセッション](http://rubykaigi.org/2014/presentation/S-LaurentSansonetti "Inside RubyMotion for Android") があり、かなり感銘を受けたのでブログにまとめることにする。

### RubyMotion とは何か

すごく簡単に言うと iOS や OSX, そして Android のネイティブアプリを Ruby で書けるというツール群である。

http://www.rubymotion.com/

Titanium や PhoneGap とは明確に異なり、各ランタイムが直接解釈できるネイティブコードを生成するのが特徴。従ってパフォーマンス的なオーバーヘッドがほとんどなく ※1 、利用できる OS の API にも基本的に制限がない。

何より Ruby で書けるのである。なんて素晴らしいんだ。

### 利用方法

公式サイトの英語を読むのがめんどくさい人のために2014年9月時点でのライセンス形態等について箇条書きしておく。

* **有料。**199.99ドル
  * 1年間のバグフィクス・改良などのアップデートを受けられる
  * 1年間のサポートのためのプライベートチケットを切ることができる
  * 自分の理解では1度買えば1年経つとアップデートは受けられないがそのまま使い続けられる
* 30日間はトライアル期間として気に入らなければいつでもやめることができる
* 今は学割的なものはない
* 1年経つと99.99ドルで（アップデート等の）ライセンスを延長できる
* 言い忘れたけど OSX の動く Mac が必要なのでそれ以外の環境の方は残念

これを高いと見るか安いと見るかは価値観次第だと思うが、トライアル期間があるので少しでも興味がある人は一度試してみると良いと思う。（最初にカードを登録する必要あり。30日以内のキャンセルは "返金扱い" なのかそれとも "30日後に初めて課金される" のかは知らない。興味もない）

### 使ってみる

[公式サイト](http://sites.fastspring.com/hipbyte/product/rubymotion "公式サイト") から早速購入してみよう。RubyKaigi 期間中はなんと15%オフ（毎年恒例らしい）なのだが、ブログにまとめるのをモタモタしているうちに閉会してしまった…

{% img /images/rubymotion/rubymotion01.jpg 'buy rubymotion' 'buy rubymotion' %}

購入し終わると登録したメールアドレス宛にライセンスキーとインストーラのダウンロードリンクが届く。 ダウンロードリンクは忘れがちなのでメールは保存しておこう。

ダブルクリックして指示に従っていると何の問題もなくインストールは完了する。

Xcode や AndroidStudio と違い、専用の（エディタを含んだ GUI の）IDE 環境がインストールされる訳ではない。RubyMotion は CUI のビルドツール群という印象に近い。

ツール群は

```bash
% /Library/RubyMotion/
```

以下にインストールされ、最も使われる（というかこれしか使ったことがない）motion コマンドは

```bash
% /usr/bin/motion
```

にリンクされる。普通にしているとここにはパスが通っているはずなのでコンソールで motion とタイプするだけですぐに利用できるはずだ。

### Hello from RubyMotion for Android

それでは RubyMotion for Android を使って Hello World をしてみよう。  
[公式の Getting Started をなぞるだけ](http://www.rubymotion.com/developer-center/guides/getting-started/#_android "Getting Started") で実に簡単に動かすことができるが、例によって英語の説明を読みたくない人のために箇条書きにする。

#### 1. プレビュー版をインストール

今日時点で Android 版は public Beta という扱いなので、標準のコンポーネントには含まれていない。以下のコマンドでインストールする。

```bash
% sudo motion update --pre
```

ファイルは以下のパスにインストールされるが普通に使っている限り意識することはない。

```bash
% /Library/RubyMotionPre
```

#### 2. JDK をインストール

JDK 6 をインストールする必要がある。

>7でも動くけど6を推奨

みたいなこと書いてあるけど、**7だと動かない場合があった** ※2 ので言うとおりにすることを強くおすすめする。

OpenJDK か Oracle の JDK か、みたいなことは書いてないが、公式サイトからは [Apple の Java for OSX 2014-001](http://support.apple.com/kb/DL1572 "Java for OSX 2014-001") というのにリンクが張ってあったので素直にこれを入れると良いだろう。

Mac に複数の JDK が入っている場合はやっかいで、僕の知る限り Java には rbenv 的なランタイムのバージョン管理とサンドボックス化を提供してくれる仕組みはない（最近の言語でこういった仕組みがない言語の方がむしろ珍しいよね…まったくトホホだ）ので、自分で JAVA_HOME を変えてやる必要がある。

```bash
% ls -l /System/Library/Frameworks/JavaVM.framework/Versions
```

とかしてみて、**1.6** がある且つ複数のバージョンが入っているようなら .zshrc とかに

```bash
% cat<<'EOS'>>~/.zshrc
export JAVA_HOME=`/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/java_home -v "1.6"`
PATH=${JAVA_HOME}/bin:${PATH}
EOS

% source ~/.zshrc

% java -version
java version "1.6.0_65"
Java(TM) SE Runtime Environment (build 1.6.0_65-b14-466.1-11M4716)
Java HotSpot(TM) 64-Bit Server VM (build 20.65-b04-466.1, mixed mode)
```

のようにして java 1.6 が使われていることを確認しておこう。[この Qiita](http://qiita.com/ringo/items/db58b34dc02a941b297f "OSXでJavaのバージョンを切り替える") を参考にすると良い。

#### 3. Androiid SDK をインストール

公式サイトの例に従い、まずは RubyMotion for Android の作業ディレクトリを作成する。

```bash
% mkdir ~/android-rubymotion
```

次に公式サイトでは [Eclipse ADT](http://developer.android.com/sdk/index.html "Eclipse ADT") をインストールしてその sdk をコピーするように書いてあるが、今更 Eclipse を使っている人も居ないと思うので [Android Studio](https://developer.android.com/sdk/installing/studio.html "Android Studio") の sdk をシンボリックリンクで利用すると良い。SDK さえ入れられれば方法は問わない。 ※3

```bash
% cd ~/android-rubymotion
% ln -s /Applications/Android\ Studio.app/sdk sdk
% ls sdk/
add-ons        build-tools    docs           extras         platform-tools platforms      samples        sources        system-images  temp           tools
```

#### 4. Androiid NDK をインストール

ネイティブコンパイルのために [Mac OS X 64-bit NDK for 32-bit target](https://developer.android.com/tools/sdk/ndk/index.html "Android NDK") を入れる必要がある。

間違えやすいが、必ず **Platform (32-bit target)** かつ **Mac OS X 64-bit** の NDK でなくてはならない。さもなくば動かない旨明記してある。

ダウンロードしたら展開して sdk と同じディレクトリにコピーする。

```bash
% tar jxvf android-ndk32-r10b-darwin-x86_64.tar.bz2
% cp -r android-ndk-r10 ~/android-rubymotion/ndk
% cd ~/android-rubymotion
% ls ndk/
GNUmakefile               build                     find-win-host.cmd         ndk-depends               ndk-gdb-py.cmd            ndk-which                 remove-windows-symlink.sh tests
README.TXT                docs                      ndk-build                 ndk-gdb                   ndk-gdb.py                platforms                 samples                   toolchains
RELEASE.TXT               documentation.html        ndk-build.cmd             ndk-gdb-py                ndk-stack                 prebuilt                  sources
```

最後に SDK と NDK を環境変数に登録する。

```bash
% cat<<'EOS'>>~/.zshrc
export RUBYMOTION_ANDROID_SDK=~/android-rubymotion/sdk
export RUBYMOTION_ANDROID_NDK=~/android-rubymotion/ndk
EOS

% source ~/.zshrc

% env | grep RUBYMOTION_ANDROID
RUBYMOTION_ANDROID_SDK=/Users/shiroyama/android-rubymotion/sdk
RUBYMOTION_ANDROID_NDK=/Users/shiroyama/android-rubymotion/ndk
```

こんな感じになっていればOK。

#### 5. 必要なバージョンの SDK パッケージを入手

自分の持っている Android のバージョン (API Level) に合わせてビルドするため、SDK Manager からパッケージを入手する。自分の持っている Android の API レベルは [このへん](http://so-zou.jp/mobile-app/tech/android/data/api-level/ "AndroidのバージョンとAPIレベルの対応関係") を参考にすれば良いだろう。  
例えば Android 4.4 KitKat の端末を持っている場合は API Level 19 ということになる。

確認し終わったら、

```bash
% cd ~/android-rubymotion
% ./sdk/tools/android
```

とすると下図のようなウィンドウが表示される。

{% img /images/rubymotion/rubymotion02.jpg 'SDK Manager' 'SDK Manager' %}

対応する API レベルのパッケージをとりあえず全部入れておけば間違いない。 ※4

#### 6. 端末を開発モードへ

既に Android 開発をしたことのある人はここは読み飛ばしてください。  
初めての人は [このへん](http://smhn.info/201311-android-developer-option "Androidの「開発者向けオプション」を表示する方法") を参考にしながら開発者モードを表示して USB デバッグを有効にしてください。

いよいよ準備完了！

#### 7. Hello World

任意の場所で以下のコマンドを実行すると、RubyMotion for Android 用のテンプレートプロジェクトが生成される。

```bash
% motion create --template=android Hello
% cd Hello
```

Android 端末が USB で接続され、開発用に認識されていることを確認してから ※5 いよいよ実行してみよう。

```bash
% rake device
```

コンソールに以下のようなログが表示され、Android 端末にタイトルが Hello とだけ書かれた真っ黒な画面が表示されたら成功である。

```bash
% rake device

   Compile ./app/main_activity.rb
    Create ./build/Development-19/lib/armeabi/libpayload.so
    Create ./build/Development-19/lib/armeabi/gdbserver
    Create ./build/Development-19/lib/armeabi/gdb.setup
    Create ./build/Development-19/AndroidManifest.xml
    Create ./build/Development-19/classes/com/yourcompany/hello/MainActivity.class
    Create ./build/Development-19/classes.dex
    Create ./build/Development-19/Hello.apk
      Sign ./build/Development-19/Hello.apk
     Align ./build/Development-19/Hello.apk
   Install ./build/Development-19/Hello.apk
6102 KB/s (633859 bytes in 0.101s)
     Start com.yourcompany.hello/.MainActivity
--------- beginning of /dev/log/main
--------- beginning of /dev/log/system
```

{% img /images/rubymotion/rubymotion03.png 'Hello World 01' 'Hello World 01' %}

もし以下の様なエラーが表示されたら、期待した API バージョンに対してビルドされていない。

```bash
% rake device

    ERROR! It looks like your version of the NDK does not support API level L. Switch to a lower API level or install a more recent NDK.
```

先ほどの Hello ディレクトリ以下の Rakefile を任意のエディタで以下のように編集しよう。  
10行目がミソだ。

```ruby
% vim Rakefile

# -*- coding: utf-8 -*-
$:.unshift("/Library/RubyMotionPre/lib")
require 'motion/project/template/android'

Motion::Project::App.setup do |app|
  # Use `rake config' to see complete project settings.
  app.name = 'Hello'
  app.api_version = '19'
end
```

もう一度 rake device してうまくいくか試す。

以上で Hello World は終了である。ここまではそれほど難しくないはずだ。

### はじめてのコントローラ

いよいよ自分でコードを書いていく。

```bash
% app/main_activity.rb
```

を任意のエディタで開き、以下のように編集してみよう。

```ruby
% vim app/main_activity.rb

class MainActivity < Android::App::Activity
  def onCreate(savedInstanceState)
    super
    text_view = Android::Widget::TextView.new(self)
    text_view.text = 'Hello RubyMotion!'
    self.contentView = text_view
  end
end
```

保存して rake device すると以下のような画面が表示されるはずだ。

{% img /images/rubymotion/rubymotion04.png 'Hello World 02' 'Hello World 02' %}

コードの内容は簡単で、

1. 6行目で TextView のインスタンスを生成し
1. 7行目で文字列を設定し
1. 8行目でそれをこの Activity にセットしている

というわけである。なんだか Android 1.5 の頃によく見た入門サイトのようなコードだ。

後で少し補足するが、**RubyMotion for Android はこれ以上でも以下でもない。**  
一旦次に読み進めて欲しい。

### 素晴らしき REPL

RubyMotion には素晴らしい REPL ※6 が備わっている。  
先ほどのコードを次のように変更してみよう。

```ruby
% vim app/main_activity.rb

class MainActivity < Android::App::Activity
  def onCreate(savedInstanceState)
    super
    text_view = Android::Widget::TextView.new(self)
    text_view.text = 'Hello RubyMotion!'
    text_view.textSize = 10
    text_view.setId 12345
    self.contentView = text_view
  end
end
```

1. 8行目でテキストサイズを指定し
1. 9行目で View に対して一意な ID を割り振っている

再び実行するが、今度はコンソールで

1. ```text = self.findViewById 12345```
1. ```text.setTextSize 100```

のように入力してみよう。

```bash
% rake device

>> text = self.findViewById 12345
=> #<android.widget.TextView:0x50600025>
>> text.setTextSize 100
=> #<android.widget.TextView:0x50800025>
>>
```

{% img /images/rubymotion/rubymotion05.png 'Hello World 03' 'Hello World 03' %}

なんと、対話的に TextView オブジェクトを取り出し、あまつさえ値をセットしなおして即座に端末に反映されてしまった！

**SUPER COOOOOOOOOOL!!!**

一旦これだけのために RubyMotion を使ってみて欲しいぐらいだ。マジで。

### 更に踏み込んでいく

実際のところ、これだけでは実用的なアプリではない。

1. 現実問題、レイアウトをコードで指定することは実際の Android 開発現場ではほとんどありえない
1. 外部ライブラリどうやって使うの？
1. リソースどこで指定するの？

などなど問題は山積みである。

幸い、RubyKaigi で開発者の方と直接お話しをする機会があり、世の中に数多存在するネイティブ Java で書かれたライブラリたちは RubyMotion で使えないのか質問したところ「使える」という回答をもらっており、やり方もざっくりお聞きしてある。

その辺り、どんどん踏み込んで行きたいのだが、いい加減エントリも長くなってきたので別エントリを立てて解説したいと思う。

RubyMotion for Android はまだドキュメントも満足に揃っていないので、この辺りを自分で調べるのは少々骨が折れるのだが、幸い GitHub に [サンプルプロジェクト](https://github.com/HipByte/RubyMotionSamples "RubyMotion Samples") を上げてくれているので、興味のある人は是非 clone して実行してみて欲しい。

### RubyMotion for Android は世界を救うか

というわけで、本エントリはまとめに入るが、RubyMotion はネイティブ開発の銀の弾丸になるだろうか？

答えはもちろんノーだ。

先ほどのコントローラのコードを思い出して欲しい。  
あのコードは、Ruby を生まれてこの方みたことなくても Android 開発経験があれば読める。  
逆に言えば、Rubyist でも Android 未経験であればあのコードは（全く理解不能ということはないだろうが）良くわからないはずだ。

**RubyMotion はそれさえあれば Android/iOS のことを何も知らなくてもアプリを開発できるものでは全くない。**

ライフライクル、イベントハンドリング、非同期処理などなど、OS のフレームワークをもろに意識したコードを書く必要がある。結局はネイティブ開発の知識なしには絶対に成り立たない。

しかしながら、それがどうした？とも言いたい。

* OS と密接に関わる部分以外は柔軟で記述性の高い Ruby で書ける
* そのような部分は Android/iOS で共通のライブラリとして実装可能
* **Ruby で書くと楽しい！**

僕はこれらの現実を踏まえた上で、それでも RubyMotion が痛く気に入った。同じように感じる人が RubyMotion を触ってみるきっかけになれば幸いである。

以下、補足。

### RubyMotion 補足

ライセンスが有効なうちは以下のコマンドで定期的に RubyMotion をアップデートしよう。開発はかなり活発なようだ。

```bash
% sudo motion update
```

バージョンは以下で確認することができる。

```bash
% motion --version
2.24
```

バージョンは一日に一回サーバと通信してチェックしているとのことで、ライセンス失効が近付くとコマンドラインで教えてくれる。必要に応じて更新すると良いだろう。

その他、サポートが必要な場合は以下のコマンドでブラウザのサポートページへ移動できる。ライセンスが有効な間は自分でチケットを作成して対応してもらうことも可能なはずだ。

```bash
% motion support
```

それでは良い RubyMotion ライフを。

---

##### 注釈

※1 Android の場合、ランタイムは Dalvik でも ART でもちゃんと動く。~~詳しくはよく分かってないのだが、Dalvik 環境下では Ruby のコードから DEX を、ART 環境下では LLVM が解釈できるコードをそれぞれ直接吐き出しているのだろうか？詳しい方教えてください。~~

（9/24 追記）

ART ランタイムについて勘違いしていたが、ART はインストール時に dex を端末内でコンパイルするものなので RubyMotion がどうこうとはあまり関係ないかも、と教えていただいた。ありがとうございます！

ちなみに [公式ブログ](http://blog.rubymotion.com/post/87048665656/rubymotion-3-0-sneak-peek-android-support "RubyMotion 3.0 Sneak Peek: Android Support") を読んでいたら、**RubyMotion for Android では Ruby のコードを LLVM ベースの静的コンパイラで直接 ARM のマシン語に変換し、ランタイムは JNI 越しにそれらのコードにアクセスして実行される**のだと解説してあった。素晴らしい。

（追記ここまで）

※2 JDK 7 だと署名時のコマンドライン引数が変わっていて普通に adb install しようとしても入らない端末(or version?) がある。[このへん](http://java.dzone.com/articles/android-solution-install-parse-1) 参照。

※3 良くわからない人は [このへん](http://techfun.cc/android/mac-android-sdk-install.html "Android環境構築(Mac版) Android SDK インストール") とか見ればいいかも知れません。

※4 良くわからない人は API 10〜19 ぐらいまで全部入れておけば良いです。時間とストレージの無駄ではありますが分からずに悩むよりはマシです。

※5 以下のコマンドを実行して何も表示されないと、開発端末として認識されていない。再度説明を読み直して USB デバッグが有効になっているか確認し、それでもダメならケーブルが「充電専用」とかでないことを確認してみよう。意外と良くあるミス。

```bash
% ./sdk/platform-tools/adb devices
List of devices attached
0301044a08e4ae97    device
```

※6 Read Eval Print Loop のこと。その名のごとく、書いたコードを対話的に逐次評価して表示してくれるツール。 個人的に REPL を提供しない言語はクソ言語認定している。
