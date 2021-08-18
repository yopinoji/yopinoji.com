---
title: "Homebrew を使わずに MacOS に Git をインストールする方法とその問題点"

category: "Tech"
lang: "ja"
date: "2020-01-02"
slug: "install-git-in-mac-without-homebrew"
draft: false
template: "posts"
tags:
  - Git
  - Mac
---

最近、年末年始ということで思い切って PC のクリーンインストールを行い、Git など開発に使うものを全てインストールしなおしました。

Git を Mac にインストールする際に一般的なのは、Mac 用のパッケージ管理ソフト Homebrew を使ってインストールする方法だと思います。  
Google で「Git Mac インストール」と検索してみましたが、Homebrew で Git をインストールする方法の紹介がほとんどでした。

私は自分の PC には不要なソフトは極力インストールしたくない派です。  
しばらくするとあまり触らなくなるソフトはできるだけインストールしたくない性分です。

そのため、Homebrew 抜きで純粋に Git だけを簡単にインストールする方法を考えたのですが、  
ただ、結果として Homebrew を使った方がいいかもしれないねということに気づいたので、その話です。

## Git が何を経由してインストールしたのか確認する方法

`which` コマンドで Path を確認することで何を用いて Git をインストールしたのかを確認できます。

Mac に元々用意されているものは、以下の Path にインストールされています。

```bash
which git
/usr/bin/git
```

Homebrew でインストールした場合、以下の Path にインストールされています。

```bash
which git
/usr/local/bin/git
```

## Homebrew 抜きで Mac に Git をインストールする方法

とりあえず、最初に Homebrew 抜きで Git を Mac にインストールする方法を紹介しておきます。  
ただ、この方法でインストールするのかは、最後まで読んでから決めていただければと思います。

とても簡単です。

Mac でターミナルを開いて、`git`と入力します。

![install-git-in-mac-without-homebrew-01](./install-git-in-mac-without-homebrew-01.png)

Git がインストールされていない場合、コマンドラインデベロッパーツールのインストールを促されます。

また、以下のコマンドからインストールすることも可能です。

```bash
xcode-select --install
```

他にも Xcode 経由でインストールすることも可能ですが、当方の PC に Xcode をインストールしていないため説明は省かせていただきます。

![install-git-in-mac-without-homebrew-02](./install-git-in-mac-without-homebrew-02.png)

コマンドラインデベロッパーツールのインストールが終わったら、すでに Git コマンドが使えるようになっています。

## 問題点

さて、Homebrew 抜きで Git を使える状態まで持っていくことはできたのですが、  
残念ながら、**このインストール方法には重大な問題点があります。**

それは、**Git のバージョンを簡単に変更できないことです。**

バージョン管理に Homebrew を使わないので、当たり前といえば当たり前ですが、  
Git それ自体に脆弱性があった場合など、バージョン変更が容易ではないと少し困ります。

実は、この記事を書いている最近にも Git に脆弱性がありました。  
https://forest.watch.impress.co.jp/docs/news/1223826.html

こういった際に Homebrew を使っていれば、以下のコマンド一行でアップデートできます。

```bash
brew upgrade
```

ただ、コマンドラインデベロッパーツール経由の場合、ツールの更新を待つ必要があります。

Homebrew を使っていない場合、今後のことも考えて Homebrew で Git をインストールしなおすのが賢明かなと思います。

## 結論

何か問題があった際にすぐにバージョンを切り替えられるのは Homebrew の強みですね。  
やっぱり Git のインストールは Homebrew 使った方がいいですね。
