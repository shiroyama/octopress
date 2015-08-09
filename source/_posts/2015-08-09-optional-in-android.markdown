---
layout: post
title: "Android で Optional<T> 使おう"
date: 2015-08-09 22:09:12 +0900
comments: true
categories: Android Optional RxJava RxAndroid
---

[Shibuya.apk #2](http://shibuya-apk.connpass.com/event/16640/ "Shibuya.apk #2") というイベントで登壇させていただきました。資料は[ここ](http://www.slideshare.net/fumihikoshiroyama/rxjavaoptionalandroid "Rxjavaとoptionalで関数型androidしよう")にあります。  
テーマは「RxjavaとOptionalで関数型Androidしよう」ですが昨今の巷での関数型言語の流行りに便乗した釣りタイトルです。誠に申し訳ございません。  
で、そのテーマの前半部分である `Optional<T>` についてここで詳しくまとめておきたいと思います。

### Optionalとは何か

Optional とは **あるかもしれないしないかもしれない** ものを表現する型です。  
Haskell でいう `Maybe a  =  Nothing | Just a` です。Scalaでいう `Option[T] = Some(T) | None` です。  
`Optional<String>` ってのがあると、文字列があるかもしれないしないかもしれないって意味になります。

これの何が嬉しいかっつーと `if (str != null && !TextUtils.isEmpty(str))` みたいなつまらないボイラープレートを書かなくて済むんですよ。  
あと、これが一番大事なので最初に書くけど、 **Optionalの中身があるかどうか気にせずOptionalオブジェクトをmapで写したりできる** んです。これは死ぬほど大事なので後でコード例とともに示します。  
あとはまあ、 `Optional<T>` って書いとくと **ああこれは中身がnullになりうるんだな** ってことが仕様としてハッキリします。これはドキュメントに書くより、`@Nullable` 修飾するより強力です。

### AndroidはJava8使えない

Java8からは `Optional<T>` が使えます。Java8ではラムダ式だとか関数クラス型だとかStream APIとかが追加されてますが、個人的にはOptionalが最も大切です。  
で、御存知の通りAndroidは未だにJava6互換でいくつかJava7のAPIが使えるにとどまっており、非常にレガシーな書き方を余儀なくされます。とても悲しい。

ところが、私の敬愛するsys1yagiさんという方の [RxJavaのObservable<T>でOptional<T>を代行する](http://sys1yagi.hatenablog.com/entry/2015/01/26/183000 "RxJavaのObservable<T>でOptional<T>を代行する") という神のようなブログエントリでAndroidでもOptionalが使えることが示されました。

当該エントリをそのままコピペしてもほとんど申し分ないOptionalが扱えるのですが、一部RxJavaのObservableがむき出しになっており、RxJava内でOptionalを取り回したいときにちょっと見た目がわかりづらくなるので完全にOptionalという名前でラップしたライブラリを自作しようとしたところ、なんと既にありました。[eccyan/RxJava-Optional](https://github.com/eccyan/RxJava-Optional "eccyan/RxJava-Optional") です。作者さま本当にありがとうございます。

### AndroidでOptional早速使おう

gradleで入れられます。

```
compile 'io.reactivex:rxandroid:0.25.0'
compile 'com.eccyan:rxjava-optional:1.1.2'
```

バージョンに注意してください。公式のREADMEだと`1.1.0`になってるのですが、このバージョンだとAPIレベル19以降でしか使えない`Objects.requireNonNull`がそのまま使われており、古いAndroidで動きません。  
`1.1.2`では自作のObjectsクラスがバンドルされており古いAndroidでもちゃんと動きます。

インストールできたら早速使いまくりましょう。

なお、以下の例ではラムダ式が出てきまくりますが、これは`Retrolambda`というライブラリとAndroid Studioのプラグインを組み合わせて使ってます。  
インストール方法がちょっとややこしいので別エントリにまとめ次第追記します。

#### Optionalでくるむ

まずOptionalを使うポイントですが、前述のとおりあるかもしれないしないかもしれないところで使います。  
具体的に言うと、関数の最後で `Map#get` だとか `String#indexOf` の結果を戻してるような箇所はOptionalで包む格好の場所だと思います。

値をOptinalで包むメソッドは3つ用意されてます。

* `Optional.of(T)`
* `Optional.ofNullable(T)`
* `Optional.empty()`

`Optional.of(obj)` はobjがnullだった場合にぬるぽを投げます。対して、`Optional.ofNullable(obj)` はnullかも知れないobjも安心して包めます。  
だからと言って **ofNullabeを毎回使いましょう** みたいなルールにしちゃダメですよ。  
nullじゃないことが明らかな場合、またはnullであっちゃならない場所では`of`を使うべきです。なぜなら本来nullであってはならない場所ならその場でぬるぽで落ちるべきであるからです。

nullでないのが明らかなら`T`でいいじゃんという声もありそうですが、戻り値としては`Optional<T>`を返したいというのとその場でobjがnullでないことが明らかというのはまた別の話です。

話がそれましたが、`Optional.empty()` は `Optional.ofNullable(null)` と同じことです。

#### Optionalから値を取り出す

Optionalから値を取り出します。ホントは`map`とかが一番重要なんですが、後に譲ります。

* `Optional#get()`
* `Optional#orElse(T)`
* `Optional#orElseCall(() -> {T})`
* `Optional#orElseThrow(() -> {Throwable})`

`Optional#get()` は中身がnullのときは`NoSuchElementException`が投げられます。これはぬるぽを踏むのと一緒であんま意味がないので僕は使ったことないです。

`Optional#orElse(T)` は中身がnullのときに引数で与えた値が取り出せます。`map`と組み合わせて死ぬほど使うので覚えておいてください。

`Optional#orElseCall(() -> {T})` は`orElse`に似てますが、中身がnullのときに初めてラムダ式が評価されてその結果が取り出せます。引数が遅延評価されるのでより関数型っぽいですね。初期化処理がコスト高い場合なんかはこっちを使うと良いかもしれません。

`Optional#orElseThrow(() -> {Throwable})` は中身がnullのときはラムダ式で作られた例外が投げられます。  
Optionalを何回も`map`や`filter`して最終的に中身がなかったら例外を上げるっていうケース、時々あるような気がするので何回か使った記憶がありますが、それでも`orElse`を使う頻度よりはるかに少ないはずです。  
もし`orElseThrow`をどこででも使ってるとしたらそれはなんかOptionalを誤解してる気がします。普通にTを返すメソッドを`throws HogeException`してください。

#### Optionalをモナドとして使うよ

冒頭で **Optionalは中身があるかどうか気にせずOptionalのままmapで写したりできる** って書きました。これめっちゃ大事なことなんですよ。

たとえば、`Optional#isPresent()` っていう、中身の値があるかどうか調べるメソッドがあるんですが、ちょっと下のコード見てください。

```java
if (opt.isPresent()) {
   String val = opt.get();
}
```

これ何の意味があるんでしょうか。これ今までの退屈なnullチェックと同じですよね。Optional使った意味全くないですよね。なのでこれは最悪です。こんなの書かないようにしてください。

Optionalは下みたいなクールな方法で扱います。

* `Optional#ifPresent(値を消費するラムダ式)`
* `Optional#map(TをUに写すラムダ式)`
* `Optional#flatMap(TをOptional<U>に写すラムダ式)`
* `Optional#filter(Tを新しいTに写すときの条件を示すラムダ式)`

##### Optional#ifPresent

`ifPresent`はOptionalの中身があったときだけ引数のラムダ式を実行してくれます。

```java
Optional<String> strOpt = Optional.of("V8!!!");
strOpt.ifPresent(s -> Log.i(TAG, s));
```

これはもう見ての通りです。注意点は、`ifPresent`は **値を返す式ではない** ということです。そういう場合は後述の`map`を使います。

##### Optional#map

`map`は元のOptionalに包まれた`<T>`の値を、新しい`<U>`に写します。要するに変換です。

```java
Optional<String> strOpt = Optional.of("123");
strOpt.map(s -> Integer.valueOf(s)) // strOpt.map(Integer::valueOf) とも書けるよ！
      .ifPresent(i -> Log.i(TAG, "i: " + i));
```

上記は分かりやすさのためにほとんど意味のないコードですが、何かエンティティ（例えばユーザ情報）を`map`してユーザ名だけの圏を得たりするのは非常によくする操作なんじゃないでしょうか。

`map`に`orElse`をつなぐと例外処理など使わずとも上から下までOptionalとその写像だけを連ねた結果をエレガントに扱うことができますよ。

```java
int result = Optional.ofNullable(strNullable)
                     .map(Integer::valueOf)
                     .orElse(123);
```

この例はしょうもなさすぎてエレガントとは言い難いですがね！けどアイディアは伝わるでしょう。`map`はいくら連ねてもいいんですよ。

##### Optional#flatMap

`flatMap`はOptionalで包まれた値を次に渡すときに使います。ちょっと分かりづらいという人も居ますが、簡単ですよ。  
`map`が`T`から`U`を写すのに使う、つまり`map`の返り値は値型そのものなんですが、既に`Optional<U>`を返す関数とかがあるときにそれをそのまま使っちゃうと`Optional<Optional<U>>`が写されちゃうんで、それが適切でないときに`flatMap`でぺしゃんこにします。flatten（=平らにする）して（map=写す）だけです。

```java
Optional<Token> tokenOpt = getToken();
tokenOpt.flatMap(token -> someApiCall(token, args))
        .ifPresent(apiResult -> processApiResult(apapiResult)); // this::processApiResult とも書けry
```

こんな感じ。  
この例でいうと`someApiCall`は`Optional<ApiResult>`を返すので、そのままだと`ifPresent`に`Optional<Optional<ApiResult>>`が渡ってしまうので`flatMap`でぺったんこにしています。ぺったんこの意図するところ、伝わりますかね。

##### Optional#filter

最後に`filter`です。これは例を見てもらうと簡単です。

```java
Optional.of(123)
        .filter(i -> i > 100)
        .orElse(100);
```

`filter`に渡すラムダ式は値を受け取って真偽値を返すようなものを渡します。真になったものだけ生き残るというわけです。

#### まとめ

この通り、Java8のOptionalとほぼ遜色のないコードを書けることが分かると思います。正しく使う限りデメリットが見当たらないのでぜひ使うことをおすすめしたいです。

本エントリでは触れませんが、この代用版OptionalとRxJava/RxAndroidは組み合わせて使うとめっちゃ強力です。（元々同じものなんですけどね）  
機会があったらその辺も書こうかなと思います。では。

#### 参考リンク
* [RxJavaのObservable<T>でOptional<T>を代行する](http://sys1yagi.hatenablog.com/entry/2015/01/26/183000 "RxJavaのObservable<T>でOptional<T>を代行する")
* [Java Optionalメモ(Hishidama's Java8 Optional Memo) ](http://www.ne.jp/asahi/hishidama/home/tech/java/optional.html "Java Optionalメモ(Hishidama's Java8 Optional Memo) ")
* [Optionalの取り扱いかた](http://irof.hateblo.jp/entry/2015/05/05/071450 "Optionalの取り扱いかた")
