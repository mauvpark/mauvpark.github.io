---
layout: post
title: useShallow를 이용한 zustand 최적화
description: useShallow hook을 이용해 컴포넌트 리렌더링을 관리해봅니다.
date: 2025-02-27 00:00:00 +0900
parent: React
categories: react,zustand,useShallow,shallow,optimization,rendering,nextjs
nav_order: 3
---

_2025-02-27_

# useShallow를 이용한 zustand 최적화

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

_2025-03-04 NEXT JS 환경 반영_

## Intro

Zustand는 작은 용량과 빠른 속도 그리고 확장성이 용이한 상태 관리 라이브러리입니다.

상태 관리 라이브러리를 통해 컴포넌트를 개발할 때, 마주치는 가장 기본적인 문제는 상태가 변했을 때, 상태 관리 라이브러리를 사용하고 있는 컴포넌트들이 모두 리렌더링이 이루어진다는 것입니다.

렌더링을 할 때, 렌더링 대상이 되는 컴포넌트들이 단순 입력과 같은 것들이라면 매 렌더링마다 계산해야 하는 사이즈가 작기 때문에 문제가 없습니다.

하지만 고용량 이미지, 큰 파일들이 들어가 있는 복잡한 컴포넌트들의 상태를 관리하고자 한다면 브라우저가 리렌더링 시에 계산해야 하는 비용이 너무도 크기 때문에 화면이 멈출 수 있습니다.

극단적인 예를 들자면, 동영상과 고용량 이미지가 들어간 페이지에 `textarea`가 있고 그 안에서, 문자를 입력한다고 합시다.

그럼 이 페이지에서 textarea에 문자를 하나 입력할 때 마다 페이지의 고용량 이미지와 동영상을 다시 그려내야 하기 때문에 화면 응답이 극단적으로 느려지게 되고, 고객은 결국 페이지를 이탈하게 됩니다.

그렇다면 이 문제는 어떻게 해결하면 좋을까요?

## useShallow

Zustand 5.x.x 버전 이후부터는 `useShallow`라는 hook이 제공됩니다. `useShallow` hook을 통해 업데이트 되지 않은 컴포넌트는 리렌더링이 되지 않도록 할 수 있습니다.

아래의 store가 있다고 생각해봅시다.

