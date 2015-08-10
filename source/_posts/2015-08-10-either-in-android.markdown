---
layout: post
title: "Android で Either<L,R> を使おう"
date: 2015-08-10 21:45:49 +0900
comments: true
categories: Android Either Optional RxJava RxAndroid
---

先日 [Android で Optional を使おう](http://blog.shiroyama.us/blog/2015/08/09/optional-in-android/ "Android で Optional を使おう") というエントリを書きました。

`Optional<T>`ってのは **値があるかないか** を表現するって言いましたけど、あるメソッドが`Optional`で包んだ値`T`を返して来るってことは、言い換えると`Optional`は **ある処理が成功したかまたは失敗した** ことを表していると言えます。

しかしこれだと失敗したことは分かるけどその理由までは分からないですね。こういうときHaskellでは`Either`というデータ型を利用します。

```haskell
Prelude> :i Either
data Either a b = Left a | Right b
```

`Either`は **失敗理由と成功データ** の両方を表現するデータ型です。  
`Left`が失敗、`Right`が成功に対応します。rightという単語の "右" と "正しい" の両方の意味が掛かった洒落ですね。これ、Androidにもあると便利なんですよ。

Javaでは伝統的に例外ケースはその名の通り "例外" で表現してきましたが、これは本質的に副作用です。それに微妙にRxJavaと相性が悪いです。

RxJavaでは複数の`Observable`をチェインさせてプロミス的な使い方をすることがままありますが、この間に起きた例外は`Subscriber`の`onError`に来ます。  
しかし **例外が発生したわけではないけど成功じゃないパターン** って時々ありますよね。  
んー例えば何でも良いんだけど、

1. HTTP 200だけど`{'availability':false}`みたいなJSONでエラーを伝えてくるAPI
1. 必要なパラメータが足りてない、形式が不正
1. 2みたいなケースを`filter`オペレータで間引くんじゃなくてエラーは伝播させたい

などなど。少なくとも自分はこういうときの銀の弾丸をまだ見つけられていません。

こういう時独自の例外を`throw`して`onError`で捕まえるのはあんまりしっくりきてません。  
例外を単純な場合分けに使うのはいかにも筋が良くないし、そもそも`Observable`は`onComplete`か`onError`でそのライフサイクルを終えるので、復旧困難なケース以外でここに放り込むのは何か違う気がしてます。

これは`Either`を使うしかないでしょう！

#### Either を作る

Javaには`Either`なんてイカしたものは勿論ないので作るしかありませんが、「Java Either」とかでググると [Is there an equivalent of Scala's Either in Java 8?](http://stackoverflow.com/questions/26162407/is-there-an-equivalent-of-scalas-either-in-java-8 "Is there an equivalent of Scala's Either in Java 8?") なんてのがすぐに見つかります。

この例はJava8でしか動かせないのでAndroid用に書き換えたのが以下です。  
一部うまく推論してくれないところがありましたが使わなそうなので無理に移植せず削除しました。

```java
package functional.data;

import com.eccyan.optional.Optional;

import rx.functions.Action1;
import rx.functions.Func1;

/**
 * borrowed from http://stackoverflow.com/questions/26162407/is-there-an-equivalent-of-scalas-either-in-java-8
 */
public final class Either<L, R> {
    public static <L, R> Either<L, R> left(L value) {
        return new Either<>(Optional.of(value), Optional.empty());
    }

    public static <L, R> Either<L, R> right(R value) {
        return new Either<>(Optional.empty(), Optional.of(value));
    }

    private final Optional<L> left;
    private final Optional<R> right;

    private Either(Optional<L> l, Optional<R> r) {
        left = l;
        right = r;
    }

    public <T> Either<T, R> mapLeft(Func1<? super L, ? extends T> lFunc) {
        return new Either<>(left.map(lFunc), right);
    }

    public <T> Either<L, T> mapRight(Func1<? super R, ? extends T> rFunc) {
        return new Either<>(left, right.map(rFunc));
    }

    public void apply(Action1<? super L> lFunc, Action1<? super R> rFunc) {
        left.ifPresent(lFunc);
        right.ifPresent(rFunc);
    }
}
```

早速自分のアプリに組み込んでみました。使い方はこんな感じです。  
HaskellやScalaだと`Left`と`Right`でパターンマッチできるのですが、Javaだとそれも無理なので `Either#apply(leftラムダ式, rightラムダ式)` みたいな感じでお茶を濁していますね。

```java
@Bind(R.id.weatherInput)
EditText editTextCity;

@OnClick(R.id.submit)
void onClickSubmit(Button button) {
    WeatherApiCreator.create(CurrentWeatherService.class).getByCityName(editTextCity.getText().toString())
            .map(currentWeatherResponse -> {
                Either<Throwable, String> either;
                if (currentWeatherResponse.getCode() != 200) {
                    either = Either.left(new RuntimeException("error code: " + currentWeatherResponse.getCode()));
                } else {
                    either = Either.right("Humidity: " + currentWeatherResponse.getMain().getHumidity());
                }
                return either;
            })
            .subscribeOn(Schedulers.newThread())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                    either -> either.apply(
                            left -> Toast.makeText(this, left.getMessage(), Toast.LENGTH_SHORT).show(),
                            right -> Toast.makeText(this, right, Toast.LENGTH_SHORT).show()
                    ),
                    error -> Toast.makeText(this, error.getMessage(), Toast.LENGTH_SHORT).show()
            );
}
```

もうひとつ大事なことがあります。 **Eitherは写像を作る※** ことができます。  
以下の例をみてください。

```java
.map(currentWeatherResponse -> {
    Either<Throwable, String> either;
    if (currentWeatherResponse.getCode() != 200) {
        either = Either.left(new RuntimeException("error code: " + currentWeatherResponse.getCode()));
    } else {
        either = Either.right("Humidity: " + currentWeatherResponse.getMain().getHumidity());
    }
    return either;
})
.map(either -> either.mapRight(String::toUpperCase))
.subscribeOn(Schedulers.newThread())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(
        either -> either.apply(
                left -> Toast.makeText(this, left.getMessage(), Toast.LENGTH_SHORT).show(),
                right -> Toast.makeText(this, right, Toast.LENGTH_SHORT).show()
        ),
        error -> Toast.makeText(this, error.getMessage(), Toast.LENGTH_SHORT).show()
```

この例だと、処理が成功していれば結果をすべて大文字に変換しています。例があまりにもしょうもなくて申し訳ないです！

重要なのは、途中で処理が成功していようが失敗していようが気にせず `mapRight` でRightを写すことができている点です。  
処理が途中で失敗しても、Rightは空っぽなので変換処理は単に空振りして、LeftはLeftのまま伝播するという寸法です。

言いたいこと伝わりますかね？ **利用者が isRight とか isLeft とか判定してるようじゃ意味が無い** のです。

#### Either 右翼？

以下、余談です。

上で僕は **Eitherは写像を作ることができる** と言いましたが、この例だとそれが嘘であることが識者にはバレバレだと思います。端的に言うと上のような`Either`は **モナドじゃありません。**  
Either自体はmapで写すことができておらず `mapRight`, `mapLeft` なんていう方法でそれぞれを操作しています。

こういうのは、LeftとRightを対等に扱ったEitherとか言われるみたいです。Scalaの標準ライブラリのEitherや上のJavaのコードなんかがその例です。

対して、HaskellやScalazのEitherは **Right-Biased Eitehr** とか呼ばれています。右寄りの、右派のっていう意味です。EitherをそもそもRightのコンテナとして利用しようという考え方です。  
このようなEitherはRight値をひとつ包むモナドのように動作します。また、LeftとRightを結合するといかなる場合もLeftになります。

Androidで利用できるRight-BiasedなEitherは夏休みの自由研究にでもしようかと思います。それでは。

##### 参考リンク
* [EitherとValidation](http://slides.pab-tech.net/either-and-validation/#1 "EitherとValidation")
* [Scalazを使おう #1](http://tech.recruit-mp.co.jp/server-side/post-2540/ "Scalazを使おう #1")
