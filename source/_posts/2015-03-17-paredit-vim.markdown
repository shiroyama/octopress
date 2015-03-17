---
layout: post
title: "Lisper必携 paredit.vim"
date: 2015-03-17 23:52:11 +0900
comments: true
categories: Vim Lisp Scheme SICP
---

[paredit.vim](https://github.com/vim-scripts/paredit.vim "paredit.vim") という Vim プラグインを今日はじめて知ったが凄すぎる！  
あまりに人生を変えられる出会いだったのでブログに書かざるを得なかった。

#### paredit.vim is 何

Vimmer が Lisp のS式を書くのが著しく快適になるプラグイン

```scheme
(define (sum a b c) (+ a b c))
```

例えばこの関数の仮引数 b のところで D して行末までバッサリ削除しようとすると…

```scheme
(define (sum a))
```

なんと括弧の対応を残した上で削除してくれる。奇跡か！

他にも、

* 改行するといい感じにインデントしてくれる
* 括弧を書くと閉じ括弧を自動挿入してくれる

などS式を書きやすいようになっている。

まだこれだけしか調べてないが、これだけで200倍ぐらい書きやすくなった！

#### 使い方

```vim
NeoBundle 'vim-scripts/paredit.vim'

:NeoBundleInstall

:help paredit

" 必要に応じて
:set filetype=scheme
```

Scheme と Clojure は対応していることを確認済み！

以上、SICP 勉強会のお供にどうぞ。
