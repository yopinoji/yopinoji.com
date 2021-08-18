---
title: "Auth0 による認証機能を備えた Hasura GraphQL サーバーを Nuxt.js 用に構築する"

category: "Tech"
lang: "ja"
date: "2020-06-03"
slug: "build-hasura-graphql-server-with-authentication-function-by-auth0-for-running-in-nuxt-js"
draft: false
template: "posts"
tags:
  - Auth0
  - GraphQL
  - Hasura
  - Nuxt.js
---

さて、今回の題目は、Hasura を使った GraphQL サーバーに対して、Auth0 を用いた認証機能（ログイン機能）を付けてみるというものです。

Hasura については、[こちらの過去記事](quick-build-graphql-server-by-hasura-with-nuxt-js)で述べているので説明を省略します。

なお、この記事が参考にしている内容は下記の英語文献なので、そちらも参考にしてみてください。

https://auth0.com/blog/building-a-collaborative-todo-app-with-realtime-graphql-using-hasura/

## 前提条件

- Hasura が AWS EC2 などパブリックなネットワークの上で起動している
- Auth0 のアカウントを作成済み
- Nuxt.js を触ったことがあり、自力でセットアップできる

## 今回作成する認証機能のデータの流れ

今回作成するものの大まかなデータの流れとしては以下になります。

![Auth0 with Hasura flowchart](./auth0_with_hasura_flowchart.png)

1. Auth0 を用いてログインする（Auth0 にユーザーの情報を登録）
2. ログインしたユーザーのデータを Hasura で使っている PostgreSQL に連携する

Auth0 でログインした情報を Hasura 側に連携する際には、Auth0 の Rules という機能を用います。

## Hasura を JWT モードで起動する

![Hasura JWT mode](./hasura_jwt_mode.png)

今回、JWT による認証を入れる都合で、Hasura を JWT モードで起動する必要があります。  
`HASURA_GRAPHQL_JWT_SECRET` と `HASURA_GRAPHQL_ADMIN_SECRET` を設定してあげれば、JWT モードで起動します。

実際の yml ファイルでいうと、以下のような感じです。  
（DB は AWS RDS に用意されている前提です）

```yml
version: "2"
services:
  graphql-engine:
    image: hasura/graphql-engine:latest
    ports:
      - "80:8080"
    restart: always
    environment:
      HASURA_GRAPHQL_DATABASE_URL: postgres://hasura:postgrespassword@dev-hasura.test12345678.ap-northeast-1.rds.amazonaws.com:5432/dev_hasura
      HASURA_GRAPHQL_ACCESS_KEY: password
      HASURA_GRAPHQL_ENABLE_CONSOLE: "true" # set to "false" to disable console
      HASURA_GRAPHQL_ENABLED_LOG_TYPES: startup, http-log, webhook-log, websocket-log, query-log
      ## generate by https://hasura.io/jwt-config/
      HASURA_GRAPHQL_JWT_SECRET: '{"type": "RS512", "key": "-----BEGIN CERTIFICATE-----\nTEST1234567890\n-----END CERTIFICATE-----"}'
      HASURA_GRAPHQL_ADMIN_SECRET: sercretkey
```

`HASURA_GRAPHQL_ADMIN_SECRET` については、任意の文字列を指定してください。

`HASURA_GRAPHQL_JWT_SECRET` については、Auth0 のアカウントを作成済みであれば以下のサイトから生成可能です。

https://hasura.io/jwt-config/

![Hasura generate jwt config](./hasura_generate_jwt_config.png)

Hasura の JWT モードについての詳細は以下を確認してください。

https://hasura.io/docs/1.0/graphql/manual/auth/authentication/jwt.html#generating-jwt-config

## Auth0 Rules を設定する

### Rules とは

- Auth0 提供する機能の一部
- Rules は、ユーザーがアプリケーションに対して認証するときに実行される
- 認証プロセスが完了すると実行され、Auth0 の機能をカスタマイズおよび拡張するために使用可能

