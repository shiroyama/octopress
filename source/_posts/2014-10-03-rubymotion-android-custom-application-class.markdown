---
layout: post
title: "RubyMotion for Android でカスタム Appliation クラスを利用する"
date: 2014-10-03 21:16:29 +0900
comments: true
categories: Android RubyMotion
---

[RubyMotion](http://www.rubymotion.com/ "RubyMotion") のバージョンが 3.1 になり、Android 版でカスタム Application クラスが利用できるようになった。

実を言うとこれまでは独自定義の Application クラスを利用することが出来なかったのだが、その旨 RubyMotion のサポートで質問したところ、作者の Laurent さんが一瞬で対応してくれたのだ。  
**なんてCOOLなんだ！みんなも RubyMotion を買おう！**

では早速、独自 Application クラスの使い方を簡単に紹介しておく。RubyMotion for Android の入門に関しては前回書いた[拙エントリ](http://blog.shiroyama.us/blog/2014/09/20/rubymotion-for-android/ "RubyMotion for Android を使ってみる")を参考にして欲しい。

#### 1) RubyMotion を更新

以下のコマンドで更新。`--pre` を忘れないようにしよう

```bash
% sudo motion update --pre

Password:
Connecting to the server...
Downloading software update...
######################################################################## 100.0%
Installing software update...
RubyMotion pre-release update installed in /Library/RubyMotionPre

= RubyMotion 3.1 =

  * Added the RUBYMOTION_VERSION and RUBYMOTION_ENV constants, which have the
    same values as RubyMotion for iOS / OS X.
  * Added support for the Android R class. Thanks to Mark Villacampa for the
    patch.
  * Added the `app.application_class' setting, which can be used to specify
    the name of a custom Android::App::Application subclass that should be used
    by the application. By default, the variable has a nil value, which means
    a default class will be used.
  * Improved app versioning. `app.api_version' will now map to the
    `minSdkVersion' manifest attribute. Added `app.target_api_version' which
    will map to `targetSdkVersion'. Both settings will have the latest API
    version as the default value, except for 20 (L). We recommend that you
    leave `app.target_api_version' intact and only modify `app.api_version'.
  * Improved the REPL so that "self" now always points to the current app
    activity. (Only works for API 14 or above.)
  * Added support for `&foo' constructs (which should dispatch #to_proc).
  * Added Symbol#to_proc.
  * Fixed a bug where `app.vendor_project' would generate a .bridgesupport
    file containing illegal XML characters (such as '<' or '>') inside
    attributes. Thanks to Mark Villacampa for the patch.
  * Fixed a bug where the dispatch of an overloaded Java method without
    arguments would fail. The runtime will now return the first method of the
    list (since we can't match any argument).
  * Fixed the `rake {emulator,device}' tasks to terminate the application in
    case the user leaves the REPL session.
  * Fixed a bug when dispatching certain methods using reflection (REPL) that
    return java.lang.Object.
  * Fixed a bug in the compiler where a crash would happen when trying to
    override a Java method accepting a Java array as argument.
  * Fixed a bug in the dispatcher when yielding certain runtime-generated
    blocks (ex. enumerators) where the return value would be destroyed.
```

#### 2) カスタム Application クラスを準備

Android 用のテンプレートを生成し、

```bash
% motion create --template=android Hello
% cd ./Hello
```

Android::App::Application クラスを継承した独自 Application クラスを定義する。

試しに onCreate(), onTerminate() でログを出すようにしてみた。  
※ onTerminate() は[実機では呼ばれない](http://developer.android.com/reference/android/app/Application.html#onTerminate\(\))ので注意。理解の便宜上書いてみた。

```ruby
% vim app/my_application.rb

class MyApplication < Android::App::Application
  def onCreate()
    super
    puts "Application#onCreate()"
  end

  # will never be called on a production Android device
  def onTerminate()
    super
    puts "Application#onTTerminate()"
  end
end
```

#### 3) Rakefile を修正

`app.application_class` に独自定義の Application クラスを指定

```ruby
% vim Rakefile

# -*- coding: utf-8 -*-
$:.unshift("/Library/RubyMotionPre/lib")
require 'motion/project/template/android'

Motion::Project::App.setup do |app|
  # Use `rake config' to see complete project settings.
  app.name = 'Hello'
  app.application_class = 'MyApplication'
end
```

最初どんなプロパティか分からなくて戸惑ったけど `/Library/RubyMotionPre/lib/motion/project/template/android.rb` みたらすぐ分かった。

#### 4) 確認

確認してみよう！

```bash
% rake device

    Create ./build/Development-19/classes.dex
    Create ./build/Development-19/Hello.apk
      Sign ./build/Development-19/Hello.apk
     Align ./build/Development-19/Hello.apk
   Install ./build/Development-19/Hello.apk
4066 KB/s (638813 bytes in 0.153s)
     Start com.yourcompany.hello/.MainActivity
--------- beginning of /dev/log/main
--------- beginning of /dev/log/system

I/com/yourcompany/hello(11394): Application#onCreate()
```

一番下の行に注目！Application の開始時に一度だけ出ている！

前述のとおり onTerminate() は実機では呼ばれないが仕様なので気にしないでください。

### まとめ

独自 Application クラスを利用できるようになったことで実戦的なアプリ開発の可能性がグッと広がったように思う。  
ライブラリの初期化とか、静的 Context 取得メソッドとか、アプリケーションライフサイクルで管理する Observer とか、実際の Android 開発では利用シーンの枚挙にいとまがない。

いよいよ次回は少し実戦的なアプリを実装して GitHub に上げてその記事を書こうと思う。それでは楽しい RubyMotion ライフを！
