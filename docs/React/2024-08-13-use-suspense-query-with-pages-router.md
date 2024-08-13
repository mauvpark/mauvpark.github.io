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

_2024-08-13 작성_

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

In Pages Router, `useSuspenseQuery()` with dynamically imported component is drawn in client side very first time and then the rendering is captured as server side html following an error message. It means somewhat reason `useSuspenseQuery()` is not fully rendered by client side, rather that is tangled in server side.

To deal with this problem, needed to fetch `useSuspenseQuery()` in client side only. So I focused on `React.lazy()` wrapper for lazy load.

First, I wanted to check 'Is `useSuspenseQuery()` fetched in client side only when I use `React.lazy()`?'.

```javascript
// Component
const ExampleComponent = () => {
  const { data } = useSuspenseQuery({
    queryKey: ["EXAMPLE_KEY"],
    queryFn: () => axios.get("https://example.co.kr/myDoc"),
  });

  return (
    <>
      {data.body.map((doc, index) => (
        <div key={index}>{doc.name}</div>
      ))}
    </>
  );
};

// export Component
const LazyExample = React.lazy(async () => {
  const Component = await import("./ExampleComponent");

  return {
    default: () => <Component.default />,
  };
});

export default LazyExample;
```

As a result, It threw an error that it can't pre-render so those renderings will be passed to client side rendering.

So, `lazy` wrapper is fully passing component rendering to client side. To remove the error, it seems including `dynamic()` would be the solution.

```javascript
const LazyExample = dynamic(() => import("./lazyExample"), {
  ssr: false,
  loading: () => <Loading />,
});

const Page = () => {
  return (
    <Suspense fallback={<Loading />}>
      <LazyExample />
    </Suspense>
  );
};
```

`dynamic()`'s `loading` property is for the lazy component. And `Suspense`'s `fallback` attribute is for the `useSuspenseQuery()`'s lazy data. Finally hydration error was gone from my project.

Pages Router tries to render in server side as much as possible. This is not the official solution but if you want to deal with hydration error, you could try this.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
