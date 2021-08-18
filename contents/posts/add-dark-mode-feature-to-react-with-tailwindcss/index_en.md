---
title: "Support dark mode using React and Tailwind CSS"

category: "Tech"
lang: "en"
date: "2020-12-16"
slug: "en/add-dark-mode-feature-to-react-with-tailwindcss"
draft: false
template: "posts"
tags:
  - React
  - Gatsby
  - Tailwind CSS
  - Dark Mode
---

Tailwind CSS v2.0 was released on November 19, 2020.

Among the new features released is dark mode support.

https://blog.tailwindcss.com/tailwindcss-v2#dark-mode

If you take advantage of this update, you can use Tailwind CSS alone to support dark mode.

I personally use dark mode most of the time, so I really wanted to try it out, so I did.

## A brief review of Tailwind CSS

It is a framework for class-based CSS styling.

```html
<div class="text-gray-300 bg-red-700 rounded px-2">Sample Div</div>
.
```

Only the CSS for the classes is provided.  
It is easy to use even if you need to implement elaborate designs, because it does not come with a predetermined design like Bootstrap, but allows you to do your own styling.

I'm sure many of you have had the experience of writing a single CSS for a long time due to frequent design revisions or renewals, which results in a chaotic and unreadable mess of CSS.  
This is one of the solutions to solve such chaos without messing with CSS.

https://tailwindcss.com/

## Implement Tailwind CSS in your project

I may write about this later, but I'm sure you can find the information by searching, so I'll skip it for now.

## Using Tailwind CSS and React to support dark mode

The version of Tailwind CSS we will be using is as follows.  
As mentioned earlier, v2.0 or higher is required to enable dark mode in Tailwind CSS.

```json
{
  "tailwindcss": "^2.0.1"
}
```

First, we need to enable dark mode in the configuration.  
Add the following to `tailwind.config.js`.

```js
// tailwind.config.js
module.exports = {
  // darkMode: "media",
  // if you neeed to support toggling Dark mode
  darkMode: "class",
  // ...
};
```

If you want to enable dark mode using `prefers-color-scheme: dark`, set `darkMode: "media"`.  
If you want to enable dark mode using HTML classes, set `darkMode: "class"`.

In this case, select `darkMode: "class"` since we want to implement the dark mode so that we can switch it later using the class.

Next, add a script that will change the class when the site is loaded.

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

When you run the above script, it will add the class `dark` to the `html` tag depending on the execution environment.

If this class `dark` is present, dark mode will be enabled.

If you are using regular React, you can use `React.useEffect()` to run the above script when the site loads.

If you are using Gatsby, add it to `gatsby-browser.js` and it will be executed when the site loads.

This completes the implementation of enabling dark mode according to the environment in which the site is loaded.  
The only thing left to do is to change the appearance of the site when dark mode is enabled.

```tsx
import React from "react";

export const Sample: React.FC = ({ . .props }) => {
  return (
    <div className="text-black dark:text-white dark:bg-black dark:hover:bg-red-400">
      {props.children}
    </div>.
  );
};
```

In Tailwind CSS, CSS properties when dark mode is enabled can be identified by a class with `dark:`.  
When dark mode is enabled, this class will be loaded first.  
This class can be used in conjunction with `hover:` and other classes.

In the above example, when dark mode is enabled, the background is black and the text color is white.

Finally, here is the implementation for switching the dark mode.

```ts
const handleOnChange = (v: boolean) => {
  if (v) {
    document.querySelector("html")? .classList.add("dark");
  } else {
    document.querySelector("html")? .classList.remove("dark");
  }
};
```

Switching the dark mode is quite simple, just add or remove the class `dark`.  
The implementation above is a sample, but you can switch the dark mode just by manipulating the class.

You can prepare your own buttons to switch the mode.

If you don't want to prepare a toggle button by yourself, you can use the React component of a toggle button that I wrote in about 30 minutes using Tailwind CSS.

```tsx
import React, { useState } from "react";

interface ToggleProps {
  onChange?: (v: boolean) => void;
  text?: string;
}

const Toggle: React.FC<ToggleProps> = ({ onChange, text, . .props }) => {
  const [checked, setChecked] = useState(false);
  return (
    <div className="flex items-center justify-center w-full m-2" {. .props}>
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
            }}
          ></div>.
          <div
            className={`absolute w-4 h-4 bg-gray-50 rounded-full shadow inset-y-0 left-0 transition-transform transform ${
              checked && "translate-x-full bg-green-300"
            }}
          ></div>.
        </div>
        <div className="ml-3 text-gray-700 font-medium">{text}</div> </div>
      </label>
    </div>.
  );
};

export default Toggle;
```

## Finally

So, I've tried to add a dark mode feature to my site with Tailwind CSS.
It is very easy to do and I recommend it.
