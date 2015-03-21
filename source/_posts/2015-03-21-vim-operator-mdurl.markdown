---
layout: post
title: "VimでMarkdownのリンク書くときに便利なﾌﾟﾗｷﾞﾝ"
date: 2015-03-21 23:06:12 +0900
comments: true
categories: Vim Markdown
---

このブログは [Octopress](http://octopress.org/ "Octopress") 使ってるのでMarkdownで書いてるんだけどMarkdownのリンク書くのドイメンじゃないですか。そういうひとは [vimtaku/vim-operator-mdurl](https://github.com/vimtaku/vim-operator-mdurl "vimtaku/vim-operator-mdurl") 使うといいですよ。

とりあえず `NeoBundle 'vimtaku/vim-operator-mdurl'` して入れてください。

#### 使い方

初期状態で `L` と `M` がマップされてます。バッティングしてる人はうまいこと変えてください。

##### 1) Lの方

`http://mana.bo/` みたいなテキストがあったらここで `LiW` とかすると `[http://mana.bo/](http://mana.bo/)` になります。`mattn/vim-textobj-url` とか入れてる人は `Liu` の方が自然ですね。

##### 2) Mの方

`http://mana.bo/` みたいなURLがヤンクされた状態で `株式会社マナボ` みたいなテキストの上で `MiW` とかすると `[株式会社マナボ](http://mana.bo/ "株式会社マナボ")` になります。最高クールじゃないすか。

Vim は世界で最も優れたテキストエディタなんでみなさん積極的に使いましょう。それでは。

参考リンク：[Vim-operator-mdurl という Vim Plugin 書いた](http://vimtaku.github.io/blog/2014/03/08/vim-operator-mdurl/ "Vim-operator-mdurl という Vim Plugin 書いた")
