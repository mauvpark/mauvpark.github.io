---
layout: post
title: useEffect와 DOM 조작
date: 2021-11-05 22:40:00 +0900
parent: React
categories: react, useEffect, dom
nav_order: 3
comments: false
---

*2022-03-13 22:40 작성*

# useEffect와 DOM 조작
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

## useEffect에서 DOM 조작하기

useEffect에서 documnet를 이용해 scrollIntoView() 등의 직접 조작을 할 때, 정상적으로 조작되지 않는 경우가 있다.

```js
useEffect(() => {
    // id를 이용해 조작한다고 가정
    document.getElementById('id').scrollIntoView();

    // React useRef를 이용해 조작한다고 가정
    ref.current.scrollIntoView();
}, [])
```

위와 같은 명령어를 실행하게 되면, 정상적으로 실행되지 않는 경우가 있다. 그 이유는 useEffect가 실행 될 때 dom이 그려지지 않아 접근할 수 없기 때문에 발생한다.

그리고 이는 useRef를 사용하는 경우에도 동일하게 적용된다.

<br/>

## 해결 방법

그럼에도 불구하고 useEffect를 이용해 접근해야 하는 경우가 존재할 수 있다. 예를 들어, 어떤 페이지에서 돌아와 화면 조작이 되어야 하는 경우가 있을 수 있겠다.

이 경우, 다음과 같이 `setTimeout()`을 통해 dom이 그려지길 기다렸다가 그리는 방법을 대체재로서 사용할 수 있다.

```js
useEffect(() => {
    setTimeout(() => {
         // id를 이용해 조작한다고 가정
        document.getElementById('id').scrollIntoView();

        // React useRef를 이용해 조작한다고 가정
        ref.current.scrollIntoView();
    }, 300)
}, [])
```

잠시의 간격을 둠으로써 dom이 생성 되기를 인위적으로 기다리고, 그 후에 조작을 하도록 한다.
