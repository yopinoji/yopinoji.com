---
title: "PHP でクライアントとサーバの IP アドレスを取得する"

category: "Tech"
lang: "ja"
date: "2020-02-21"
slug: "get-ip-address-with-php"
draft: false
template: "posts"
tags:
  - PHP
---

PHP を用いて IP アドレスを取得する方法についてのメモです。

## 現在ページをみているクライアントの IP アドレス

こいつは簡単です。

```php
$_SERVER['REMOTE_ADDR'];
```

Laravel なら下記でも取得できます。

```php
\Request::ip()
```

ロードバランサーやリバースプロキシを挟んでいるサーバの場合は、以下も試してみてください。

```php
$_SERVER['HTTP_X_FORWARDED_FOR']
```

## プログラム実行しているサーバーの IP アドレス

こいつは色々と厄介です。  
複数台でサーバー分散されていたり、サーバーの構成が違ったりすると、特に面倒です。

まずは PHP から用意されている、`$_SERVER` を使うパターン。

```php
$_SERVER['SERVER_ADDR'];
```

上記で上手く行けばここで終了です。

ただ、上記だとサーバー自身のプライベート IP が取れてしまい、グローバル IP が取れないことがあります。

次のパターンは、以下です。

```php
$hostname = "www.google.com";
$ip = gethostbyname($hostname);
```

ホスト名を指定して IP を取得するパターンです。

上記ならグローバル IP が取れます。  
ただ、サーバー複数で分散してるとダメですね。  
それにホスト名が変わることも考慮せねばならず、保守性も悪いです。

なので、`curl` コマンドを使った下記を使います。

```php
$ip = rtrim(`curl inet-ip.info 2>/dev/null`);
```

```php
$ip = rtrim(`curl ifconfig.me　2>/dev/null`);
```

inet-ip.info、ifconfig.me は HTTP リクエストを投げた元のグローバル IP を返却してくれるサービスです。

これを利用することで簡単にサーバのグローバル IP を取得可能です。

速度的には、`curl inet-ip.info` の方が良いと思います。
