---
title: "GitHub Actionsを使って、プルリクエストごとにビルドが成功するかどうかを確認する"

category: "Tech"
lang: "ja"
date: "2020-12-28"
slug: "use-github-actions-to-check-build-is-passed-for-each-pr"
draft: false
template: "posts"
tags:
  - GitHub Actions
  - GitHub
---

GitHub を用いて何名かのチームで開発をしていると、メンバーから送られてくるプルリクエストを確認する必要がありますよね。

ただ、毎回手動でプログラムのビルドが通るかどうか確認するのは少々しんどいです。

なので、GitHub Actions で自動化してしまいましょうという話です。

## GitHub Actions でビルドを確認する

`.github/workflows/PR-Check.yml` というファイルで作成を作成します。

```yml
# .github/workflows/push.yml
name: PR-Check

# On every pull request
on:
  pull_request:
    branches:
      - master
      - develop

jobs:
  PR-Check:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 12.x
        uses: actions/setup-node@v1
        with:
          node-version: 12.x
          registry-url: https://registry.npmjs.org/
      - name: Build Check
        run: |
          npm install
          npm run build
```

今回の例では、Node.js を用いて `npm run build` というコマンドで、プログラムのビルドを実行します。

![GitHub PR check](github_pr_check.png)

これでプルリクエストごとに GitHub Actions が自動でビルドが成功するかどうか確認できるようになりました。

## GitHub でのブランチの保護設定

せっかくプルリクエストごとにビルドが成功するかどうか自動で確認できるようになったので、チェックが通らないプルリクエストはマージできないように設定しておきましょう。

GitHub の GUI からプルリクエストの向き先になるブランチに対してマージ前のチェックを必須にしておきます。

![GitHub branch protection](github_branch_protection.png)

GitHub でのブランチの保護については、詳しくは以下を読んでください。

https://docs.github.com/en/free-pro-team@latest/github/administering-a-repository/enabling-branch-restrictions

![GitHub require to check](github_require_to_check.png)

ブランチの保護を有効にすることで、上記の画像のようにチェックを通ってないプルリクエストに警告を表示させることができています。
