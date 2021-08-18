---
title: "Git 操作を改めてまとめたカンニングペーパー的な記事"

category: "Tech"
lang: "ja"
date: "2020-01-29"
slug: "how-to-manipulate-git"
draft: false
template: "posts"
tags:
  - Git
---

表題の通り。  
Git 操作をたまに忘れるので、あらためてまとめておいて自分用のカンニングペーパーとして使うための記事です。

## 設定する系

### 設定の確認

```bash
git config --global -l
git config --local -l
```

`local` がローカルの Git リポジトリのコンフィグの確認、`global` は PC 全体でのコンフィグの確認です。

### 設定の追加

```bash
git config --global user.email "your@email.address"
```

ちなみに、GitHub に登録してあるメールアドレスの設定をしておかないと、GitHub の Contributions は増えないので注意してください。

## 確認する系

### リポジトリの確認

```bash
git remote -v
```

### 全ブランチの確認

```bash
git branch -a
```

頭が `remotes` から始まるものがリモートブランチで、それ以外がローカルブランチだと考えておけば OK です。

## ブランチ作成

### リモートブランチからローカルブランチを作成

```bash
git checkout -b local_branch_name remotes/origin/remote_branch_name
```

ちなみに `origin` はクローン元のリポジトリのことです。

### 特定のコミットからブランチを作成

`git log` でコミットハッシュを特定してブランチ作成するときは、以下のコマンドです。

```bash
git checkout 9aaa6bbbb9a6b9 -b feature/some-function
```

上記を活用することで、特定のコミットまで遡った状態でローカルに環境を作成することができます。

## コミット

### 何も変更を加えずにコミットする

```bash
git commit --allow-empty -m "empty commit"
```

GitHub Actions や Jenkins など CI 環境を作成していると、テスト目的で Git にコミットしたくなることがあると思います。

そういう際には何も変更を加えずにコミットできます。

## 変更する系

### 直前のコミットメッセージを変更する

割とよくあるコミットメッセージのタイプミス。

```bash
git add .
git commit -m "Refactoringggg componenttttt"
```

上記のようにコミットメッセージをタイプミスすることもあると思います。

```bash
git commit --amend -m "Refactoring component"
```

上記のコマンドでメッセージを修正することができます。

## 取り消す系

### ローカル変更の取り消し

いったん消したら戻ってこないので気をつけてね！

```bash
git checkout .
git clean -df .
```

### git add の取り消し

#### 個別に取り消す場合

```bash
git rm -r --cache target_folder
```

#### 全部取り消す場合

```bash
git reset HEAD .
```

## 削除する系

GitHub でブランチ自動削除を設定することもできますので、設定しておいたほうがいいでしょう。

今回は主にローカル環境で溜まっていく作業ブランチの整理方法について。

### ローカル環境においてマージ済みブランチの全削除（master/develop は除く）

```bash
git branch --merged|egrep -v '\*|develop|master'|xargs git branch -d
```

### 単体でのブランチ削除

```bash
git branch -D your_local_branch
```

## マージする系

### 作業ブランチへのマージ

```bash
git fetch
git merge origin/develop
```

作業ブランチに対して、`develop` ブランチをマージする例。

## fork したリポジトリで本家に追従する

OSS に貢献する際など、フォークしたリポジトリを扱う場合。

```bash
git remote add root_branch https://github.com/(Fork元のユーザ名)/(フォークしたいリポジトリ.git)
git fetch root_branch
git merge root_branch/master
git push origin master
```
