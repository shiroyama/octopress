---
layout: post
title: "'エンジニアのブログは Octopress が最適'"
date: 2014-02-26 23:54:23 +0900
comments: true
categories: Octopress
---

### ブログを Octopress に移行した

既存のブログを [Octopress](http://octopress.org/ "Octopress") に移行した。  
理由は以下の通りである。

* 無料
* Markdown で書ける
* 独自ドメインが利用できる
* Vim で書いて `rake deploy` するとブログが公開されるのが COOL

Markdown は表現力に乏しいので大して好きでもないのだが、何かをメモする程度なら非常に簡便で技術ブログには向いている。  
また、ブログを書こうと思い立った瞬間に普段作業しているターミナルから Vim でブログを書いて `rake deploy` すると GitHub 上に push されてブログが公開されるというのは UX として非常に良い。

折角なので Octopress 導入メモを残すことにする。これから使ってみたい方の参考になれば幸い。

### Octopress とは

ブログ記事を簡単に生成するためのフレームワーク。  
ブログ記事そのものは [GitHub Pages](http://pages.github.com/ "GitHub Pages") 等にホスティングする。

### GitHub Pages とは

GitHub 上で静的なページを簡単に生成・管理する仕組み。  
GitHub に `your_name.github.io` というリポジトリを作成して push するだけで `http://your_name.github.io` にアクセスすると静的コンテンツにアクセスすることができる。  
ここには HTML や CSS, JavaScript 等を普通に配置することが出来るので、特にブログにこだわらない場合は GitHub Pages だけで立派な静的なページを作成することができる。

リポジトリのルートに CNAME というファイルを設置し、中に独自ドメインを記入することで独自ドメインも利用可能。

```bash
% cat<<'EOS'>CNAME
blog.your-domain.com
EOS
```

もちろん、DNS サーバの CNAME レコードに your_name.github.io. を設定する必要がある。

```bash
# 擬似コード

blog.your-domain.com    IN    CNAME    your_name.github.io.
```

CNAME レコードは （your-domain.com のような）Apex ドメインには指定できないので、その場合は [Route 53](http://aws.amazon.com/jp/route53/faqs/ "Amazon Route 53 に関するよくある質問") のような A レコードの Alias レコードを設定できるような DNS の利用を検討するか、さもなくば [GitHub の DNS を IP 直指定](https://help.github.com/articles/setting-up-a-custom-domain-with-pages "Setting up a custom domain with Pages") する必要がある。  
この方法は GitHub の CDN の仕組みを利用できないので明確に非推奨とされている。素直に `subdomain.your-domain.com` の方法を採るほうが良い。

### 5分で出来る初めての Octopress

octopress を GitHub からクローンしてくる。

```bash
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```

必要な gem をインストールする。

```bash
bundle --path vendor/bundle
```

デフォルトのブログテーマをインストールする。

```bash
bundle exec rake install
```

プレビューしてみる。

```bash
bundle exec rake preview
```

`http://localhost:4000/` にアクセスすると、ブログのプレビューを見ることが出来る。

{% img /images/2014-02-26/octopress_preview.png 'octopress preview' 'octopress preview' %}

### Octopress の初期設定

Octopress のプレビュー確認が出来たので初期設定を施していく。

`bundle exec rake setup_github_pages` タスクを実行し GitHub Pages にデプロイするファイルを生成する。  
この時、`your_name.github.io` のように GitHub Pages のリポジトリを git か https プロトコルで指定してやる。

```bash

% bundle exec rake setup_github_pages

Enter the read/write url for your repository
(For example, 'git@github.com:your_username/your_username.github.io.git)
           or 'https://github.com/your_username/your_username.github.io')
Repository url: git@github.com:srym/srym.github.io.git
Added remote git@github.com:srym/srym.github.io.git as origin
Set origin as default remote
Master branch renamed to 'source' for committing your blog source files
rm -rf _deploy
mkdir _deploy
cd _deploy
Initialized empty Git repository in /Users/shiroyama/playGround/octopress/_deploy/.git/
[master (root-commit) dacfaa2] Octopress init
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
cd -

---
## Now you can deploy to git@github.com:srym/srym.github.io.git with `rake deploy` ##
```

このタスクを実行すると、以下のように GitHub Pages のリポジトリが自動的に origin に指定される。

```bash
% git remote -v
octopress       git://github.com/imathis/octopress.git (fetch)
octopress       git://github.com/imathis/octopress.git (push)
origin          git@github.com:srym/srym.github.io.git (fetch)
origin          git@github.com:srym/srym.github.io.git (push)
```

次に _config.yml を編集し、ブログのタイトルなどを入力する。

```yaml
# _config.yml

url: http://blog.shiroyama.us
title: 白山軟件有限公司
subtitle: 東洋太平洋ブログ三日坊主チャンピオンのブログ
author: Fumihiko Shiroyama
simple_search: http://google.com/search
description:
```

以上で最低限の初期設定は完了となる。  
twitter 連携などを設定する箇所もあるが、これについては後述する。

### 記事を書く

Octopress の初期設定ができたので、いよいよブログ記事を作成する。  
以下の Rake タスクで記事のひな形が生成される。

```bash
# shell によっては "new_post[title]" とする必要があるかも知れない
% bundle exec rake new_post[title]

mkdir -p source/_posts
Creating new post: source/_posts/2014-02-26-title.markdown
```

生成された `source/_posts/2014-02-26-title.markdown` を Vim などのエディタで開いて編集する。  
ファイル上部にはブログのメタ情報が記述されているので適宜編集する。

```yaml
---
layout: post
title: "'エンジニアのブログは Octopress が最適'"
date: 2014-02-26 23:54:23 +0900
comments: true
categories:-
---
```

ブログのカテゴリは以下のように色々な方法で指定することが出来る。

```
# One category
categories: Sass

# Multiple categories example 1
categories: [CSS3, Sass, Media Queries]

# Multiple categories example 2
categories:
- CSS3
- Sass
- Media Queries
```

記事はメタ情報の `---` の下から Markdown で好きなだけ書くことが出来る。  
Markdown の文法についてはここでは触れない。

### 画像を載せる

Octopress で唯一面倒なのが画像を載せる方法だが、出来ないことはないのでメモしておく。

```
# basic image
{% img [class names] /path/to/image [width] [height] [title text [alt text]] %}

# examples
{% img http://placekitten.com/890/280 %}
{% img left http://placekitten.com/320/250 Place Kitten #2 %}
{% img right http://placekitten.com/300/500 150 250 Place Kitten #3 %}
{% img right http://placekitten.com/300/500 150 250 'Place Kitten #4' 'An image of a very cute kitten' %}
```

画像ファイルは `source/images` の下に好きなだけ置くことが出来る。  
日付ごとにディレクトリを掘って画像を配置し、

```
{% img /images/2014-02-26/octopress_preview.png 'octopress preview' 'octopress preview' %}
```

のようにすると問題なく画像を表示できたが、はっきり言って面倒だったので個人的には今後は Gyazo 等を使うと思う。

### GitHub Pages にデプロイする

いよいよブログをデプロイする。Rake タスク一発である。

```bash
bundle exec rake deploy
```

もしデプロイに失敗したらエラーログを注意深く眺める必要があるが、もしかしたら `bundle exec rake setup_github_pages` したときに既に your_name.github.io が存在していて fast-forward ではないせいで git push に失敗しているのかもしれない。  
その場合は Rakefile を開き、`git push -f origin` にして force push してやれば良い。

git push -f の意味が分からない人は無闇に実行しないこと。当ブログでは責任を一切持てない。


```ruby
262   cd "#{deploy_dir}" do
263     system "git add -A"
264     puts "\n## Committing: Site updated at #{Time.now.utc}"
265     message = "Site updated at #{Time.now.utc}"
266     system "git commit -m \"#{message}\""
267     puts "\n## Pushing generated #{deploy_dir} website"
268     #system "git push origin #{deploy_branch}"
269     system "git push -f origin #{deploy_branch}"
270     puts "\n## Github Pages deploy complete"
271   end
```

### Octopress で独自ドメインを利用する

`source/CNAME` に独自ドメインを記述するだけである。  
DNS の設定等は GitHub Pages で独自ドメインを使う時と同様である。

```bash
echo 'blog.shiroyama.us' >> source/CNAME
```

### テーマを変更する

Octopress で利用できるカスタムテーマは数多くある。  
[http://opthemes.com/](http://opthemes.com/ "http://opthemes.com/") などを参照するといいだろう。  
テーマのインストール方法は各テーマのページに書いてあるが、どれも `rake install` するだけといった風に簡単である。

### 外部サービスと連携する

初期設定に使った `_config.yml` の後半に Twitter, LinkedIn, Google+ 等との連携の設定箇所がある。  
どれも以下のようにアカウントを書くだけといった具合の簡単さなので試してみて欲しい。

```
# Twitter
twitter_user: fushiroyama
twitter_tweet_button: true

# Google +1
google_plus_one: true
google_plus_one_size: medium
```

### octopress リモートリポジトリも自分で管理したい

octopress を clone してきて普通にセットアップすると、以下のように octopress リモートリポジトリは開発元の imathis リポジトリを指すことになると思う。

```bash
% git remote -v
octopress       git://github.com/imathis/octopress.git (fetch)
octopress       git://github.com/imathis/octopress.git (push)
```

個人的には、原稿の元になる octopress リポジトリも自分のリポジトリに管理してどこででもブログを書きたい。  
その場合には以下のようにすれば出来た。

1. octopress のリポジトリから自分のリポジトリに **fork** する。
1. fork した自分のリポジトリを clone する。
1. 以下、全く同じように `bundle exec rake setup_github_pages` する。

```bash
% git remote -v
octopress       git@github.com:srym/octopress.git (fetch)
octopress       git@github.com:srym/octopress.git (push)

% git push octopress source
```

### その他

思いついたら追記する。

