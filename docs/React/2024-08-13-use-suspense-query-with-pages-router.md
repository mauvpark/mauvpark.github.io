---
layout: post
title: How to use UseSuspenseQuery with Next js's Pages Router
description: Client side rendering without Hydration error
date: 2024-08-13 00:00:00 +0900
parent: React
categories: react,next-js,react-query,useSuspenseQuery,hydration,error,csr,pages-router
nav_order: 3
comments: false
---

_2024-08-16 업데이트_

# How to use UseSuspenseQuery with Next js's Pages Router

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

## How to use `useSuspenseQuery()` in Next js's Pages Router

If you want to render lazy components as client side rendering(CSR), you could use `dynamic()` with `{ssr: false}` option in Next js. This [lazy loading](https://nextjs.org/docs/pages/building-your-application/optimizing/lazy-loading) prevents server side rendering as the doc says.

If you use `useSuspenseQuery()` without using solution above, Pages Router reads all api data from server side.

To adjust Skeleton UI in client side, you need two conditions.

1. First, Loader should be shown until a component is loaded.
2. Component should be waited until api data is fully fetched.

### Component lazy load

Component's lazy load can be accomplished by using `next/dynamic` and `{ssr: false}` option if you want csr.

```javascript
const ExampleComponent = dynamic(() => import("./exmaple"), {
  ssr: false,
  loading: () => <Loading />,
});

const MyPage = () => {
  return <ExampleComponent />;
};
```

### `useSuspenseQuery()` lazy load

To use `useSuspenseQuery()`, you need a fallback component which is called `Suspense`.

```javascript
// ./doc
const Doc = () => {
  const { data } = useSuspenseQuery({
    queryKey: ["DOCS"],
    queryFn: () => axios.get("https://www.example.com/docs"),
  });
  return (
    <ul>
      {data.body.map((doc, index) => (
        <li key={index}>{doc.name}</li>
      ))}
    </ul>
  );
};

// ./suspenseDoc
import Doc from "./doc";
import Loader from "./loader";

const SuspenseDoc = () => (
  <Suspense fallback={<Loader />}>
    <Doc />
  </Suspense>
);

export default SuspenseDoc;
```

### Cases of using both useSuspenseQuery and dynamic altogether

#### Case 1

```javascript
// ./doc
const Doc = () => {
  const { data } = useSuspenseQuery({
    queryKey: ["DOCS"],
    queryFn: () => axios.get("https://www.example.com/docs"),
  });
  return (
    <ul>
      {data.body.map((doc, index) => (
        <li key={index}>{doc.name}</li>
      ))}
    </ul>
  );
};

// ./page
const Doc = dynamic(() => import("/doc"), { ssr: false, loading: <Loader /> });

const Page = () => (
  <Suspense fallback={<Loader />}>
    <Doc />
  </Suspense>
);
```

When you use this code, you will face a following error.

_Error: The server could not finish this Suspense boundary, likely due to an error during server rendering. Switched to client rendering._

This means Next js handles `Suspense` from server default. But `Suspense` should be rendered from client side so this is not the what we are expecting.

#### Case 2

```javascript
// ./suspenseDoc
import Doc from "./doc";
import Loader from "./loader";

const SuspenseDoc = () => (
  <Suspense fallback={<Loader />}>
    <Doc />
  </Suspense>
);

export default SuspenseDoc;

// ./page
const SuspenseDoc = dynamic(() => import("/suspenseDoc"), {
  ssr: false,
  loading: <Loader />,
});

const Page = () => <SuspenseDoc />;
```

Next js's `dynamic()` with `{ssr: false}` option will make component be rendered from client side. Then we can expect `Suspense` in dynamic component also will be rendered in client side. So, it means there will be no hydration error.

## Conclusion

After I tested [Case2](#case-2), finally I got the lazy component with `useSuspenseQuery()` with no hydration error.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
