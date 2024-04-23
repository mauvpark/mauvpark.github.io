---
layout: post
title: useEffect cleanup 함수와 dependency 정리
date: 2024-04-23 00:00:00 +0900
parent: React
categories: react, useEffect, object.is
nav_order: 3
comments: false
---

_2024-04-23 작성_

# useEffect cleanup 함수와 dependency 정리

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

링크: [React-useEffect](https://react.dev/reference/react/useEffect)

useEffect의 기본적인 사용방법은 생략합니다.

## useEffect cleanup 함수

- cleanup 함수는 관련된 setup 함수가 있는 useEffect 내에서 실행되어야 합니다.
- useEffect cleanup 함수는 컴포넌트가 DOM에서 사라진 후 사라집니다.
- dependency에 따라 리렌더링이 발생하면 앞서 실행된 setup 함수에 대한 cleanup 함수를 실행한 후, 새로운 value에 대한 setup 함수를 실행합니다.

## useEffect dependency

- useEffect의 dependency 값 비교 방식은 `Object.is` 메서드를 사용해 비교합니다.
- 컴포넌트 내에 정의된 object를 dependency에 추가하는 경우, 렌더링이 일어날 때마다 새로운 object를 생성하므로 매번 새로운 object로 인식하게 되고, 이는 불필요한 렌더링이 일어나게 합니다.
- object와 function을 useEffect에서 사용하고자 하는 경우, memoizing을 통해 불필요한 렌더링이 발생하지 않도록 방지하거나 useEffect 내에서 object와 function을 선언해 불필요한 렌더링이 발생하지 않도록 합니다.

## 서버 컴포넌트와 클라이언트 컴포넌트의 Hydration

서버 컴포넌트에서 작성된 initial HTML은 클라이언트 컴포넌트의 HTML과 동일해야 합니다. 하지만 localStorage와 같은 웹 API를 사용해야 하는 경우, 클라이언트에서 불일치 문제가 발생하게 됩니다. 이를 해결하기 위해서 useEffect를 활용할 수 있습니다.

```javascript
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    return <div>클라이언트 전용 JSX</div>;
  } else {
    return <div>초기 JSX</div>;
  }
}
```

해당 방법은 조심스럽게 사용해야 하고 필요한 곳에 최소로 사용되어야 합니다. 인터넷이 느린 사용자의 경우 초기 HTML을 몇 초 동안 바라보다가 갑자기 바뀐 HTML을 봐야 한다면 좋지 않은 경험일 것입니다.

## `Object.is()`의 비교 결과값

링크: [Mozilla-Object.is()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)

### `Object.is()` 값이 같은 경우

- undefined
- null
- boolean
- string
- object(메모리 상 같은 object를 참조하는)
- BigInt(같은 숫자값을 가지는)
- Symbol(같은 symbol 값을 참조하는)
- number(같은 `+0`, 같은 `-0`, 같은 `NaN`, 나머지 같은 숫자)

### `Object.is()`와 `==` 연산자 그리고 `===` 연산자와의 차이점

`Object.is()`는 `==` 연산자와 같지 않습니다. `==` 연산자는 양쪽이 설령 타입이 같지 않더라도 연산을 강제하는데(`"" == false` => `true`), `Object.is()`는 두 값을 강제 연산하지 않습니다.

`Object.is()`는 `===` 연산자와도 동일하지 않습니다. `===`은 `-0`과 `+0`을 같은 것으로 연산하지만 `Object.is()`는 그렇지 않습니다. `===`은 두 개의 `NaN`을 같지 않은 것으로 연산하지만 `Object.is()`는 같은 것으로 여깁니다.

### `Object.is()` 참고

링크: [Mozilla-example](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#using_object.is)

```javascript
// Case 1: Evaluation result is the same as using ===
Object.is(25, 25); // true
Object.is("foo", "foo"); // true
Object.is("foo", "bar"); // false
Object.is(null, null); // true
Object.is(undefined, undefined); // true
Object.is(window, window); // true
Object.is([], []); // false
const foo = { a: 1 };
const bar = { a: 1 };
const sameFoo = foo;
Object.is(foo, foo); // true
Object.is(foo, bar); // false
Object.is(foo, sameFoo); // true

// Case 2: Signed zero
Object.is(0, -0); // false
Object.is(+0, -0); // false
Object.is(-0, -0); // true

// Case 3: NaN
Object.is(NaN, 0 / 0); // true
Object.is(NaN, Number.NaN); // true
```
