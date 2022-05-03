---
layout: post
title: useEffect와 window scroll
date: 2021-11-05 22:40:00 +0900
parent: React
categories: react, useEffect, scroll
nav_order: 3
comments: false
---

*2022-03-13 22:40 작성*
*2022-05-03 13:17 수정*

# useEffect와 window scroll
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

## useEffect와 window scroll

useEffect에서 documnet를 이용해 scrollIntoView() 등의 직접 조작을 할 때, 정상적으로 조작되지 않는 경우가 있다.

```js
const [data, setData] = useState([]);

useEffect(() => {
 (async () => {
   const res = await axios.get('url');
   setData(res.data);
 })()
}, [])

useEffect(() => {
    // target이라는 id를 이용해 조작한다고 가정
    document.getElementById('target').scrollIntoView();
}, [])

// const containerStyle = {width: '100vw', height: '100vh'};
// const apiDataStyle = {width: '100vw', height: 100, background: 'blue'};
// const targetStyle = {width: '100vw', height: 500, background: 'green'};

return (
  <div>
  {data.map((e, i) => 
    <div key={i}>{e.name}</div>
  )}
    <div id='target'>target</div>
  </div>
)
```

위와 같은 명령어를 실행하게 되면, 정상적으로 실행되지 않는 경우가 있다. 그 이유는 useEffect가 실행 될 때 dom이 그려지지 않아 접근할 수 없기 때문에 이 문제가 발생하게 된다.

보통 dom이 그려지지 않아 원하는 위치로 scrolling이 되지 않는 상황은 useEffect 내에서 api를 불러와 정보를 업데이트 한 후에 scrolling을 해야 할 때에 생기는데, 이는 *비동기 처리된 api*의 정보를 불러와 화면 상에 뿌리는 시점이 scrolling을 해야 하는 시점보다 느리기 때문이다.

<br/>

## 해결 방법

그럼에도 불구하고 useEffect를 이용해 접근해야 하는 경우가 존재할 수 있다. 예를 들어, 어떤 페이지에서 돌아와 화면 조작이 되어야 하는 경우가 있을 수 있겠다.

이 경우, 다음과 같이 `setTimeout()`을 통해 dom이 그려지길 기다렸다가 그리는 방법을 대체재로서 사용할 수 있다.

```js
const [data, setData] = useState([]);

useEffect(() => {
 (async () => {
   const res = await axios.get('url');
   setData(res.data);
 })()
}, [])

useEffect(() => {
  setTimeout(() => {
    // api에서 불러오는 데이터가 0.5초면 충분하다고 가정했다고 할 때
   document.getElementById('target').scrollIntoView();
  }, 500)
}, [])

// const containerStyle = {width: '100vw', height: '100vh'};
// const apiDataStyle = {width: '100vw', height: 100, background: 'blue'};
// const targetStyle = {width: '100vw', height: 500, background: 'green'};

return (
  <div>
  {data.map((e, i) => 
    <div key={i}>{e.name}</div>
  )}
    <div id='target'>target</div>
  </div>
)
```

잠시의 간격을 둠으로써 api 처리가 적당히 되는 시점에 조작을 하는 방식이다. 따라서 setTimeout이 종료되는 시점이 api가 데이터를 불러오는 시점과 같거나 늦어질 수록 정확한 지점에 scrolling이 될 것이라는 점을 예측할 수 있다.
