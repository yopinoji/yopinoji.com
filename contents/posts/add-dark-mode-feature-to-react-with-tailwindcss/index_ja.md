---
title: "React と Tailwind CSS だけを使ってダークモードに対応する"

category: "Tech"
lang: "ja"
date: "2020-12-16"
slug: "add-dark-mode-feature-to-react-with-tailwindcss"
draft: false
template: "posts"
tags:
  - React
  - Gatsby
  - Tailwind CSS
  - Dark Mode
---

2020 年 11 月 19 日にリリースされた Tailwind CSS の v2.0。

リリースされた新機能の中にダークモード対応があります。

https://blog.tailwindcss.com/tailwindcss-v2#dark-mode

これを活用すれば Tailwind CSS だけでダークモード対応もできてしまうわけです。

個人的にダークモードを選択することが多いので是非とも試してみたい、というわけで試してみました。

## Tailwind CSS について簡単におさらい

簡単に言うと、クラスベースで CSS のスタイリングをするためのフレームワークです。

```html
<div class="text-gray-300 bg-red-700 rounded px-2">Sample Div</div>
```

提供されるのはあくまでもクラスに対応する CSS だけ。  
Bootstrap のように予め定まったデザインが用意されているというわけではなく、自分で自由にスタイリングができるので、デザインの凝った実装が必要な場合でも使いやすいです。

度重なるデザイン修正やリニューアルなどで 1 つの CSS を長く書いていると、 CSS がゴチャゴチャしてカオスで読みづらいものになるという経験をされている方も多いと思います。  
そのようなカオスを CSS を弄らずに解決できるソリューションの 1 つでもあります。

https://tailwindcss.com/

## Tailwind CSS を導入する

あとで書くかもしれませんが、検索すれば情報が手に入ると思うので一旦省きます。

## Tailwind CSS と React でダークモードに対応する

今回使用する Tailwind CSS のバージョンは以下の通りです。  
最初に述べた通り、Tailwind CSS でダークモードを有効にするには v2.0 以上が必要です。

```json
{
  "tailwindcss": "^2.0.1"
}
```

まず、設定からダークモードを有効にする必要があります。  
`tailwind.config.js` に以下を追記します。

```js
// tailwind.config.js
module.exports = {
  // darkMode: "media",
  // if you neeed to support toggling Dark mode
  darkMode: "class",
  // ...
};
```

`prefers-color-scheme: dark` を使ってダークモードを有効にする場合は、`darkMode: "media"` を設定します。  
HTML のクラスを使用してダークモードを有効にする場合は、`darkMode: "class"` を設定します。

今回はクラスを利用してダークモードを後から切り替えられるように実装したいので、`darkMode: "class"` を選択します。

次に、サイトを読み込んだ際にクラスを変更するスクリプトを追加します。

```js
// Set "Dark Mode" option
if (
  localStorage.theme === "dark" ||
  (!("theme" in localStorage) &&
    window.matchMedia("(prefers-color-scheme: dark)").matches)
) {
  document.querySelector("html").classList.add("dark");
} else {
  document.querySelector("html").classList.remove("dark");
}
```

上記のスクリプトを実行すると、実行環境に応じて `html` タグに `dark` というクラスを追加します。

この `dark` というクラスが存在する場合はダークモードが有効になります。

普通の React を使っている場合は、`React.useEffect()` などを使用してサイト読み込み時に上記のスクリプトを実行してあげてください。

Gatsby を使っている場合は `gatsby-browser.js` に追記すればサイト読み込み時に実行されます。

これでサイトを読み込む環境に応じてダークモードを有効にする実装は完了です。  
あとはダークモードが有効になった際の見た目を変更するだけです。

```tsx
import React from "react";

export const Sample: React.FC = ({ ...props }) => {
  return (
    <div className="text-black dark:text-white dark:bg-black dark:hover:bg-red-400">
      {props.children}
    </div>
  );
};
```

Tailwind CSS でダークモード有効時の CSS プロパティは `dark:` の付いたクラスで識別できます。  
ダークモード有効時にはこのクラスが優先的に読み込まれます。  
`hover:` などと併用することも可能です。

上記の例の場合、ダークモード有効時には背景が黒、文字色が白になります。

最後に、ダークモードを切り替えるための実装です。

```ts
const handleOnChange = (v: boolean) => {
  if (v) {
    document.querySelector("html")?.classList.add("dark");
  } else {
    document.querySelector("html")?.classList.remove("dark");
  }
};
```

ダークモードの切替は至って簡単で、`dark` というクラスを追加または削除する処理を追加するだけです。  
上の実装例はサンプルですが、クラスを操作するだけでダークモードを切り替えることができます。

切り替えるためのボタンは自前で好きなものを用意してください。

トグルボタンを自前で用意するのがめんどくさい場合は、私が Tailwind CSS を使って 30 分くらいで雑に書いたトグルボタンの React コンポーネントを以下に置いておくので使ってください。

```tsx
import React, { useState } from "react";

interface ToggleProps {
  onChange?: (v: boolean) => void;
  text?: string;
}

const Toggle: React.FC<ToggleProps> = ({ onChange, text, ...props }) => {
  const [checked, setChecked] = useState(false);
  return (
    <div className="flex items-center justify-center w-full m-2" {...props}>
      <label className="flex items-center cursor-pointer">
        <div className="relative">
          <input
            type="checkbox"
            className="hidden"
            onChange={() => {
              setChecked(!checked);
              onChange && onChange(checked);
            }}
          />
          <div
            className={`w-8 h-4 bg-gray-300 rounded-full shadow-inner ${
              checked && "bg-green-50"
            }`}
          ></div>
          <div
            className={`absolute w-4 h-4 bg-gray-50 rounded-full shadow inset-y-0 left-0 transition-transform transform ${
              checked && "translate-x-full bg-green-300"
            }`}
          ></div>
        </div>
        <div className="ml-3 text-gray-700 font-medium">{text}</div>
      </label>
    </div>
  );
};

export default Toggle;
```

## 最後に

というわけで、Tailwind CSS で自分のサイトにダークモード機能を追加してみました。
とても簡単にできるのでおすすめです。