[참고](https://zustand.docs.pmnd.rs/guides/slices-pattern)

### NEXT JS를 사용하는 경우 [참고](https://github.com/pmndrs/zustand/blob/main/docs/guides/nextjs.md)

```javascript
// store.ts
export const createFishSlice = (set) => ({
  fishes: 0,
  addFish: () => set((state) => ({ fishes: state.fishes + 1 })),
})

export const createBearSlice = (set) => ({
  bears: 0,
  addBear: () => set((state) => ({ bears: state.bears + 1 })),
  eatFish: () => set((state) => ({ fishes: state.fishes - 1 })),
})

expot const createBoundStore = () => createStore((...a) => ({
    ...createBearSlice(...a),
    ...createFishSlice(...a),
}))
```

```javascript
// provider.tsx
import { useStore } from 'zustand'
import {createBoundStore} from './store';

export const BoundStoreContext = createContext(undefined);

export const BoundStoreProvider = ({children}) => {
    // 타겟 위치마다 새롭게 생성
    const storeRef = useRef(null);
    if (!storeRef.current) {
        storeRef.current = createBoundStore();
    }

    return (
        <BoundStoreContext.Provider value={storeRef.current}>
            {children}
        </BoundStoreContext.Provider>
    )
}

export const useBoundStore = (selector) => {
    const boundStoreContext = useContext(BoundStoreContext);

    // Provider와 같이 사용하지 않은 경우 에러 발생
    if (!boundStoreContext) {
        throw new Error("useBoundStore는 BoundStoreProvider와 함께 사용해야 합니다.");
    }

    return useStore(boundStoreContext, selector);
}
```

NEXT JS에서 위와 같이 설정하지 않고 사용하게 되면, 페이지와 페이지 간 상태가 공유되어 예상치 못한 버그가 발생할 수 있습니다.

그러므로 NEXT JS를 사용할 때는 이러한 버그를 예방하기 위해서, 상기의 방식처럼 매 route request 마다 새로운 store를 생성하는 전략을 취할 수 있습니다.

### 다른 프레임워크를 사용하는 경우

```javascript
import { create } from 'zustand';

export const createFishSlice = (set) => ({
  fishes: 0,
  addFish: () => set((state) => ({ fishes: state.fishes + 1 })),
})

export const createBearSlice = (set) => ({
  bears: 0,
  addBear: () => set((state) => ({ bears: state.bears + 1 })),
  eatFish: () => set((state) => ({ fishes: state.fishes - 1 })),
})

export const useBoundStore = create((...a) => ({
  ...createBearSlice(...a),
  ...createFishSlice(...a),
}))
```

그리고 각 컴포넌트에서 상태 업데이트를 다음과 같이 한다고 가정하겠습니다.

```javascript
import {useBoundStore} from "@/store";

const BearHouse = () => {
    const {bears, addBear, eatFish} = useBoundStore((state) => ({bears: state.bears, addBear: state.addBear, eatFish: state.eatFish}));

    return (
        <div>
            <h1>My bear family</h1>
            <p>Number of bears: {bears}</p>
            <button onClick={() => addBear()}>Add bear</button>
        </div>
    )
}

const Sea = () => {
    const {fishes, addFish} = useBoundStore((state) => ({fishes: state.fishes, addFish: state.addFish}));

    return (
        <div>
            <h1>Sea</h1>
            <p>Number of fishes: {fishes}</p>
            <button onClick={() => addFish()}>Add fish</button>
        </div>
    )
}

// NEXT JS
export default const App = () => {
    return (
        <>
            <BoundStoreProvider>
                <BearHouse />
                <Sea />
            </BoundStoreProvider>
        </>
    )
}

// 그 외 프레임워크
export default const App = () => {
    return (
        <>
            <BearHouse />
            <Sea />
        </>
    )
}
```

위 예제에서 bear를 추가하거나 fish를 추가하게 되면 각 개별 컴포넌트만 업데이트 되는 것이 아니라 `useBoundStore`가 있는 모든 컴포넌트가 렌더링 됩니다.

여기에 `useShallow`를 추가하게 되면 업데이트 되는 영역만 렌더링이 될 수 있도록 할 수 있어 최적화를 할 수 있습니다.

```javascript
import {useBoundStore} from "@/store";
import {useShallow} from "zustand/shallow";

const BearHouse = () => {
    const {bears, addBear, eatFish} = useBoundStore(useShallow((state) => ({bears: state.bears, addBear: state.addBear, eatFish: state.eatFish})));

    return (
        <div>
            <h1>My bear family</h1>
            <p>Number of bears: {bears}</p>
            <button onClick={() => addBear()}>Add bear</button>
        </div>
    )
}

const Sea = () => {
    const {fishes, addFish} = useBoundStore(useShallow((state) => ({fishes: state.fishes, addFish: state.addFish})));

    return (
        <div>
            <h1>Sea</h1>
            <p>Number of fishes: {fishes}</p>
            <button onClick={() => addFish()}>Add fish</button>
        </div>
    )
}

// NEXT JS
export default const App = () => {
    return (
        <>
            <BoundStoreProvider>
                <BearHouse />
                <Sea />
            </BoundStoreProvider>
        </>
    )
}

// 그 외 프레임워크
export default const App = () => {
    return (
        <>
            <BearHouse />
            <Sea />
        </>
    )
}
```

## useShallow의 정체

그렇다면 기존에 사용하고 있던 zustand를 강제로 버전을 올려 사용해야 할까에 대한 고민이 생길 수 있습니다. `useShallow`는 어떻게 기존 변경되지 않은 컴포넌트가 memoization 되도록 할 수 있을까요?

소스를 봅시다.

```typescript
import React from 'react';
import { shallow } from '../vanilla/shallow.ts';

export function useShallow<S, U>(selector: (state: S) => U): (state: S) => U {
  // 1. 객체를 저장하는 ref입니다.
  const prev = React.useRef<U>(undefined);
  return (state) => {
    // 2. 'next'의 값은 custom store hook에서 선택된 반환값입니다.(e.g. `{fishes: state.fishes, addFish: state.addFish}`)
    const next = selector(state);
    // 3. Zustand의 shallow 비교 함수를 통해 렌더링에 사용된 저장값과 이번 최신값이 같은지 여부를 검사합니다.
    return shallow(prev.current, next)
    // 4. 저장된 값과 최신값이 같을 경우, 주소가 같은 과거 객체를 반환하여 렌더링을 방지합니다.
      ? (prev.current as U)
    // 저장된 값과 최산값이 다를 경우, 주소가 다른 새로운 객체를 반환하여 렌더링을 시행합니다.
      : (prev.current = next)
  }
}
```

위의 코드에서 사용된 `shallow`와 `Object.is()`의 차이는 [여기](https://zustand.docs.pmnd.rs/apis/shallow#shallow)에서 확인 가능합니다.

## 결론

`useShallow` hook을 이용해 최적화를 하는 방법을 알아봤습니다. `useShallow`를 사용하지 못하는 저버전의 zustand를 사용하는 프로젝트는 상기의 코드를 공통 hook으로 만들어 사용해볼 수 있겠습니다.

추가로, Next js 테스트 중 `useShallow`를 사용하는 store에서 일부에만 `useShallow`만 적용하는 경우 무한 리렌더링이 발생하는 버그가 있으니 `useShallow`를 적용할 때는 관련 store에 전부 적용해주도록 합니다.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>