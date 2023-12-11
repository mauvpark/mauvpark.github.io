---
layout: post
title: NEXT JS 14 버전의 App Router와 Pages Router 그리고 App Router의 Middleware trigger 공백 시간에 대해
date: 2023-12-07 14:16:00 +0900
parent: React
categories: react, nextjs, next14, middleware, router-cache, app-router, pages-router
nav_order: 4
comments: true
---

*2023-12-07 14:16 작성*

# NEXT JS 14 버전의 App Router와 Pages Router 그리고 App Router의 Middleware trigger 공백 시간에 대해
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

## Pages Router와 App Router

### Pages Router
App Router 기능을 사용하길 원하지 않고 기존의 프로젝트 구조를 유지하고 싶은 개발자는 프로젝트의 root에 `pages` 폴더를 생성해 개발하면 된다. 개인적으로 Pages Router는 개발 진입에 있어 장벽이 높지 않다는 장점이 있다고 생각한다.
이것저것 코드를 집어 넣어도 기본적으로 client에서 실행되기 때문에 디버깅 난이도도 쉽고 나중에 추가로 필요할 경우, 캐싱을 하면 된다는 장점이 있다.

### App Router
NEXT JS 13 버전부터 App Router를 본격적으로 사용할 수 있게 되었다. 그리고 이 App Router를 사용하기 위해서는 프로젝트의 root에 `app` 폴더를 생성하여 시작할 수 있다.
기본적으로 App Router는 서버 컴포넌트를 default로 하는데 이에 따라 caching 기능을 더 강력하게 사용할 수 있고, client의 기기 부하를 많이 줄여줄 것으로 기대하고 있다.
다만 App Router는 서버 컴포넌트가 기본이기 때문에 서버 쪽에서 실행되어 client로 가져오는 것이 많아 고려해야 되는 것들이 더 많다고 느껴진다.

## 각 Router에서의 Middleware 실행
App Router와 Pages Router에 다음과 같은 파일이 있다고 하자.

```js
route: /
export default Page () {
  return (
    <div>
      <h1>Hello, this is HOME!</h1>
      <Link href="/a">Go to A page!</Link>
    </div>
  );
}

route: /a
export default Page () {
  return (
    <div>
      <h1>Hello, this is A!</h1>
      <Link href="/b">Go to B page!</Link>
    </div>
  );
}

route: /b
export default Page () {
  return (
    <div>
      <h1>Hello, this is B!</h1>
      <Link href="/a">Go to A page!</Link>
      <Link href="/">Go to Home page!</Link>
    </div>
  );
}
```

그리고 middleware는 다음과 같이 설정되어 있다.

```ts
export function middleware(request: NextRequest) {
  console.log("Middleware is running at: ", request.nextUrl.pathname);
}
```

이 때, 각 페이지를 route하게 되면 Pages Router와 App Router의 middleware는 어떻게 동작하게 될까?

**Pages Router**
> Middleware is running at: / <br/>
> Middleware is running at: /a <br/>
> Middleware is running at: / <br/>
> Middleware is running at: /a <br/>
> ...반복

**App Router**
> Middleware is running at: / <br/>
> Middleware is running at: /a <br/>
> Middleware is running at: / <br/>
> ... 캐시된 페이지 이동에 한해 30초간 동작 안 함

