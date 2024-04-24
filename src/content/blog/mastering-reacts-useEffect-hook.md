---
author: Ryan Jordan
pubDatetime: 2023-09-18
title: "Mastering React's `useEffect` Hook"
slug: mastering-reacts-useeffect-hook
draft: false
tags:
  - react
  - hooks
description: "React has revolutionized the way we think about UI development, turning it into a more component-centric approach. One of the hooks that stands out in making this approach seamless is the `useEffect` hook."
---

React has revolutionized the way we think about UI development, turning it into a more component-centric approach. One of the hooks that stands out in making this approach seamless is the `useEffect` hook. In this post, we'll dive deep into understanding this powerful tool.

#### What is `useEffect`?

At its core, `useEffect` allows you to perform side effects in function components. Think of side effects as any action that interacts with the outside of your component, such as data fetching, manual DOM manipulations, or subscriptions.

In class components, we had lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` to handle these side effects. `useEffect` encompasses all these lifecycle methods into a unified API.

#### Basic Usage

The basic structure of `useEffect` looks like this:

```javascript
useEffect(() => {
  // Your side effect code here
});
```

By default, this effect will run after every render of your component.

#### Dependency Array

The real power of `useEffect` comes when you introduce the dependency array:

```javascript
useEffect(() => {
  // Runs when `someValue` changes
}, [someValue]);
```

If `someValue` changes between renders, the code inside `useEffect` will run. If it remains the same, React will skip this effect, optimizing performance.

An empty dependency array `[]` means the effect will only run once, similar to `componentDidMount`.

#### Cleanup with `useEffect`

Some side effects need to be cleaned up to prevent memory leaks, like event listeners or subscriptions. `useEffect` allows you to return a function for this purpose:

```javascript
useEffect(() => {
  const handleResize = () => console.log(window.innerWidth);
  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

This ensures that the event listener is removed when the component is no longer in use.

#### Practical Example: Theme Toggler

Let's build a simple theme toggler using `useEffect`:

```javascript
const [theme, setTheme] = useState("light");

useEffect(() => {
  document.body.className = theme;
}, [theme]);
```

Here, whenever our `theme` state changes, the class of the document's body will update, allowing us to switch between light and dark themes using CSS.

#### Conclusion

The `useEffect` hook is a testament to React's commitment to creating a more intuitive and powerful UI development experience. By mastering this hook, you can ensure your UI is truly reactive and can handle any side effects your application needs.
