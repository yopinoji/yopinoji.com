---
title: "React Library Comparison for Implementing Parallax Effect Animation"

category: "Tech"
lang: "en"
date: "2020-12-08"
slug: "en/add-parallax-effect-animation-with-react"
draft: false
template: "posts"
tags:
  - React
---

I wanted to implement a UI with parallax effect easily using React, so I tried using various libraries.

## Implementation using react-parallax

Let's start with the react-parallax library.

https://github.com/rrutsche/react-parallax

```bash
npm i react-parallax
```

The versions we tried are as follows.

```json
{
  "dependencies": {
    "react-parallax": "^3.1.2"
  }
}
```

The samples we implemented are as follows.

```jsx
import React from "react";
import { Parallax } from "react-parallax";
import "./styles.css";
const image1 =
  "https://images.unsplash.com/photo-1498092651296-641e88c3b057?auto=format&fit=crop&w=1778&q=60&ixid=dW5zcGxhc2guY29tOzs7Ozs%3D";

const insideStyles = {
  background: "white",
  padding: 20,
  position: "absolute",
  top: "50%",
  left: "50%",
  transform: "translate(-50%,-50%)",
};
export default function App() {
  return (
    <div className="App" style={{ height: "1000px" }}>
      <Parallax bgImage={image1} strength={500}>
        <div style={{ height: 500 }}>
          <div style={insideStyles}>HTML inside the parallax</div>
        </div>
      </Parallax>
      <Parallax
        bgImage={image1}
        strength={200}
        renderLayer={(percentage) => (
          <div>
            <div
              style={{
                position: "absolute",
                background: `rgba(255, 125, 0, ${percentage * 1})`,
                left: "50%",
                top: "50%",
                borderRadius: "50%",
                transform: "translate(-50%,-50%)",
                width: percentage * 500,
                height: percentage * 500,
              }}
            />
          </div>
        )}
      >
        <div style={{ height: 500 }}>
          <div style={insideStyles}>renderProp</div>
        </div>
      </Parallax>
    </div>
  );
}
```

Depending on the amount of scrolling, the value of the argument passed to the function `renderLayer()` changes, so you can rewrite the style using the value to achieve parallax animation by scrolling.

The provided API is relatively easy to understand, so you can proceed with the implementation without any learning cost.

The size after the build is as follows.

![react-parallax](./react-parallax-bundle.png)

I think it's a good category to call it lightweight.

### Thoughts on using react-parallax

- It is originally implemented in TypeScript, so type support is expected.
- Light in size after build
- Implementation of animations other than the parallax effect is not expected.

I've created a sample below, so feel free to rewrite it and try different things.

<iframe src="https://codesandbox.io/embed/sad-shamir-z3v6r?fontsize=14&hidenavigation=1&theme=dark"
  style="width:80%; height:400px; border:0; border-radius: 4px; overflow:hidden; margin:auto;"
  title="react-parallax"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

## react-scroll-parallax を使った実装

Next, let's use the react-scroll-parallax library.  
As the name of the library suggests, it will be used only for scrolling animation.

https://github.com/jscottsmith/react-scroll-parallax

```bash
npm i react-scroll-parallax
```

The versions we tried are as follows.

```json
{
  "dependencies": {
    "react-scroll-parallax": "^2.3.5"
  }
}
```

You need to wrap the root component or the area where react-scroll-parallax is used with ParallaxProvider.

```js
import React from "react";
import { ParallaxProvider } from "react-scroll-parallax";
import Root from "./Root";
const mountNode = document.getElementById("root");

ReactDom.render(
  <ParallaxProvider>
    <Root />
  </ParallaxProvider>,
  mountNode
);
```

If you are using Gatsby, you need to wrap it in `gatsby-browser.js` with ParallaxProvider as shown below.  
(In the current version of Gatsby, `gatsby-browser.js` acts as the root file.)

```js
import React from "react";
import { ParallaxProvider } from "react-scroll-parallax";

export const wrapRootElement = ({ element }) => (
  <ParallaxProvider>{element}</ParallaxProvider>
);
```

