---
title: "React に Cypress & Percy を導入して CI で自動で E2E と画面差分のテストを行う"
category: "Tech"
lang: "ja"
date: "2021-04-12"
slug: "introducing-cypress-percy-in-react-app-and-test-automatically-with-ci"
draft: false
template: "posts"
tags:
  - GitHub
---

以下を用いて E2E Testing と Visual Regression Testing を行い、CI での自動テストも構築するというのが今回のお題です。

最初に Cypress から、次に Percy と順を追って解説していきます。

## Cypress とは？

ざっくりと説明するなら、「実際のブラウザと同じ機能を用いながらアプリの挙動をテストするためのツール」となると思います。

フロントエンドのテストツールとして、Jest や Selenium などがあります。  
それらと比較して Cypress は開発環境や本番環境をそのままテスト環境として使うことができるので設定がとても簡単です。

また、Jest を使ったテストは実際のブラウザではなく仮想ブラウザ環境で行うという制約がありました。  
しかし、Cypress ではブラウザの機能を使いながらテストが可能なため「ルーティング機能」や「レンダリングされたピクセルの情報」などもブラウザで確認するのと同じようにテスト可能です。

しかも、Jest と同じようにコードベースでテストケースを記述することができ、開発過程からもテストを容易に導入できます。

ただ、Jest でできていた Redux の値をテストするというような単体テストはできません。  
あくまでも、ブラウザを使った E2E Testing がメインになります。

今回は、まず Cypress を React アプリで動かして、CI で自動テストを実行するところから初めてみます。

## React プロジェクトに Cypress による E2E Testing を設定する

ローカル環境で Cypress を動かす場合、最初の準備として Cypress をローカル PC にインストールしておきましょう。  
もちろん、Docker などの環境で動かす場合には不要です。

https://docs.cypress.io/guides/getting-started/installing-cypress#Direct-download

![Cypress App for Mac](./cypress_desktop_app.png)