위와 같이 Pages Router에서는 페이지를 실행하기 전 매번 middleware가 실행된다. 그렇기에 매 페이지를 이동할 때 마다 middleware가 trigger 될 것이라는 것을 신뢰할 수 있다.
그러나 현재 NEXT JS 14.0.3 버전의 AppRouter는 **처음에는 cache 되지 않은 새페이지를 방문할 때만 middleware가 실행된다.**
그리고 **30초 후부터(동적 렌더링인 경우)** 매번 middleware가 실행된다. App Router의 middleware가 30초 간 동작하지 않는 이유는 [Route Cache](https://nextjs.org/docs/app/building-your-application/caching#duration-3) 때문으로 보인다.
Routing이 될 때, Next js에서는 Router Caching을 하게 되고, 이에 따라 사용자가 앞으로 가기나 뒤로 가기를 눌러도 이전 페이지의 상태를 유지하며(새로고침을 하지 않고) 빠르게 로드할 수 있게 된다.
App Router에서 caching을 통해 사용자는 사이트를 보다 끊김없이 이용할 수 있음을 보장한다는 강력한 장점이 있다. 그러나 App Router를 이용하는데 익숙하지 않은 개발자는 middleware가 뚫리는 시간이 존재한다는 것을 놓칠 수 있다.

다음의 경우는 어떨까?

```ts
let i = 0;
export function middleware(request: NextRequest) {
  i++;
  console.log("request", request.nextUrl.pathname);
  // 어떤 특정 조건에서 url이 redirect 되어야 하는 상황일 경우
  if (i > 10) {
    return Response.redirect(new URL("/b", request.url));
  }
}
```

위의 상황에서 AppRouter를 이용하는 프로젝트에서 페이지를 이동하면 `i`의 값이 10을 넘었을 때, redirect가 일어날까?

**그렇지 않다.** 앞서 말했듯이 30초 간 middleware는 작동하지 않으므로, i의 값은 카운트 값은 올라가지도 않고 계속 접근 가능할 것이다.
만약 이벤트 페이지가 Client의 현재 시간이라던지 특정 조건을 통해 매번 middleware에 의해 실행을 보장 받은 뒤 redirect 시켜야만 하는 페이지가 있다고 해보자.
그리고 어떤 Client가 종료 시간 전 찰나의 타이밍에 접속했다면 이벤트 시간이 지나고도 30초 간은 페이지를 이용할 수 있게 된다는 말이 될 수 있다.
즉, Caching으로 인해 middleware를 신뢰할 수 없게 된 점이 큰 단점이다.

이것은 Next JS 개발자들이 놓쳤다기 보다는 의도한 결과인 것으로 보인다.

[Router Cache Invalidation](https://nextjs.org/docs/app/building-your-application/caching#invalidation-1) 문서에 따르면, useRouter의 `refresh`를 통해 페이지의 상태를 유지한 채, caching이 되지 않도록 페이지를 리렌더링 할 수 있다.

예를 들어, `refresh`를 통해 서버 컴포넌트를 최신화 시켜 middleware가 해당 페이지에서 매번 작동하도록 한다.

```ts
'use client'
import router from 'next/navigation';
import {useEffect} from 'react';

export default Page() {
  const router = useRouter();

  useEffect(() => {
    router.refresh();
  }, [router]);

  return <div>{...}</div>;
}
```

또 다른 방법으로 `revalidatePath`를 사용할 수도 있다. 가고자 하는 페이지에 캐싱을 하지 않고 revalidation을 진행함으로써 middleware가 작동하도록 하는 것이다.

```ts
export default Page() {
  return (
    <Link
    href="/b"
    onClick={() => revalidatePath("/b")}
    >
      Go to B
    </Link>
  );
}
```

## 정리
**App Router**
App Router를 통해 caching의 기능을 최대한 이끌어내 사용자 경험을 향상시킬 수 있다.
그 외에도 App Router 만의 구조설계 방식을 통해 쉽게 Error Boundary 및 Loading 처리를 할 수 있고, url을 통해 하나의 페이지에서도 또다른 별개의 페이지로 구분하여 렌더링 시킬 수 있다.
그러나 서버 컴포넌트와 캐싱에 대한 이해가 필수적이고, 기존 프레임워크와 달리 Client 중점이 아닌 Server 중점적 처리임을 인지하고 개발에 임해야 한다는 점은 개발 진입 난이도를 높일 것으로 생각된다.

**Pages Router**
기본적인 규칙만 지킨다면 개발 난이도가 높지 않고, 선택적으로 서버 API를 사용해 성능을 향상시킬 수 있다. 그러나 Error Boundary 및 Loading 등을 자체적으로 설계해야 하고, App Router의 좋은 기능을 사용할 수 없다는 단점이 있다.

## 라이센스

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="크리에이티브 커먼즈 라이선스" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />이 저작물은 <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">크리에이티브 커먼즈 저작자표시-동일조건변경허락 4.0 국제 라이선스</a>에 따라 이용할 수 있습니다.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io"
        issue-term="pathname"
        label="✨💬✨"
        theme="github-dark"
        crossorigin="anonymous"
        async>
</script>
