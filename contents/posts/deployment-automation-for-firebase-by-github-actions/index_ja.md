---
title: "GitHub Actions で Firebase に Nuxt.js 製の静的サイトを自動デプロイしてみる"

category: "Tech"
lang: "ja"
date: "2020-05-27"
slug: "deployment-automation-for-firebase-by-github-actions"
draft: false
template: "posts"
tags:
  - GitHub Actions
  - GitHub
  - Firebase
  - Nuxt.js
---

インフラエンジニアの業務も兼任しているので、最近 GitHub Actions の yml ファイルばかり書いています。

というとこで、GitHub Actions の知見について少しばかり共有します。

## Nuxt.js 製の静的サイトを GitHub Actions で Firebase に自動デプロイする

さて今回は、Nuxt.js / Firebase の構成の Web アプリにおいて、Nuxt.js により生成された静的サイトを Firebase Hosting にデプロイするということを、GitHub Actions を用いて自動化してみます。

```bash
|--.env.example
|--.firebaserc
|--README.md
|--assets
|--components
|--docker-compose.yml
|--firebase.json
|--functions
|  |--.gitignore
|  |--index.js
|  |--package-lock.json
|  |--package.json
|--jest.config.js
|--layouts
|--middleware
|--nuxt.config.js
|--package-lock.json
|--package.json
|--pages
|--plugins
|--static
|--store
|--test
```

今回の例では上記のような構成のプロジェクトを用います。  
（約 1 年前に作ったポートフォリオサイトがこんな構成でした）

Nuxt.js と Firebase で別々に `package.json` を持っており、Nuxt.js で生成した静的サイトを Firebase でホスティングする形です。

処理の流れとしては以下のような形になります。

- `develop` ブランチの更新を検知して GitHub Actions が動く
- 最新のブランチをチェックアウト
- Nuxt.js で静的サイトを生成する
- 生成された静的サイトを Firebase にホスティングする

注意しておきたいのは、Firebase の Node.js のバージョンは 10.x 系を用いていて、Nuxt.js は 12.x 系の Node.js を使ってビルドする必要がある点です。  
（現時点での Firebase は古いバージョンの Node.js しか対応していないのが少し残念な感じではあります）

さて、それらを考慮して作成した yml ファイルが以下になります。

```yml
# Firebaseへの自動デプロイ
name: Deploy to Firebase

# develop ブランチの更新を検知して動くという記述
on:
  push:
    branches:
      - develop

jobs:
  firebase-deploy:
    runs-on: ubuntu-latest
    steps:
      # 最新のブランチをチェックアウト
      - name: Checkout
        uses: actions/checkout@v2

      # Nuxt.js ように Node.js をセットアップ
      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: "12.x"

      # Nuxt.js のビルドに .env を使用している場合、.env をサンプルから作成
      - name: Copy .env
        run: cp .env.example .env

      # Nuxt.js 依存性パッケージをインストール
      - name: npm install in Nuxt.js
        run: npm i

      # Nuxt.js で静的サイトを生成
      - name: Nuxt generate
        run: npm run generate

      # Firebase 用に Node.js をセットアップ
      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: "10.x"

      # Firebase 依存性パッケージインストール
      - name: npm install in Firebase
        run: cd functions && npm i

      # Firebase へデプロイ
      - name: deploy to Firebase
        uses: w9jds/firebase-action@master
        with:
          args: deploy --only hosting
        env:
          FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
          PROJECT_ID: your-firebase-project-id
```

さて、上記の GitHub Actions を使うには、Firebase のトークンを取得して `secrets.FIREBASE_TOKEN` に設定してあげる必要があります。

トークンを取得するためには、`firebase-tools` をインストールしてログインしてあげるだけです。
以下のコマンドを参考にしてください。

```bash
npm i -g firebase-tools
firebase login:ci
```

ログインに完了すると以下のようにトークンが表示されます。

```bash
Waiting for authentication...

✔  Success! Use this token to login on a CI server:

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

![GitHub Sercret](./github-sercret.png)

トークンが取得できたら GitHub から設定してあげれば完了です。

もし分からない場合でも、以下を参考にしながら行えばできるはずです。

https://fireship.io/snippets/github-actions-deploy-angular-to-firebase-hosting/

## トラブルシューティング

### Firebase の Functions のビルドで失敗する

```bash
Error: Error occurred while parsing your function triggers.

Error: 'linux-x64' binaries cannot be used on the 'linuxmusl-x64' platform. Please remove the 'node_modules/sharp' directory and run 'npm install' on the 'linuxmusl-x64' platform.
    at Object.hasVendoredLibvips (/github/workspace/functions/node_modules/sharp/lib/libvips.js:68:13)
    at Object.<anonymous> (/github/workspace/functions/node_modules/sharp/lib/constructor.js:7:22)
    at Module._compile (internal/modules/cjs/loader.js:1138:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1158:10)
    at Module.load (internal/modules/cjs/loader.js:986:32)
    at Function.Module._load (internal/modules/cjs/loader.js:879:14)
    at Module.require (internal/modules/cjs/loader.js:1026:19)
    at require (internal/modules/cjs/helpers.js:72:18)
    at Object.<anonymous> (/github/workspace/functions/node_modules/sharp/lib/index.js:3:15)
    at Module._compile (internal/modules/cjs/loader.js:1138:30)
```

`sharp` というライブラリが `linuxmusl-x64` のプラットフォームでは使えないと怒られています。

なので、npm でライブラリをインストールする際に、以下のようにプラットフォームを指定してあげましょう。

```yml
# 依存性パッケージをインストール
- name: npm install
  run: npm install --arch=x64 --platform=linuxmusl
```

これで問題なく動くはずです。