より詳細な情報は以下を参照してください。

https://auth0.com/docs/rules

日本語の情報だと、以下がまとまっていると思います。

https://future-architect.github.io/articles/20200128/

### Hasura に対して、アクセスしているユーザーの権限を伝える JWT を宣言するための Rules

```js
function (user, context, callback) {
  const namespace = "https://hasura.io/jwt/claims";
  context.idToken[namespace] =
    {
      'x-hasura-default-role': 'user',
      // do some custom logic to decide allowed roles
      'x-hasura-allowed-roles': user.email === 'admin@foobar.com' ? ['user', 'admin'] : ['user'],
      'x-hasura-user-id': user.user_id
    };
  callback(null, user, context);
}
```

上記のコードでは、ログインユーザーのメールアドレスが `admin@foobar.com` の場合のみ、`admin` 権限で Hasura を実行できます。

「コード上の `context.idToken` って何？」という疑問がある場合、以下を読むと解消するかもしれません。

https://auth0.com/docs/rules/references/context-object

![Hasura data permisson](./hasura_data_permisson.png)

Hasura を使用する権限については上記の画面から追加できます。  
初期では `admin` のみ設定されています。

一般ユーザーが使う `user` 権限についても追加する必要があります。

なお、権限がないのにリソースへアクセスしようとすると、以下のようなエラーでデータが取得できないことになってしまいます。

```bash
Error: GraphQL error: field "users" not found in type: 'query_root'
```

データがないという風に読み取れるので最初は焦るかもしれませんが、単純に権限がなくてデータにアクセスできない時もこのエラーが出るようです。（執筆時点での情報）

### Auth0 の情報を Hasura に連携する Auth0 Rules

Auth0 で保持しているユーザー情報を Hasura の DB に連携してあげるための Rules は以下のように記述します。

```js
function userSyncRule(user, context, callback) {
  const userId = user.user_id;
  const email = user.email;

  const mutation = `mutation($userId: String!, $email: String) {
    insert_users(objects: [{
        auth0_id: $userId,
        email: $email
      }],
      on_conflict: {
        constraint: users_pkey,
        update_columns: [last_login]
      }) {
        affected_rows
      }
    }`;

  request.post(
    {
      headers: {
        "content-type": "application/json",
        "x-hasura-admin-secret": configuration.HASURA_GRAPHQL_ADMIN_SECRET,
      },
      url: "http://your-graphql-endpoint/v1/graphql",
      body: JSON.stringify({ query: mutation, variables: { userId, email } }),
    },
    function (error, response, body) {
      console.log(body);
      callback(error, user, context);
    }
  );
}
```

`url` の項目には、各自の Hasura のエンドポイントを記入してください。（なお、`localhost` で動作する Hasura に対しては連携できないと思います）

今回は、ユーザーの Auth0 での ID とメールアドレスを Hasura 側に連携していますが、Auth0 で持っている情報であれば大抵は連携可能です。  
Rules で参照可能な Auth0 が持っているユーザー情報については以下を参照可能です。

https://auth0.com/docs/rules/references/user-object

## Hasura で使用するテーブルを PostgreSQL に作成する

Hasura 側で保持する Users テーブルを作成します。

- id - integer(auto-increment), primary key
- email - text
- auth0_id - text
- created_at - timestamp with time zone, default: now()
- last_login - timestamp with time zone, default: now()

今回はテーブルの定義は上記のような形で作成します。  
（各自のプロジェクトに合わせて適宜書き換えてください）

これらのユーザー情報を保持するテーブルは、Auth0 にログインした際に先ほどの Rules を使って Hasura に登録されます。

## Auth0 による認証を Nuxt.js で実現する

最後に Auth0 による認証機能を実際のアプリケーションに落とし込む作業です。

Nuxt.js と Hasura を使った環境のセットアップは、[こちらの過去記事](./quick-build-graphql-server-by-hasura-with-nuxt-js)にも書いてあるので参考にしてください。  
（GraphQL を使って、Hasura と情報のやりとりをする方法までは載っています）

