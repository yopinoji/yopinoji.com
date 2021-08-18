---
title: "React の CSS-in-JS でより柔軟にスタイルを拡張するためのライブラリ Styled System の紹介"

category: "Tech"
lang: "ja"
date: "2020-10-14"
slug: "more-flexible-use-of-react-css-in-js-with-styled-system"
draft: false
template: "posts"
tags:
  - React
  - CSS-in-JS
---

**唐突ですが、CSS-in-JS を使い React コンポーネントを作る際、コンポーネントのスタイルを柔軟に変更する必要がある場合にどのようにコンポーネントを作るでしょうか？**

大抵の場合は、プロパティの値に応じてコンポーネントのスタイル変更をするのではないでしょうか。

簡単な例として、styled-components を用いて以下のようなコンポーネントを書いてみました。

```ts
import styled from "styled-components";

interface Props {
  label: string;
  color: string;
}

const WrappedButton = styled.button<Props>`
  font-size: 1rem;
  margin: 1rem;
  border-radius: 2rem;
  color: ${(color) => color};
`;

const Button: React.FC<Props> = ({ label, ...props }) => {
  return <WrappedButton {...props}>{label}</WrappedButton>;
};

export default Button;
```

色だけを動的に変更可能な単純な React コンポーネントですね。

また、以下のようにコンポーネント側でスタイル定義を予め用意しておいて、プロパティに応じてスタイルを使い分けるということもできますね。  
（こちらの方法の方がポピュラーな気がします）

```ts
import styled, { css } from "styled-components";

interface Props {
  label: string;
  type: ButtonType;
}

type ButtonType = "normal" | "special";

const WrappedButton = styled.button<Props>`
  ${({ type }) => renderStyle(type)}
  font-size: 1rem;
  margin: 1rem;
  padding: 1rem;
  border-radius: 1rem;
`;

const renderStyle = (type: ButtonType) => {
  switch (type) {
    case "normal":
      return css`
        color: black;
      `;
    case "special":
      return css`
        color: white;
        background-color: black;
      `;
    default:
      return ``;
  }
};

const Button: React.FC<Props> = ({ label, ...props }) => {
  return <WrappedButton {...props}>{label}</WrappedButton>;
};

export default Button;
```

このように styled-components（CSS-in-JS） を使えばコンポーネントに任意のスタイルを当てることは可能です。

**ただ、もっと柔軟にコンポーネントのスタイルを変更したいという需要がある場合はどうしますか？**  
例えば、コンポーネントのマージンや高さや幅の数値を指定したい場合などです。

先述した例のように受け取った値に応じて CSS のプロパティを変更するコンポーネントを作る方法も可能ですね。

また、styled-components を拡張するために CSS を変更するための関数を用意する方法があります。  
これなら関数をコンポーネント間で共有して再利用することもできそうです。  
以下は、引数に応じてマージンを調整する関数のサンプルです。

```ts
const Margin = ({ m = 0 }: { m: number }) => {
  return css`
    margin: ${m}rem;
  `;
};
```

ただ、そのような関数を自前で用意した場合に起こる問題として、CSS のプロパティの数に応じて関数を増やす必要があるのでコードの管理が大変なことになります。  
コンポーネントによって別々の関数を使われることがあれば目も当てられないような悲惨な状態になります。

また、プロパティを受け取る部分の TypeScript の型が、変更する CSS のプロパティに応じて増え続けるというのも考えものですね。  
以下は例です。

```ts
export default interface Props {
  tag?: "p" | "span" | "div";
  m?: number;
  mt?: number;
  mb?: number;
  p?: number;
  pt?: number;
  pb?: number;
  // So many Props there...
}
```

このように CSS-in-JS で柔軟に CSS の数値を変更して使いたい時って色々と面倒な問題も多いですよね？

そんな問題解決の手助けをしてくれるのが Styled System です。

## Styled System ことはじめ

