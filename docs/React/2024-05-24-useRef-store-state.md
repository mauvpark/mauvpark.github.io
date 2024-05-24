---
layout: post
title: Use case of useRef which substitutes for useState
description: Some cases useRef can be considered for storing data.
date: 2024-05-24 00:00:00 +0900
parent: React
categories: React
nav_order: 3
comments: false
---

_2024-05-24 작성_

# Use case of useRef which substitutes for useState

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## `useRef` can store latest data without rendering.

I use `useState` almost every case to store state(=data). Because I use these _new_ data to render a page as the updated. But if you use `useRef` not `useState`, the page would not be rendered even though it has the latest data. Usually this is regarded as a bug(_actually this is misunderstanding about useRef_).

## Has `useRef` to be used only for manipulating DOM?

In many cases, `useState` is a better hook but if there are following conditions which hook do you want to use?

### Situation

1. You want to make a page and it will have several _steps(page like) components._ The page's data should be submitted when the step is at the end.
2. Each step component does not interact each other only parent component(_the page_) knows it.
3. Parent Component has many logics for handling each step component and I want that logics are rendered as little as possible.

#### `useState` based code

```jsx
// Page.tsx
const Page = () => {
  const [route, setRoute] = useState("one"); // Step router
  const [parentStore, setParentStore] = useState({ id: "", marketName: "" });
  const [parentReview, setParentReview] = useState({ id: { rate: 0, description: "" } });
  ...other states

  const handleRoute = () => {...};
  const handleStore = (localData) => {...};
  const handleReview = (localData) => {...};
  ...other logics

  const renderChildren = () => {
    switch (route) {
      case "one":
        return <Store handleParentStore={handleStore} />
        ...
    }
  }

  return (
    <main>
      {renderChildren()}
    </main>
  );
};

export default Page;
```

```jsx
// Store.tsx
const Store = (handleParentStore, handleRoute) => {
  const [childStore, setChildStore] = useState({ id: "", marketName: "" }); // The page's state goes down here.
  const handleSubmit = () => {
    handleParentStore(childStore) // Propagate local data to the parent
    handleRoute(); // Go to next step
  };
  return (
    <div>{...}</div>
  );
}

export default Store;
```

In this case, every data is stored in the each state where it belongs. For example, `<Store />` component's data will be stored in `childStore` state. Then its state is propagated through `handleSubmit()`. At this point in `handleSubmit()` function two different useState hooks are used, `setParentStore`, `setRoute`. The page would render **only once** when the step is submitted with this strategy.

_Even though `setParentStore` and `setRoute` is used, why does the page render once?_

React has **[auto batch](https://react.dev/learn/queueing-a-series-of-state-updates)** function from version 18. When it reads event handler, if it has multiple `setState`, it deals with all `setState`s then generate a render.

#### `useRef` based code

```jsx
// Page.tsx
const Page = () => {
  const [route, setRoute] = useState("one"); // Step router
  const storeRef = useRef({ id: "", marketName: "" }); // useState => useRef
  const reviewRef = useRef({ id: { rate: 0, description: "" } }); // useState => useRef
  ...other states // useState => useRef

  const handleRoute = () => {...};
  const handleStore = () => {...};
  const handleReview = () => {...};
  ...other logics

  const renderChildren = () => {
    switch (route) {
      case "one":
        return <Store storeRef={storeRef} {...} /> // useState => useRef
        ...
    }
  }

  return (
    <main>
      {renderChildren()}
    </main>
  );
};

export default Page;
```

```jsx
// Store.tsx
const Store = (storeRef, handleRoute) => {
  const [store, setStore] = useState({ id: "", marketName: "" }); // The page's state goes down here.
  const handleSubmit = () => {
    storeRef.current = store; // Propagate local data to the parent
    handleRoute(); // Go to next step
  };
  return (
    <div>{...}</div>
  );
}

export default Store;
```

What is different? Only `useState` is changed to `useRef` in the page. The data which is stored in `useRef` will be used only when set a step component's initial data and communicate with API. So, rendering is no matter in this case.

## Conclusion

At the first, I had wondered "Isn't it possible to optimize rendering counts with `useRef`. But that thought was solved with react's **[auto batch](https://react.dev/learn/queueing-a-series-of-state-updates)**. Still I believe this strategy can be used. So, if you have any situation to use this strategy for optimization, try out with deep consideration.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
