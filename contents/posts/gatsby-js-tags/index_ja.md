---
title: "GraphQL で一意の値を取得することで Gatsby ブログにタグ一覧を作る"

category: "Tech"
lang: "ja"
date: "2019-09-19"
slug: "gatsby-js-tags"
draft: false
template: "posts"
tags:
  - Gatsby
---

![Gatsby](./gatsby.png)

Gatsby で作成したブログにタグ一覧を表示する機能を実装したので、その話です。

なお、本記事ではすでに記事に対してタグ機能が実装済みという前提で話が進んでいきます。  
GraphQL を用いて重複しないタグ情報を取得するということについて主に書いています。

## そもそも Gatsby はどのようにデータ取得しているのか

![GraphQL](./graphql-gui.png)

もうすでにご存知かもしれませんが、Gatsby では GraphQL を用いてデータを取得しています。  
GraphQL で取得したデータを元に、静的サイトを生成しているわけです。

Gatsby をローカル環境で動作している場合、[localhost:8000/\_\_\_graphql](http://localhost:8000/___graphql)にアクセスすることで、GraphQL のクエリを GUI から試してみることができます。  
GUI の左側の入力欄にクエリを入力して実行ボタンを押すと、取得したデータ（JSON 形式）を確認できます。

試しに、自分の環境で以下のクエリを実行してみます。

```JavaScript
{
  allMarkdownRemark(
    limit: 1000
    sort: { fields: [fields___date], order: DESC }
  ) {
    edges {
      node {
        html
        fields {
          slug
          date
        }
        frontmatter {
          title
          category
          tags
          date
        }
      }
    }
  }
}
```

登録されている記事のデータが取得できました。  
上記のクエリでは、取得データの上限を 1000 件に絞り、fields 配列の date 項目を降順（DESC）に並び変えて取得しています。

そのデータの中で、各記事のタグ情報は frontmatter 配列の中にある tags 項目に格納されています。

## どうやって重複しない値を GraphQL で取得するのか

さて、肝心の GraphQL を使って重複しない値を取得する方法ですが、`distinct`というクエリを使います。  
`distinct`を用いる事で重複を除いた一意の値を取得することが可能になります。

記事データを取得するクエリを元に、frontmatter 配列の中にある tags 項目から重複を除いてデータを取得するクエリを書いてみました。  
試しに、下記のクエリを実行してみます。

```JavaScript
{
  allMarkdownRemark(
    limit: 1000
    sort: { fields: [fields___date], order: DESC }
  ) {
    distinct(field: frontmatter___tags)
  }
}
```

全てのタグ情報から重複を除いてデータ取得できました。

あとはタグの一覧を表示するコンポーネントを作成するだけですね。

## StaticQuery を用いたコンポーネントを作成する

Gatsby で React コンポーネントから GraphQL を呼ぶ場合、StaticQuery というものを使います。  
この StaticQuery を使う事で、ページ生成用のファイル以外からも GraphQL を使う事ができます。

StaticQuery を用いてタグの一覧を表示するコンポーネントを作ってみました。  
Gatsby の Link コンポーネントを用いて、各タグが使われている記事の一覧を表示するページへのリンクもつけてみました。

```js
import React from "react";
import { useStaticQuery, graphql, Link } from "gatsby";

export const TagListing = ({ category, title }) => {
  // タグを取得
  const data = useStaticQuery(graphql`
    query {
      allMarkdownRemark(
        limit: 1000
        sort: { fields: [fields___date], order: DESC }
      ) {
        distinct(field: frontmatter___tags)
      }
    }
  `);

  const tags = data.allMarkdownRemark.distinct;

  // タグがあれば表示する
  if (!tags) {
    return null;
  }
  return (
    <ul>
      {tags.map((tag) => (
        <Link to={`/tags/${tag}`}>
          <li>{tag}</li>
        </Link>
      ))}
    </ul>
  );
};
```

上記のコンポーネントを使う事で、任意の場所にタグ一覧を表示させる事ができました。

## 終わりに

今回は、Gatsby でタグの一覧を表示する機能を作ってみましたが、  
カテゴリーの一覧を作るなどにも活用できるので、もし機会があればぜひ試してみてください。

## 参考

[Gatsby 公式 StaticQuery](https://www.gatsbyjs.org/docs/static-query/)
