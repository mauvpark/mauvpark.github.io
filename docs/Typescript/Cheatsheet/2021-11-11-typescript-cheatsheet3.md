---
layout: post
title: "3. Functions"
date: 2021-11-11 22:20:00 +0900
parent: Cheatsheet
grand_parent: Typescript
categories: typescript, cheat-sheet
nav_order: 2
comments: false
---

*2021-11-11 22:20 작성*

# Typescript 메모 3 (Functions)
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

본 내용은 [Typescript 공식 메뉴얼](https://www.typescriptlang.org/docs/)을 참고해 나오는 내용 중 필요한 내용만 발췌해 공부 목적으로 **번역 및 재구성**하였습니다.

## 1. Call Signatures

In JavaScript, functions can have properties in addition to being callable. However, **the function type expression syntax doesn’t allow for declaring properties**. If we want to describe something callable with properties, we can write a call signature in an object type:

```js
// Call Signatures 예시
// Function 내의 param에 타입 지정이 불가능하므로 Call Signature 사용
interface Foo {
	// x param의 타입 지정과 return 타입(number) 지정
	(x: string): number;
	(x: number): string;
	bar: Array<any>;
}

const foo: Foo = Object.assign(
	function (x: any) {
		if (typeof x === "string") {
			const t = parseInt(x);
			console.log(t);
			return t;
		} else {
			return x.toString();
		}
	},
	{
		bar: [],
	}
);

foo("111");

// 다른 방법 1: Object 형태의 타입 지정
type DescribableFunction = {
	description: string,
	sl: (someArg: number) => boolean,
};
function doSomething(fn: DescribableFunction) {
	console.log(fn.description + " returned " + fn.sl(6));
}

const obj = {
	description: "Hello",
	sl: (d: number) => {
		console.log(d);
		return true;
	},
};

doSomething(obj);

// 다른 방법 2: Function 단일 타입 지정
type test = (k: string) => void;
```

Note that the syntax is slightly different compared to a function type expression - use `:` between the parameter list and the return type rather than `=>`.

## 2. Construct Signatures

Construct signatures in interfaces are not implementable in classes; they're only for defining existing JS APIs that define a 'new'-able function. Here's an example involving interfaces `new` signatures that does work:

So an interface with a construct signature defines the signature of a constructor ! The constructor of your class that should comply with the signature defined in the interface(think of it as the constructor implements the interface). It is like a factory !

Here is a short snippet of code that tries to demonstrate the most common usage:

```js
interface ClassicInterface { // old school interface like in C#/Java
    method1();
    ...
    methodN();
}

interface Factory { //knows how to construct an object
    // NOTE: pay attention to the return type
    new (myNumberParam: number, myStringParam: string): ClassicInterface
}

class MyImplementation implements ClassicInterface {
    // The constructor looks like the signature described in Factory
    constructor(num: number, s: string) { } // obviously returns an instance of ClassicInterface
    method1() {}
    ...
    methodN() {}
}

class MyOtherImplementation implements ClassicInterface {
    // The constructor looks like the signature described in Factory
    constructor(n: number, s: string) { } // obviously returns an instance of ClassicInterface
    method1() {}
    ...
    methodN() {}
}

// And here is the polymorphism of construction
function instantiateClassicInterface(ctor: Factory, myNumberParam: number, myStringParam: string): ClassicInterface {
    return new ctor(myNumberParam, myStringParam);
}

// And this is how we do it
let iWantTheFirstImpl = instantiateClassicInterface(MyImplementation, 3.14, "smile");
let iWantTheSecondImpl = instantiateClassicInterface(MyOtherImplementation, 42, "vafli");
```

## 3. Generic Functions

In TypeScript, _generics_ are used when we want to describe a correspondence between two values.(*generics*는 두 값들 간의 통신 관계를 서술할 때 사용됨.) We do this by declaring _a type parameter_ in the function signature.

### Inference

Note that we didn’t have to specify `Type` in this sample. The type was _inferred_ - chosen automatically - by TypeScript.

```js
function map<Input, Output>(
	arr: Input[],
	func: (arg: Input) => Output
): Output[] {
	return arr.map(func);
}

// Input, Output의 타입을 지정하지 않아도 상호작용하며 자동 지정된다.
// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

### Constraints(특정 타입에서 사용되는 property를 사용해 타입을 제한하기)

Sometimes we want to relate two values, but can only operate on a certain subset of values. In this case, we can use a constraint to limit the kinds of types that a type parameter can accept.

Let’s write a function that returns the longer of two values. To do this, we need a `length` property that’s a number. **We constrain the type parameter to that type by writing an `extends` clause**:

```js
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// 🚫 Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
// 🚫 Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
```

### Guidelines for Writing Good Generic Functions(Generic Functions를 잘 쓰는 방법)

Writing generic functions is fun, and it can be easy to get carried away with type parameters. **Having too many type parameters or using constraints where they aren’t needed can make inference less successful, frustrating callers of your function.**

-   **Push Type Parameters Down**

```js
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}

// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

`firstElement2`’s inferred return type is `any` because TypeScript has to resolve the `arr[0]` expression using the constraint type, rather than “waiting” to resolve the element during a call.

> **_Rule: 가능하면 있는 그대로 type parameter를 쓰고 위와 같이 억지로 제한하는 것은 지양하라._**

-   **Use Fewer Type Parameters**

```js
// ⭕️
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

// ❌
function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

We’ve created a type parameter `Func` that doesn’t relate two values. That’s always a red flag, because it means callers wanting to specify type arguments have to manually specify an extra type argument for no reason. `Func` doesn’t do anything but make the function harder to read and reason about!(아무 이유없이 `Func`를 먼저 구체화해야 하므로 오히려 가독성을 흐리게 할 뿐 아니라 의미도 없음.)

> **_Rule: 되도록이면 항상 적은 type parameter를 쓰도록 하라._**

-   **Type Parameters Should Appear Twice**

가끔 우리는 굳이 generic을 쓰지 않아도 되는데 그 사실을 잊어버리고는 한다.

```js
// ⭕️
function greet(s: string) {
  console.log("Hello, " + s);
}

// ❌
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

greet("world");
```

type parameters는 *다수의 values 간 관계적 type을 지정하기 위한 것*임을 기억하라. 만약 type parameter가 function signature에서 단 한 번 쓰이는 경우라면 values 간 관계성이 형성되는 것이 아니므로 지양한다. (<u>e.g. 오직 string type만 들어오거나 values(params에 따라) 간 type 영향을 주지 않는다면 굳이 generic을 쓸 필요는 없을 것이다.</u>)

> **_Rule: type parameter가 오직 한 곳에서만 나타난다면 정말 사용할 필요가 있는 것인지 다시 고려해보라._**

## 4. Optional Parameters

### Optional Parameters in Callbacks

Optional Parameters를 배운 개발자들이 범하는 실수 중 하나는 다음과 같다.

```js
// ❌
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
	for (let i = 0; i < arr.length; i++) {
		// index가 Optional임에도 불구하고 콜백에서는 index를 확정적으로 넣는 경우
		callback(arr[i], i);
	}
}
```

보통 사람들이 Optioanl Parameter를 사용할 때는 `index?`를 선택적으로 적용하기를 원하기 때문일 것이다.

```js
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

이것이 정확히 의미하는 바는 *callback*이 한 개의 *argument*가 할당되었을 때에도 작동한다는 의미이다. 다시 말해, 다음과 같이 실행될 것이다.

```js
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
	for (let i = 0; i < arr.length; i++) {
		// I don't feel like providing the index today
		callback(arr[i]);
	}
}

// 다음과 같은 오류가 발생
myForEach([1, 2, 3], (a, i) => {
	console.log(i.toFixed());
	// 🚫 Object is possibly 'undefined'.
});
```

Javascript에서는 만약 당신이 parameters보다 더 많은 arguments를 할당하는 경우 여분의 arguments는 단순히 무시된다. TypeScript도 동일한 방식으로 동작한다.

> ❗️**_주의: callback에 function type을 작성할 때 특정 argument가 없어도 되는 function을 의도적으로 사용하는 것이 아니라면, 절대 optional parameter을 사용해서는 안 된다._**

## 5. Function Overloads

TypeScript에서 우리는 *overload signatures*라고 하는 것을 사용함으로써 *function*을 구체화할 수 있다. *overload signatures*를 적용하기 위해 복수의 *function signatures(보통 두 개 이상)*를 function의 body 앞부분에 적는다.

```js
// overload signatures 1
function makeDate(timestamp: number): Date;
// overload signatures 2
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
// 🚫 error 발생 - overload signatures가 없으면 조건에 모두 포함되므로 에러가 발생하지 않는다.
const d3 = makeDate(1, 3);
// 🚫 No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```

### Writing Good Overloads(Overloads를 잘 적기)

```js
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

이 function은 괜찮다. 우리는 이것으로 `string` 혹은 `arrays`를 적용할 수 있다. 그러나 우리는 `string`이나 `array`가 될 수도 있는 value에 대해서는 적용할 수 없다. 여기에서 TypeScript는 한 개의 overload만 다룰 수 있는 function만 해결할 수 있기 때문이다.

```js
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]); // error!
// No overload matches this call.
//   Overload 1 of 2, '(s: string): number', gave the following error.
//    Argument of type 'number[] | "hello"' is not assignable to parameter of type 'string'.
//       Type 'number[]' is not assignable to type 'string'.
//   Overload 2 of 2, '(arr: any[]): number', gave the following error.
//     Argument of type 'number[] | "hello"' is not assignable to parameter of type 'any[]'.
//       Type 'string' is not assignable to type 'any[]'.
```

두 overloads 모두 한 개의 argument와 같은 형태의 return type만을 허용하기에 우리는 대신에 overloaded 되지 않은 버전을 적용할 수 있다.

```js
function len(x: any[] | string) {
	return x.length;
}

