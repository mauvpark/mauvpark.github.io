---
layout: post
title: useReducer와 useSyncExternalStore의 사용방법에 대한 정리
date: 2023-09-25 12:45:00 +0900
parent: React
categories: react, useReducer, useSyncExternalStore
nav_order: 4
comments: true
---

*2023-09-25 12:45 작성*

# useReducer와 useSyncExternalStore의 사용방법에 대한 정리
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

## useReducer와 useSyncExternalStore의 개념

### 1. useReducer
Redux와 마찬가지로 `reducer`를 사용하여 외부에서 상태를 저장 및 관리하며, `Store`에 저장된 값이 변경 되었을 경우(`Object.is`를 통해 비교), 컴포넌트를 업데이트 할 수 있게 한다.
`reducer`를 이용해 특정 컴포넌트만 업데이트 할 수 있어 A. 드릴링(Drilling)을 방지하고, B. 상태 관리를 한 눈에 볼 수 있으며, C. 최적화가 가능하다는 장점이 있다.

### 2. useSyncExternalStore
React에서 사용하지 않는 *외부 Store*가 존재할 경우, **해당 Store를 React 환경과 통합시키기 위해** 사용하는 Hook이다.
*React 환경이 아닌 Store*의 경우, 해당 `Store`가 업데이트 된다고 하더라도 컴포넌트를 Re-rendering 시키지 않으므로 `Store`의 값이 변경 되었음을 컴포넌트에 알려주어야 한다. 
이 때, 사용하는 것이 `useSyncExternalStore`다.

## useReducer와 useSyncExternalStore의 사용방법에 대한 구분

> `useReducer`와 `useSyncExternalStore` 모두 `Store`라는 외부 상태를 가지고 핸들링 한다는 공통점을 가지고 있다.
그러나 `useReducer`와 달리 `useSyncExternalStore`은 React에서 지원하지 않는 `Store`를 React와 통합시키기 위한 용도라는 점이 차이를 가진다.

따라서 `Store`를 통해 핸들링이 필요한 경우,
1. React 라이브러리 내에서 이용할 때는 `useState`와 `useReducer`를 활용하고,
2. 다른 환경 안에 React 코드를 집어넣고, 다른 환경의 `Store`를 React와 통합시키는 경우 `useSyncExternalStore`를 활용하는 것을 공식문서에서 추천하고 있다는 점을 기억하자.

## 라이센스

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="크리에이티브 커먼즈 라이선스" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />이 저작물은 <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">크리에이티브 커먼즈 저작자표시-동일조건변경허락 4.0 국제 라이선스</a>에 따라 이용할 수 있습니다.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