さて、本題に入る前に [Styled System](https://styled-system.com) についてざっくり解説しておきます。

[Styled System](https://styled-system.com) は React で CSS-in-JS を使う際にスタイル適用の補助をしてくれる UI ライブラリです。

[Ant Design](https://ant.design/) や [Material-UI](https://material-ui.com/) のようにスタイルの付いた React コンポーネントを提供してくれる訳ではなく、あくまでスタイルを適用するための補助的なライブラリになります。

### Styled System を使ってみる

さて、ここからは実際に Styled System を使ったコンポーネントの例を見つつ解説していきます。

まず、ボタンの色を自由に変えることができるコンポーネントを Styled System で作ると以下のようになります。

```ts
import styled from "styled-components";
import { color, ColorProps } from "styled-system";

interface Props extends ColorProps {
  label: string;
}

const WrappedButton = styled.button<Props>`
  ${color}
  font-size: 1rem;
  margin: 1rem;
  border-radius: 2rem;
`;

const Button: React.FC<Props> = ({ label, ...props }) => {
  return <WrappedButton {...props}>{label}</WrappedButton>;
};

export default Button;
```

Styled System から変更したいプロパティが関連するモジュールをインポートして、styled-components で使うだけです。  
非常にシンプルかと思います。

使う際には以下のように使えます。  
文字色のプロパティだけでなく、背景色についてもプロパティを用意してくれています。

```ts
<Button color='white' />
<Button color='#f00' />
<Button color='white' backgroundColor='black' />
```

また、カラーコードなどを定数のようにプロジェクトで指定したいという場合は、別途テーマを設定することで以下のように書けます。  
（以下の例では、設定ファイルの `theme.colors.blue[0]` に設定した色が反映されます）

```ts
<Button color="blue.0" />
```

マージンや幅なども柔軟に変更できるようにしたい場合、以下のように変更できます。  
変更必要なプロパティに応じてインポートするモジュールを変える形ですね。

```ts
import styled from "styled-components";
import {
  color,
  ColorProps,
  layout,
  LayoutProps,
  space,
  SpaceProps,
} from "styled-system";

interface BaseProps {
  label: string;
}

type Props = ColorProps & LayoutProps & SpaceProps & BaseProps;

const WrappedButton = styled.button<Props>`
  ${color}
  ${layout}
  ${space}
  border-radius: 2rem;
`;

const Button: React.FC<Props> = ({ label, ...props }) => {
  return <WrappedButton {...props}>{label}</WrappedButton>;
};

export default Button;
```

上記の例だとインポートするモジュールが多くて冗長だと言う場合は、処理を共通化して再利用することも可能です。  
以下のようにまとめることができます。

```ts
import { css } from "styled-components/macro";
import { space, color, flexbox, layout, background } from "styled-system";

export const CommonStyledSystem = css`
  ${color}
  ${space}
  ${flexbox}
  ${layout}
  ${background}
`;
```

コンポーネント側でまとめたものを使う際は以下のように使えます。

```ts
import { CommonStyledSystem } from "./common";
const WrappedButton = styled.button<Props>`
  ${CommonStyledSystem}
`;
```

上記のように色やマージンなど様々なプロパティを動的に変更可能にしたコンポーネントを使う場合、以下のように変更したいプロパティを埋めるだけで使えます。

どういったプロパティが存在しているのかは Styled System のドキュメントに記載があるので、それを読むことで適宜スタイルを変更することができます。

```ts
<Button
  color="red"
  margin="auto"
  padding={12}
  width={100}
  display="inline-block"
/>
```

ところで、このスタイルの書き方どこかで似たようなものを見たことがありませんか？

私は [Tailwind CSS](https://tailwindcss.com/) に少し似ているなと思いました。

```html
<button class="text-red-600 m-auto p-12 w-20 inline-block">
  Tailwind CSS Button Sample
</button>
```

[Tailwind CSS](https://tailwindcss.com/) の場合、上記のように書きます。  
クラスに CSS プロパティと対応するクラスを追加していくスタイルです。  
カラーコードをテーマに設定してから使うということも同様に可能です。

ただ、tailwind CSS ではクラス名に追加していくという性質上 TypeScript を使う場合に型による制御がしづらいという欠点があると思います。

Style System を用いた場合、プロパティに対する型の定義が提供されているのでそのような問題も起こりづらいのは良い点だと思います。

## 最後に

他に良い方法などありましたら、共有いただければ幸いです。