len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]); // OK
```

> **_되도록이면 항상 overloads 보다 union types가 적용된 parameters를 선호하라._**

### Declaring `this` in a Function (Function 안에 `this` 선언하기)

TypeScript will infer what the this should be in a function via code flow analysis, for example in the following:

```js
const user = {
	id: 123,

	admin: false,
	becomeAdmin: function () {
		this.admin = true;
	},
};
```

Functions의 `user.becomeAdmin`은 `this`를 통해 바깥쪽에 위한 `user` object와 통신한다. JavaScript 설명서에 따르면 당신은 `this`로 불리우는 parameter를 가질 수 없다. 그래서 TypeScript는 `this` syntax를 function body 안에서 type 지정을 위한 공간으로 선언할 수 있다.

```js
interface User {
  id: number;
  admin: boolean;
}
declare const getDB: () => DB;
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});

// 🚫 Error!
const admins = db.filterUsers(() => this.admin);
// 🚫 The containing arrow function captures the global value of 'this'.
// 🚫 Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.
```

`function`을 실행했을 때 다른 object가 일반적으로 control하는 곳에서 흔히 사용하는 callback-style APIs 패턴이다. **`function`을 사용하지 않고 arrow functions를 사용하게 되면 error가 발생하니 주의한다.**

## 6. Other Types to Know About (알아야 할 다른 Types)

### A. `void`

`void`는 return value가 존재하지 않을 때 사용한다. JavaScript에서 아무 값도 return 하지 않는 function은 `undefined`를 return하는 것을 의미한다. **그러나 TypeScript에서 `void`와 `undefined`는 서로 다른 것을 의미한다.**

### B. `unknown`

`unknown` type은 _any_ value에 해당한다. 그러나 `any`보다 안전한데 그 이유는 `unknown`이 할당되는 모든 것은 비합법적(TypeScript 내에서 error 처리)이기 때문이다.

```js
function f1(a: any) {
	a.b(); // OK
}
function f2(a: unknown) {
	a.b();
	// 🚫 Object is of type 'unknown'.
}
```

`any` value를 이용하지 않고 functions body 내에서 어떤 value라도 수용할 수 있는 function을 만들 때 유용하게 쓸 수 있다. (경고 표시를 주므로 `any`보다 디버깅에 더 유용함.)

### C. `never`

```js
function fail(msg: string): never {
	throw new Error(msg);
}
```

`never` type은 _절대_ 관찰되지 않는 values를 나타낸다. return type에서 `never`는 function에서 예외가 발생했거나 프로그램이 종료됨을 의미한다.

또한 `never`는 union types 중 남은 것이 없다는 것을 의미하기도 한다.

```js
function fn(x: string | number) {
	if (typeof x === "string") {
		// do something
	} else if (typeof x === "number") {
		// do something else
	} else {
		x; // has type 'never'!
	}
}
```

## 7. Rest Parameters and Arguments

### Rest Arguments

```js
// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
// 🚫 A spread argument must either have a tuple type or be passed to a rest parameter.

// Solution
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

## 8. Assignability of Functions (Functions의 할당 가능성)

### Return type `void`

`void`의 return type은 드물지만 예상되는 행동을 생산할 수 있다.

맥락적 의미에서 `void`를 return type으로 지정하는 것은 `function`이 어떤 값도 return 하지 않는 것을 강제하지 않는다. 달리 말하면 이 `void`가 적용된 맥락적 function type이 실행 될 때 다른 _어떤_ 값이라도 return 될 수 있다. _하지만 무시될 뿐이다._

그리하여 다음의 `() => void` 실행은 유효하다.

```js
type voidFunc = () => void;

const f1: voidFunc = () => {
	return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
	return true;
};
```

그리고 아래의 functions와 같이 각각 변수에 할당되어도 실제 return 값이 존재함에도 `void`는 return 값을 무시하고 `void` 상태를 유지할 것이다.

다른 예시로 `Array.prototype.push`는 숫자를 return하고 `Array.prototype.forEach` method는 `void` 상태를 유지한다.

```js
const src = [1, 2, 3];
const dst = [0];

// 변수 t에 값을 할당해도 forEach는 void를 유지하므로 값은 undefined가 된다.
let t = src.forEach((el) => dst.push(el));
console.log(t); // undefined
```

한 가지 주의해야 할 특별한 경우가 있는데 일반적 function 정의가 `void` return type을 가지고 있는 경우, 그 `function`은 틀림없이 어떠한 return 값도 **없어야만 한다.**

```js
function f2(): void {
	// @ts-expect-error
	// ❗️어떤 값도 return 하지 않아야 함.
    return true;
}

const f3 = function (): void {
	// @ts-expect-error
    // ❗️어떤 값도 return 하지 않아야 함.
	return true;
};
```

## 출처(References)

[How does Construct signatures work?](https://newbedev.com/how-does-interfaces-with-construct-signatures-work)

{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}