今回は [Create React App](https://create-react-app.dev/) で作った React プロジェクトという想定で話を進めていきます。
前提として、Node.js やそのパッケージ管理ツール（yarn, npm）をインストールしておく必要があります。

```
npx create-react-app cyoress-react-sample
cd cyoress-react-sample
yarn install
```

まだ、設定を行うプロジェクトが存在しない場合は、上記のコマンドで簡易的に React のサンプルを作成してしまいましょう。

```
yarn add -D cypress
```

まず、プロジェクトに `cypress` ライブラリをインストールします。

```
npx @bahmutov/cly init
```

次に Cypress の設定ファイルを生成します。
上記のコマンドを実行すると、各種の設定ファイルやサンプルコードが生成されるはずです。

```
cypress.json
cypress
|--README.md
|--fixtures
|--integration
|--plugins
|--screenshots
|--support
|--videos
```

`cypress/integration` の内部には、後ほどテストコードを書いていきます。

また、Cypress を npm scripts で即座に実行できるように、`package.json` に以下のようなスクリプトを追加しておきます。

```json
{
  "scripts": {
    "start": "react-scripts start",
    "cy:run": "cypress run",
    "cy:open": "cypress open"
  }
}
```

これで `npm run cy:run` または `yarn cy:run` のコマンドで Cypress を実行できるようになります。

また、`cypress.json` にテスト対象のローカル環境の URL を登録しておくと、`cy.visit('/path')`で任意のパスのテストを行うことができて便利です。

```json
{
  "baseUrl": "http://localhost:3000"
}
```

次に、試しに Cypress のテストを 1 件作成してみます。

```ts
// enables intelligent code completion for Cypress commands
// https://on.cypress.io/intelligent-code-completion
/// <reference types="cypress" />

context("Actions", () => {
  beforeEach(() => {
    cy.visit("/");
  });

  it("Test Link Text", () => {
    cy.get("a").contains("Learn React");
  });
});
```

試しに上記のテストコードを `cypress/integration/test.spec.js` に追加してみます。

![Default Create React App](./Default_Create_React_App.png)

このテストケースでは、`http://localhost:3000` の a タグ の中に「Learn React」という文字が含まれていることをテストしています。  
[Create React App](https://create-react-app.dev/) で作ったプロジェクトのデフォルト表示が正しいことをテストしています。

Cypress では、他にも色々なメソッドを用いてテストを行うことができます。  
詳しくは以下にあります。  
https://docs.cypress.io/api/table-of-contents

それでは早速テストしてみましょう。

```
yarn start
yarn cy:run
```

上記のコマンドを実行します。
React を `localhost:3000` に立ち上げてから、Cypress を動かします。

![Run Cypress Sample Test](./cypress_sample_test_run.png)

テストの結果がコマンドラインに上記のように表示されれば、Cypress の導入は完了です。

## CI/CD のワークフローで Cypress を動かす

さて、Cypress の導入は終わったものの、やはりプロジェクトが更新されるたびにバグやエラーがないことを自動で確認しておきたいですよね。  
そのために CI/CD のワークフローの中で Cypress を動かしてみます。

まず、コマンド 1 つでテストを実行できるように `package.json` に npm scripts のコマンドを追加します。  
以下のような形です。

```json
{
  "scripts": {
    "start": "react-scripts start",
    "cy:run": "cypress run",
    "cy:run-ci": "npx start-server-and-test start http-get://localhost:3000/ cy:run"
  }
}
```

今回は `cy:run-ci` という名称の npm scripts を `package.json` に追加しました。  
このスクリプトを用いることで、`localhost:3000` に React が立ち上がるのを待ってから、Cypress を実行できるようになります。

### GitHub Actions で Cypress を実行する

CI/CD に GitHub Actions を使う場合、実行するワークフローを記述した `yml` ファイルを作成する必要があります。

```
name: Cypress Tests

on: [push]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Cypress run
        uses: cypress-io/github-action@v2
        with:
          start: yarn cy:run-ci

```

基本的には Git で Push されるたびに `yarn cy:run-ci` を実行しているだけのワークフローです。  
これで Git での更新ごとに Cypress によるテストを実行して問題がないことを確認できますね。

もし、以下のエラーになる場合、Cypress のテスト録画ファイルのパスがおかしいことになっています。

```
Error: ENOENT: no such file or directory, stat '/home/runner/work/react-cypress-percy-example/react-cypress-percy-example/cypress/videos/app.spec.js-compressed.mp4'


Test run failed, code 1
More information might be available above
Cypress module has returned the following error message:
Could not find Cypress test run results
Error: Could not find Cypress test run results
```

Cypress のコンフィグで `videoUploadOnPasses` を無効化することでひとまず解決です。

```
{
  "baseUrl": "http://localhost:3000",
  "videoUploadOnPasses": false
}
```

## Jenkins で Cypress を実行する

環境によっては GitHub Actions を使えないこともあるので、その場合の例として Jenkins で Cypress を実行する方法についても軽く記載しておきます。

CI で Cypress を実行するために基本的に Docker を用いると楽です。
[Cypress 実行用の Docker イメージ](https://hub.docker.com/r/cypress/base)が配布されているのでそれを利用すると良いです。

さて、Jenkins で Cypress を実行するために、`Jenkinsfile` を書くと上記のようになります。

```
node {
    withDockerContainer(image: 'cypress/base:14', toolName: 'Default') {
        stage('Git Checkout') {
            checkout([$class: 'GitSCM', branches: [[name: '${BRANCH_NAME}']], doGenerateSubmoduleConfigurations: false, userRemoteConfigs: [[credentialsId: 'your_credentials', url: 'https://github.com/your_organization/your_project']]])
        }
        stage('Install Dependencies') {
            sh 'yarn install'
        }
        stage('Cypress') {
            sh 'yarn cy:run-ci'
        }
    }
}

```

Cypress 用の Docker コンテナで Cypress を実行しています。

あとはこのワークフローを Git の Push ごとに実行するだけです。

## Percy を使い Cypress による画面の変更差分をスナップショットとして取得する

さて、ここまでで E2E Testing を導入し、さらに更新があるたびに自動でテストを動かすところまでできました。

ここは、さらに一歩進んで、画面の変更差分を取得して Web から簡単にできるようにしてみませんか？
デザイナーや PM などの非エンジニア職が画面の変更を確認する際に喜ばれること間違いなしです。

それを実現できるのが、[Percy](https://percy.io/)です。

![Percy Impression](./Percy_crop_Impression.png)

Percy について詳細な解説はここでは省きますが、簡単に言うと「変更前後で画面の差分を比較することができるサービス」です。  
Visual Regression Testing を簡単に実施するためのツールとして優秀です。  
詳細な解説はインターネットの大海や公式サイトに情報が転がっているのでそちらに任せます。

![Percy wait build](./Percy_waiting_for_build.png)

まず最初に Percy に登録します。
プランによっては月額費用が必要になるので予め確認しておくのが良いでしょう。

登録後、プロジェクトを作成すると Percy がビルド待ちの状態になっているはずです。  
そのため、React プロジェクト側に移動して Percy と連携できるようにします。

Percy との連携方法にはいくつか方法があるのですが、今回は Cypress を既に導入済みなので Cypress で E2E Testing を行いつつ、同時に Percy 用のスナップショットを取得する方法を用います。

以下をプロジェクトにインストールします。

```
yarn add -D @percy/cypress
yarn add -D @percy/cli
```

`@percy/cypress` では Cypress のテストの中で Percy のスナップショットを使えるようにします。
`@percy/cli` では npm scripts で percy コマンドを実行可能にします。

```ts
// enables intelligent code completion for Cypress commands
// https://on.cypress.io/intelligent-code-completion
/// <reference types="cypress" />

context("Actions", () => {
  beforeEach(() => {
    cy.visit("http://localhost:3000/");
  });

  it("Test Link Text", () => {
    cy.get("a").contains("Learn React");
  });

  it("Snapshot", () => {
    cy.percySnapshot();
  });
});
```

続いて、先ほど作成した Cypress のテストケースに Percy のスナップショット取得用の関数を追加します。  
`cy.percySnapshot()` が動いた際に画面のスナップショット（画像）を取得して、Percy に連携してくれます。

また、Percy の関数を動かすために、`cypress/support/index.js` で先ほどインストールした `@percy/cypress` を使えるようにしておく必要があります。

```js
import "@percy/cypress";
```

上記の内容で `cypress/support/commands.js` を作成します。  
ここには Cypress で用いる関数をカスタマイズして定義することができます。

詳細は以下を参照してください。
https://docs.cypress.io/api/cypress-api/custom-commands

```js
import "./commands";
```

あとは、コマンドを `cypress/support/index.js` で読み込むだけです。

次は実際にテストを動かし、Percy に連携してみましょう。

```json
{
  "scripts": {
    "cy:run": "cypress run",
    "cy:snapshot": "percy exec -- yarn cy:run"
  }
}
```

テストを動かす前に、`cy.percySnapshot()` などの Percy 関連の関数を使うためには、あらかじめ Percy を立ち上げておく必要があります。  
そのため、上記のように npm scripts を変更します。

実際に動かしてみましょう。

```
export PERCY_TOKEN=***********************
yarn cy:snapshot
```

Percy で作成したプロジェクトからトークンを環境変数に登録してから、`yarn cy:snapshot` コマンドを実行します。

![Percy Sample](./Percy_sample.png)

Percy 側に戻ると、Cypress で取得したスナップショットが連携されていることが確認できます。

今回の例ではそもそも変更差分はありませんが、変更差分がある場合は Percy を使い差分をレビューできます。

## 終わりに

ちなみに、今回の実装内容は以下からも確認可能です。  
https://github.com/yopinoji/react-cypress-percy-example
