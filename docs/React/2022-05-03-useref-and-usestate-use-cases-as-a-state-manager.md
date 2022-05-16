---
layout: post
title: useRef와 useState의 state manager로서의 사용에 관해
date: 2022-05-03 17:45:00 +0900
parent: React
categories: react, use-effect, use-ref, state
nav_order: 4
comments: false
---

*2022-05-03 17:45 작성*

# useRef와 useState의 state manager으로서의 사용에 관해
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

## useRef와 useState의 사용처

**useState**는 상태를 업데이트하고 이를 바탕으로 컴포넌트(component)를 렌더링(rendering)하는데 사용된다. 따라서 컴포넌트에 사용된 useState의 값이 변하게 되면 컴포넌트는 다시 렌더링 되며 변한 값을 화면에 표시한다.

**useRef**는 엘리먼트(element)의 속성을 참조하여 엘리먼트를 조작하거나 엘리먼트로부터 값을 얻어내는데 사용된다. useRef는 앞서 기술한 바에 대한 용도가 주된 사용처이나 특정한 경우에 상태 관리를 하는데 사용할 수도 있다. 이는 useRef의 값이 *최신값으로* 변경되어도 리렌더링(re-rendering)을 직접 수행하지 않는다는 것에 착안한다.

## useState의 사용과 한계

보통 useState를 사용해 상태관리를 한다고 하면 아래의 동작에 따라 수행될 것이다.

```js
const [data, setData] = useState<{id: number; name: string;}[]>([{id: 0, name: ''}]);

useEffect(() => {
  getData();
}, [])

const getData = async () => {
    const res = await fetch("api/testData.json", {
        headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
        },
    });
    const data = await res.json();
    setData(data);
}

return (
    <div>
        {data?.map((e: {id: number; name: string;}, i: number) => 
            <p key={e.id}>{e.name}</p>
        )}
    </div>
)
```

1. API에서 데이터를 페칭(fetch)한다.

2. 비동기 처리된 데이터를 useState에 저장한다.

3. useState의 상태가 업데이트 되면서 렌더링이 발생한다.

4. 화면이 업데이트 된다.

보통 useState를 사용한다면 위와 같이 값이 할당 되고 비동기적으로 업데이트 되어 리렌더링 해 화면을 업데이트하는 과정은 똑같을 것이다.

그러나, return 구문 안에서 분류한 특정 값을 저장해 초기 화면 구성에 사용하고자 한다면 어떻게 될까? 예를 들면 아래와 같은 식이다.

```js
const [data, setData] = useState<{id: number; name: string;}[]>([{id: 0, name: ''}]);
const [count, setCount] = useState<number>(0);

useEffect(() => {
  getData();
}, [])

const getData = async () => {
    const res = await fetch("api/testData.json", {
        headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
        },
    });
    const data = await res.json();
    setData(data);
}

return (
    <div>
        {data?.map((e: {id: number; name: string;}, i: number) => {
            // Lorem Ipsum 구문에서 아래의 단어를 가졌을 경우 카운트한다.
            if(e.name.includes('Exercitation')) {
                setCount(prev => prev++);
            }
            return <p key={e.id}>{e.name}</p>;
        }
        )}
        {count >= 2 && <div>This is shown when the count state is over 2</div>}
    </div>
)
```

동일한 코드 상에서 업데이트 된 부분은 오직 `if`문 이하의 `setCount`와 `count`가 2 이상일 경우 `div` 구문을 보여주는 것 뿐이다. 이것을 실행해보면 **infinite looping error**가 발생한다. 이는 return 구문 안에서 `useState`의 `set`을 사용해 업데이트가 무한히 일어나 발생하는 에러다. 만약, _엘리먼트가 나열되는 구간에서 상태를 업데이트해 특정 문구를 보여주거나 특정 컴포넌트를 보여줘야 한다면_ 다른 방법을 찾아봐야 할 것이다.

## 상태 업데이트에 있어서 useRef의 활용

내가 찾은 방법은 다음과 같다. `useState`의 경우, 엘리먼트가 있는 return 구문 이하에서는 사용할 수 없으므로 `useRef`의 리렌더링을 하지 않는 특성을 이용해보는 것이다. `useRef`는 `current` 내에 값을 담는다는 점에 참고하여 다음과 같이 코드를 짜보았다.

```js
const [data, setData] = useState<{id: number; name: string;}[]>([{id: 0, name: ''}]);
// const [count, setCount] = useState<number>(0);
const refCount = useRef(null);

useEffect(() => {
  getData();
}, [])

const getData = async () => {
    const res = await fetch("api/testData.json", {
        headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
        },
    });
    const data = await res.json();
    setData(data);
}

return (
    <div>
        {data?.map((e: {id: number; name: string;}, i: number) => {
            // 초기화(useEffect 내에서 초기화를 할 경우 추가 카운트가 되는 버그가 있어 array 진입 시 초기화)
            if (i === 0) {
                refCount.current = 0;
            }
            // Lorem Ipsum 구문에서 아래의 단어를 가졌을 경우 카운트한다.
            if(e.name.includes('Exercitation')) {
                // setCount(prev => prev++);
                refCount.current += 1;
            }
            return <p key={e.id}>{e.name}</p>;
        }
        )}
        {/* {count >= 2 && <div>This is shown when the count state is over 2</div>} */}
        {refCount.current >= 2 && <div>This is shown when the count state is over 2</div>} 
    </div>
)
```

위와 같은 형태로 코드를 짜게 되면 무한 업데이트 되는 버그는 발생하지 않는다. useState와 달리 useRef는 참조를 통해 최신 데이터를 유지하지만 리렌더링은 하지 않기 때문이다. 따라서 위의 코드에서는 비동기 데이터를 로드하고 화면에 뿌리는 과정에서 특정 문자를 가진 경우 카운팅해 그 카운팅된 숫자가 특정 수를 넘어갈 경우 관련된 컴포넌트를 보여주게 된다.

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