### @auth0/auth0-spa-js を利用してログイン機能を実装する手順

[`@auth0/auth0-spa-js`](https://auth0.github.io/auth0-spa-js/) を使います。

```bash
npm install @auth0/auth0-spa-js
```

また、今回は Nuxt.js のプラグイン機能にログイン関数を閉じ込めるので、`plugins/auth0.js` を作成します。  
`nuxt.config.js` にプラグインを登録する必要があります。

`plugins/auth0.js` の Auth0 クライアントを生成している箇所については、各自のプロジェクトで使われている Auth0 の設定を記述してください。  
（例では `.env` から読み込んでいます）

```js
import { Auth0Client } from "@auth0/auth0-spa-js";

export default (context, inject) => {
  // Auth0 クライアントの作成
  const auth0Client = new Auth0Client({
    domain: process.env.AUTH0_DOMAIN,
    client_id: process.env.AUTH0_CLIENT_ID,
    redirect_uri: window.location.origin,
    cacheLocation: "localstorage",
  });

  const login = () => {
    auth0Client.loginWithRedirect();
  };

  const loginPopup = () => {
    auth0Client.loginWithPopup();
  };

  const getToken = async () => {
    const isAuthenticated = await auth0Client.isAuthenticated();
    let accessToken = "";
    if (isAuthenticated) {
      const JWT = await auth0Client.getIdTokenClaims();
      const idToken = JWT.__raw;
      window.localStorage.setItem("idToken", idToken);
      accessToken = window.localStorage.getItem("idToken");
    }
    return accessToken;
  };

  const logout = async () => {
    auth0Client.logout();
  };

  const auth0 = {
    login,
    loginPopup,
    logout,
    syncUser,
  };
  inject("auth0", auth0);
};
```

`plugins/auth0.js` は何をやっているのか簡単に説明すると、`@auth0/auth0-spa-js` からログインやログアウトの関数を作成し、それを Vue コンポーネントから呼び出し可能なように注入しています。  
これで Auth0 のログインやログアウトなどの機能を以下のように画面側で呼び出すことができるようになります。

```js
HandleOnClick() {
  this.$auth0.login()
}
```

`@auth0/auth0-spa-js` が提供してくれている関数には様々な物があるので必要に応じて注入してください。

### ログイン画面の実装

```ts
// pages/login.vue
<template>
  <div>
    <i @click="login">Click to Login</i>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'Login',
  layout: 'default',
  components: {
    //
  },
  mounted() {
    //
  },
  methods: {
    login() {
      this.$auth0.loginPopup()
    }
  }
})
</script>

```

Auth0 と Nuxt.js Auth Module を使ったログイン画面は上記のように実装します。  
たった 1 行のメソッドを実行するだけです。

### ログアウト画面の実装

```ts
// pages/logout.vue
<template>
  <div>
    <i @click="logout">Click to Logout</i>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'Logout',
  layout: 'default',
  components: {
    //
  },
  mounted() {
    //
  },
  methods: {
    async logout() {
      await this.$auth0.logout()
    }
  }
})
</script>

```

ログアウトも同様です。  
注入した関数を呼び出すだけです。

### Hasura GraphQL にアクセスする

さて、最後に Hasura からデータ取得を行いましょう。

[過去記事](quick-build-graphql-server-by-hasura-with-nuxt-js)で述べた情報に加えて、以下のことが必要になります。

- Apollo のリクエストヘッダーに JWT のトークンを持たせる

```ts
async mounted() {
  const token = await this.$auth0.getToken()
  this.$apolloHelpers.onLogin("Bearer " + token)
}
```

例として、上記のようにしてあげると Nuxt.js では設定を行うことができます。

## 最後に

長文になってしまった影響でいくつか情報が抜けていないか心配なのですが、以上になります。

作業メモみたいな感じで書いたので、伝わりづらい箇所などあると思います。

誤った情報など見つけられた場合、お手数ですがお気軽にお知らせいただけると幸いです。
