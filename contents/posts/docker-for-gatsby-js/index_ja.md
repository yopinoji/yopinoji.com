---
title: "Gatsby 製のブログを Docker (Docker Compose) を使って動かす"

category: "Tech"
lang: "ja"
date: "2020-01-06"
slug: "docker-for-gatsby-js"
draft: false
template: "posts"
tags:
  - Gatsby
  - Docker
---

年末年始に PC のクリーンインストールを行い、思い切って PC を購入時の真っ白な状態に戻したのですが、  
その時に**「わざわざ Node.js を PC のインストールしなくても Docker で Node.js を動かせる環境を作ればよくない？」**とふと思い、  
当ブログの開発環境も Docker 化したのでその記録です。

## Docker 環境の紹介

今回は `Dockerfile` と `docker-compose.yml` の 2 つを Gatsby プロジェクト直下に配置して、Gatsby プロジェクトを Docker 化します。

Docker Compose は複数のコンテナを使う Docker 環境を YML ファイルに定義することで、それらを連動して起動できるツールです。  
今回はコンテナを複数使う訳ではないですが、Docker Compose を使うとコマンド 1 つで起動できて楽なので使います。

### docker-compose.yml

```yml
version: "3"
services:
  your-app-name:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: your-app-name
    tty: true
    command: npm run develop-in-container
    ports:
      - "8000:8000"
    volumes:
      - /usr/src/node_modules
      - .:/usr/src
```

まずは `docker-compose.yml` の紹介です。

とてもシンプルな構成かと思います。

`your-app-name` となっている箇所についてはご自由に書き換えてください。

Docker コンテナ内部の `/usr/src` フォルダで Gatsby を動かしています。  
8000 番のポートを解放して、PC から[localhost:8000](http://localhost:8000)にアクセスすることでサイトを確認できるようにしてあります。

また、コンテナ内部では `localhost` を使わずに `0.0.0.0` に開発環境をホスティングしてあげる必要があるので、開発環境立ち上げ用の npm スクリプトを以下のようにカスタマイズしてあります。

```json
{
  "scripts": {
    "develop": "gatsby develop",
    "develop-in-container": "gatsby develop -H 0.0.0.0"

}
```

`package.json` に上記のように `develop-in-container` というスクリプトを追加して、それを使うことで開発環境を `0.0.0.0` にホスティングすることができます。

### Dockerfile

```Dockerfile
FROM node:14-alpine
EXPOSE 8000

RUN \
  apk add --no-cache python make g++ && \
  apk add vips-dev fftw-dev --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/community --repository http://dl-3.alpinelinux.org/alpine/edge/main && \
  rm -fR /var/cache/apk/*

WORKDIR /usr/src
COPY ./package.json .
COPY . ./
RUN npm install
```

次は `Dockerfile` の紹介です。

Linux ディストリビューションは、ファイルの軽量さに定評のある Alpine Linux を用います。  
Alpine Linux を用いる際の注意点として、npm の `sharp` モジュールを動かすために `vips-dev` などのインストールが別途必要になります。  
こいつのインストールも `Dockerfile` に定義します。

その後、`package.json`を Docker コンテナにコピーして、それを元に依存性パッケージをインストールします。

Alpine Linux を用いた `Dockerfile` で動かなかった場合は以下を使ってくみてださい。

```Dockerfile
FROM node:14
EXPOSE 8000
WORKDIR /usr/src
COPY . ./
RUN npm install
```

こちらは Debian で動作するので、特に Linux に何かインストールする必要はありません。  
（ただ年月が経った場合、そのままで動作する保証はできません）

### .dockerignore

```ignore
node_modules
.dockerignore
Dockerfile
```

最後に `.dockerignore` の紹介です。  
`.dockerignore` というファイルを作成し、そこに除外対象のファイルを記入することで、Docker で扱うファイルから除外できます。

こいつは無くても動くはずです。  
ただ、ブラックホールより重いことに定評のある `node_modules` をコンテナにコピーするのは気が引けるので、一応用意します。

![heaviest_objects_in_the_universe](./heaviest_objects_in_the_universe.jpg)

### Docker 起動

さて、上で紹介した `Dockerfile` 、 `docker-compose.yml` 、 `.dockerignore` を Gatsby プロジェクト配下に配置しましょう。

配置したら下記のコマンドを入力します。

```bash
docker-compose up -d --build
```

初回はビルドに時間が少しかかりますが、問題なく動くはずです。  
もし、動かない場合はご連絡いただければ幸いです。

コンテナが起動したら[localhost:8000](http://localhost:8000)にアクセスします。  
問題なく Gatsby ブログが動いていれば問題ありません。

## コンテナの内部にシェルでアクセスしたい

コンテナの中にアクセスして何かをインストールしたいという場合、以下のコマンドでアクセス可能です。
`sh` でアクセスしていますが、コンテナ内部に `zsh` や `bash` をインストールすればそちらでアクセスすることも可能です。

```bash
docker-compose exec your-app-name sh
```