Since we only need to pass numerical values for X and Y, it can be implemented quickly without much learning cost.

In the following example, a random and irregular parallax effect is implemented by generating random numbers.

```tsx
import React from "react";
import { Parallax } from "react-scroll-parallax";

const SampleParallax: React.FC = (props) => {
  return (
    <>
      {[...Array(20).keys()].map((row, index) => (
        <Parallax
          y={[70, 100]}
          x={[Math.round(Math.random() * 100), Math.round(Math.random() * 100)]}
          key={index}
        >
          <p>TEST</p>
        </Parallax>
      ))}
    </>
  );
};
export defalut SampleParallax;
```

The size after the build is following.

![react-scroll-parallax](./react-scroll-parallax-bundle.png)

It seems to be surprisingly large in size for its limited functionality.

### Comments on react-scroll-parallax

- Easy to implement only parallax effect.
- Can't expect to implement animations other than the parallax effect
- It's hard to implement complex animations.
- TypeScript support is weak in the current version.

## Implementation using react-spring

Finally, I found that it is possible to implement parallax using react-spring, which I think is one of the best animation libraries in React.

https://www.react-spring.io/docs/props/parallax

```bash
npm install react-spring
```

```json
{
  "dependencies": {
    "react-spring": "^8.0.27"
  }
}
```

The following is a sample of how to make a parallax-like movement by moving the mouse cursor.

```jsx
import React from "react";
import { useSpring, animated } from "react-spring";

const calc = (x, y) => [x - window.innerWidth / 2, y - window.innerHeight / 2];
const trans1 = (x, y) => `translate3d(${x}px,${y}px,0)`;
const trans2 = (x, y) => `translate3d(${x / 10}px,${y / 10}px,0)`;

const App = (props) => {
  const [springProps, setSpring] = useSpring(() => ({
    xy: [0, 0],
    config: { mass: 10, tension: 550, friction: 140 },
  }));

  const onMouseMoveHandle = (e) =>
    setSpring({ xy: calc(e.clientX, e.clientY) });

  return (
    <>
      {[...Array(20).keys()].map((row, index) => {
        return (
          <div onMouseMove={onMouseMoveHandle} key={index}>
            <animated.div
              style={{ transform: springProps.xy.interpolate(trans1) }}
            >
              <p>{row}</p>
            </animated.div>
            <animated.div
              style={{ transform: springProps.xy.interpolate(trans2) }}
            >
              <p>{row}/</p>
            </animated.div>
          </div>
        );
      })}
    </>
  );
};
export default App;
```

By setting a value to the function generated by the `useSpring()` function, you can dynamically manipulate the target CSS.

In the above example, the `translate3d()` of the target CSS is manipulated by detecting the `onMouseMove` event to realize animation by mouse movement.

You can use your own CSS properties like `translate3d()`, and you can implement the animation as you want by starting from any event.

The size after the build looks like the following.

![react-spring](./react-spring-bundle.png)

It's not much different from react-scroll-parallax.

### Thoughts on react-spring

- It is originally implemented in TypeScript, so type support is expected.
- It can be used to implement animations other than parallax effects.
- A bit of learning cost is required to understand the various APIs provided.
- Can be used to implement animations with more detailed movement specifications.

I've prepared a sample in CodeSandbox, so feel free to play around with it.

<iframe src="https://codesandbox.io/embed/react-spring-ti6s4?fontsize=14&hidenavigation=1&theme=dark"
  style="width:80%; height:400px; border:0; border-radius: 4px; overflow:hidden; margin:auto;"
  title="react-spring"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

Also, the official website provides a variety of samples, so you may want to refer to them.

https://www.react-spring.io/docs/hooks/examples

## Summary

I guess it's a matter of using the right tool for the right job.

If you want something lightweight and simple, react-parallax is a good choice.  
If you want something lightweight and simple, use react-parallax. If you want to use it for other animations, use react-spring.
