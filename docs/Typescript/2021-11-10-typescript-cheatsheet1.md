---
layout: post
title: "Typescript Cheat Sheet 1"
date: 2021-11-10 16:45:00 +0900
parent: Typescript
categories: typescript, cheat-sheet
nav_order: 2
---

# Typescript 메모 1 (일반)

본 내용은 직접 작성한 내용이 아니며 [Typescript 공식 메뉴얼](https://www.typescriptlang.org/docs/)에 나오는 내용 중 필요한 내용만 발췌해 공부 목적으로 재구성하였습니다.

## 1. 객체 타입

JavaScript에서는 존재하지 않는 프로퍼티에 접근하였을 때, 런타임 오류가 발생하지 않고 undefined 값을 얻게 됩니다. 이 때문에 옵셔널 프로퍼티를 읽었을 때, 해당 값을 사용하기에 앞서 undefined인지 여부를 확인해야 합니다.

```js
function printName(obj: { first: string; last?: string }) {
  // 오류 - `obj.last`의 값이 제공되지 않는다면 프로그램이 멈추게 됩니다!
  console.log(obj.last.toUpperCase());
Object is possibly 'undefined'.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }

  // 최신 JavaScript 문법을 사용하였을 때 또 다른 안전한 코드
  console.log(obj.last?.toUpperCase());
```

## 2. 유니언 타입

TypeScript에서 유니언을 다룰 때는 해당 유니언 타입의 모든 멤버에 대하여 유효한 작업일 때에만 허용됩니다. 예를 들어 `string | number`라는 유니언 타입의 경우, string 타입에만 유효한 메서드는 사용할 수 없습니다.

```js
function printId(id: number | string) {
  console.log(id.toUpperCase());
Property 'toUpperCase' does not exist on type 'string | number'.
  Property 'toUpperCase' does not exist on type 'number'.
}
```

이를 해결하려면 코드상에서 유니언을 좁혀야 하는데, 이는 타입 표기가 없는 JavaScript에서 벌어지는 일과 동일합니다. *좁히기*란 TypeScript가 코드 구조를 바탕으로 어떤 값을 보다 구체적인 타입으로 추론할 수 있을 때 발생합니다.

예를 들어, TypeScript는 오직 string 값만이 typeof 연산의 결괏값으로 "string"을 가질 수 있다는 것을 알고 있습니다.

```js
// 예시 1
function printId(id: number | string) {
	if (typeof id === "string") {
		// 이 분기에서 id는 'string' 타입을 가집니다

		console.log(id.toUpperCase());
	} else {
		// 여기에서 id는 'number' 타입을 가집니다
		console.log(id);
	}
}

// 예시 2
function welcomePeople(x: string[] | string) {
	if (Array.isArray(x)) {
		// 여기에서 'x'는 'string[]' 타입입니다
		console.log("Hello, " + x.join(" and "));
	} else {
		// 여기에서 'x'는 'string' 타입입니다
		console.log("Welcome lone traveler " + x);
	}
}
```

else 분기 문에서는 별도 처리를 하지 않아도 된다는 점에 유의하시기 바랍니다. x의 타입이 string[]가 아니라면, x의 타입은 반드시 string일 것입니다.

때로는 유니언의 모든 멤버가 무언가 공통점을 가질 수도 있습니다. 에를 들어, 배열과 문자열은 둘 다 slice 메서드를 내장합니다. 유니언의 모든 멤버가 어떤 프로퍼티를 공통으로 가진다면, 좁히기 없이도 해당 프로퍼티를 사용할 수 있게 됩니다.

```js
// 반환 타입은 'number[] | string'으로 추론됩니다
function getFirstThree(x: number[] | string) {
	return x.slice(0, 3);
}
```

### 함수 선언 (참고용)

```js
declare function getInput(): string;
declare function sanitize(str: string): string;
// ---중간 생략---
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
	return sanitize(str);
}

// 보안 처리를 마친 입력을 생성
let userInput = sanitizeInput(getInput());

// 물론 새로운 문자열을 다시 대입할 수도 있습니다
userInput = "new input";
```

## 3. 인터페이스

```js
// 인터페이스 예시
interface Animal {
	name: string;
}

interface Bear extends Animal {
	honey: boolean;
}

interface Window {
	title: string;
}

interface Window {
	// 타입 추가
	ts: TypeScriptAPI;
}

// 타입 예시
type Animal = {
	name: string,
};

type Bear = Animal & {
	honey: Boolean,
};

interface Window {
	title: string;
}

interface Window {
	// ! Error: Duplicate identifier 'Window'.
	ts: TypeScriptAPI;
}
```

## 4. 타입 단언

예를 들어 코드상에서 document.getElementById가 사용되는 경우, TypeScript는 이때 HTMLElement 중에 *무언가*가 반환된다는 것만을 알 수 있는 반면에, 당신은 페이지 상에서 사용되는 ID로는 언제나 HTMLCanvasElement가 반환된다는 사실을 이미 알고 있을 수도 있습니다.

이런 경우, *타입 단언*을 사용하면 타입을 좀 더 구체적으로 명시할 수 있습니다.

```js
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

// 꺾쇠괄호를 사용하는 것 또한 (코드가 .tsx 파일이 아닌 경우) 가능
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

> _기억하세요: 타입 단언은 컴파일 시간에 제거되므로, 타입 단언에 관련된 검사는 런타임 중에 이루어지지 않습니다. 타입 단언이 틀렸더라도 예외가 발생하거나 null이 생성되지 않을 것입니다._

```js
const x = "hello" as number;
```

> 🚫 _Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first._

이 규칙이 때로는 지나치게 보수적으로 작용하여, 복잡하기는 하지만 유효할 수 있는 강제 변환이 허용되지 않기도 합니다. 이런 경우, 두 번의 단언을 사용할 수 있습니다. any(또는 이후에 소개할 unknown)로 우선 변환한 뒤, 그다음 원하는 타입으로 변환하면 됩니다.

```js
declare const expr: any;
type T = { a: 1; b: 2; c: 3 };
// ---중간 생략---
const a = (expr as any) as T;
```

## 5. 리터럴 타입

리터럴을 유니언과 함께 사용하면, 보다 유용한 개념들을 표현할 수 있게 됩니다. 예를 들어, 특정 종류의 값들만을 인자로 받을 수 있는 함수를 정의하는 경우가 있습니다.

```js
// 예시 1
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
🚫 Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.

// 예시 2
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

### 리터럴 추론

객체를 사용하여 변수를 초기화하면, TypeScript는 해당 객체의 프로퍼티는 이후에 그 값이 변화할 수 있다고 가정합니다. 예를 들어, 아래와 같은 코드를 작성하는 경우를 보겠습니다.

```js
declare const someCondition: boolean;
// ---중간 생략---
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

기존에 값이 0이었던 필드에 1을 대입하였을 때 TypeScript는 이를 오류로 간주하지 않습니다. 이를 달리 말하면 obj.counter는 반드시 number 타입을 가져야 하며, 0 리터럴 타입을 가질 수 없다는 의미입니다. 왜냐하면 타입은 읽기 및 쓰기 두 동작을 결정하는 데에 사용되기 때문입니다.

동일한 사항이 문자열에도 적용됩니다.

```js
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
🚫 Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```

위 예시에서 req.method는 string으로 추론되지, "GET"으로 추론되지 않습니다. req의 생성 시점과 handleRequest의 호출 시점 사이에도 얼마든지 코드 평가가 발생할 수 있고, 이때 req.method에 "GUESS"와 같은 새로운 문자열이 대입될 수도 있으므로, TypeScript는 위 코드에 오류가 있다고 판단합니다.

이러한 경우를 해결하는 데에는 두 가지 방법이 있습니다.

-   **a. 둘 중에 한 위치에 타입 단언을 추가하여 추론 방식을 변경할 수 있습니다.**

```js
// 수정 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// 수정 2
handleRequest(req.url, req.method as "GET");
```

> _<u>수정 1</u>은 ”req.method가 항상 리터럴 타입 "GET"이기를 의도하며, 이에 따라 해당 필드에 "GUESS"와 같은 값이 대입되는 경우를 미연에 방지하겠다”는 것을 의미합니다. <u>수정 2</u>는 “무슨 이유인지, req.method가 "GET"을 값으로 가진다는 사실을 알고 있다”는 것을 의미합니다._

-   **b. as const를 사용하여 객체 전체를 리터럴 타입으로 변환할 수 있습니다.**

```js
declare function handleRequest(url: string, method: "GET" | "POST"): void;
// ---중간 생략---
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

> _as const 접미사는 일반적인 const와 유사하게 작동하는데, 해당 객체의 모든 프로퍼티에 string 또는 number와 같은 보다 일반적인 타입이 아닌 리터럴 타입의 값이 대입되도록 보장합니다._

## 6. `null`과 `undefined`

JavaScript에는 빈 값 또는 초기화되지 않은 값을 가리키는 두 가지 원시값이 존재합니다. 바로 null과 undefined입니다.

TypeScript에는 각 값에 대응하는 동일한 이름의 두 가지 *타입*이 존재합니다. 각 타입의 동작 방식은 `strictNullChecks` 옵션의 설정 여부에 따라 달라집니다.

### `strictNullChecks`가 설정되지 않았을 때

strictNullChecks가 설정되지 않았다면, 어떤 값이 null 또는 undefined일 수 있더라도 해당 값에 평소와 같이 접근할 수 있으며, null과 undefined는 모든 타입의 변수에 대입될 수 있습니다. 이는 Null 검사를 하지 않는 언어(C#, Java 등)의 동작 방식과 유사합니다. Null 검사의 부재는 버그의 주요 원인이 되기도 합니다. **별다른 이유가 없다면, 코드 전반에 걸쳐 strictNullChecks 옵션을 설정하는 것을 항상 권장합니다.**

### `strictNullChecks`가 설정되었을 때

strictNullChecks가 설정되었다면, 어떤 값이 null 또는 undefined일 때, 해당 값과 함께 메서드 또는 프로퍼티를 사용하기에 앞서 해당 값을 테스트해야 합니다. 옵셔널 프로퍼티를 사용하기에 앞서 undefined 여부를 검사하는 것과 마찬가지로, *좁히기*를 통하여 null일 수 있는 값에 대한 검사를 수행할 수 있습니다.

```js
function doSomething(x: string | undefined) {
	if (x === undefined) {
		// 아무 것도 하지 않는다
	} else {
		console.log("Hello, " + x.toUpperCase());
	}
}
```

### Null 아님 단언 연산자 (접미사`!`)

TypeScript에서는 명시적인 검사를 하지 않고도 타입에서 `null`과 `undefined`를 제거할 수 있는 특별한 구문을 제공합니다. 표현식 뒤에 `!`를 작성하면 해당 값이 `null` 또는 `undefined`가 아니라고 타입 단언하는 것입니다.

```js
function liveDangerously(x?: number | undefined) {
  // 오류 없음
  console.log(x!.toFixed());
}
```

다른 타입 단언과 마찬가지로 이 구문은 코드의 런타임 동작을 변화시키지 않으므로, **`!` 연산자는 반드시 해당 값이 null 또는 undefined가 아닌 경우에만 사용해야 합니다.**

## 7. 열거형

열거형은 TypeScript가 JavaScript에 추가하는 기능으로, 어떤 값이 *이름이 있는 상수 집합*에 속한 값 중 하나일 수 있도록 제한하는 기능입니다. 대부분의 TypeScript 기능과 달리, 이 기능은 JavaScript에 타입 수준이 아닌, 언어와 런타임 수준에 추가되는 기능입니다. 따라서 열거형이 무엇인지는 알 필요가 있겠으나, 그 사용법을 명확하게 파악하지 않았다면 실제 사용은 보류하는 것이 좋습니다. 열거형에 대한 자세한 내용을 확인하려면 [열거형 문서](https://www.typescriptlang.org/ko/docs/handbook/enums.html)를 읽어보시기 바랍니다.

### **자주 사용되지 않는 원시형 타입**

**`bigint`** <br/>
ES2020 이후, 아주 큰 정수를 다루기 위한 원시 타입이 JavaScript에 추가되었습니다. 바로 `bigint`입니다.

```js
// BigInt 함수를 통하여 bigint 값을 생성
const oneHundred: bigint = BigInt(100);

// 리터럴 구문을 통하여 bigint 값을 생성
const anotherHundred: bigint = 100n;
```

**`symbol`** <br/>
symbol은 전역적으로 고유한 참조값을 생성하는 데에 사용할 수 있는 원시 타입이며, `Symbol()` 함수를 통하여 생성할 수 있습니다.

```js
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
This condition will always return 'false' since the types 'typeof firstName' and 'typeof secondName' have no overlap.
  // 절대로 일어날 수 없습니다
}
```
