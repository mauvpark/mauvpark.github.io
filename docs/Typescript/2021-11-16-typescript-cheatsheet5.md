---
layout: post
title: "5. Type Manipulation"
date: 2021-11-16 17:00:00 +0900
parent: Typescript
categories: typescript, cheat-sheet
nav_order: 2
---

# Typescript 메모 5 (Type Manipulation)

본 내용은 [Typescript 공식 메뉴얼](https://www.typescriptlang.org/docs/)을 참고해 나오는 내용 중 필요한 내용만 발췌해 공부 목적으로 **번역 및 재구성**하였습니다.

## 1. Creating Types from Types (Types에서 Types를 만들기 - 도입)

TypeScript가 대단한 이유는 TypeScript가 다른 types의 관점에서 types를 표현하는 것을 용인하기 때문이다.

이 idea의 가장 간단한 형태는 *generics*이다. 사실 우리는 다양한 종류의 *type operators*를 사용할 수 있다. 이것은 또한 우리가 이미 가지고 있는 *values*의 관점에서 types를 표현할 수 있게 한다.

다양한 형태의 type operators를 통합함으로써, 우리는 복잡한 작업과 values를 간단하고 유지보수 가능한 형태로 표현할 수 있게 된다.

-   [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html) - parameters를 받는 types
-   [Keyof Type Operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html) - `keyof` operator를 사용하여 새로운 types를 생성
-   [Typeof Type Operator](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html) - `typeof`를 사용하여 새로운 types를 생성
-   [Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) - `Type['a']` syntax를 사용하여 type의 부분 집합에 접근
-   [Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html) - Type system에서 if statements 처럼 작동하는 types
-   [Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html) - 이미 존재하는 type에서 각각의 property를 mapping해 types를 생성
-   [Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) - template literal strings를 거쳐 properties를 바꾸는 mapped types

## 2. Generics

Software engineering의 가장 중요한 부분 중 하나는 잘 정의되고 변함없는 APIs를 가지고 있을 뿐만 아니라 재사용이 가능한 components를 만들어내는 것이다. 오늘의 data와 내일의 data에 공들일 수 있는 components는 당신에게 큰 software systems를 만들어내는데 있어 가장 유연한 능력을 줄 것이다.

C#과 Java와 같은 언어에서, 재사용 가능한 components를 생성하기 위한 toolbox 안의 주요 tools 중 하나는 *generics*이다. *Generics*는 한 가지 type보다는 여러가지 다양한 types를 적용해 작업할 수 있는 component를 생성할 수 있다. 이것은 사용자들이 이러한 형태의 components를 소비하게하고 그들 스스로의 types를 적용할 수 있도록 한다.

### A. Generic Types

Generic functions의 type은 type parameters가 첫번째에 list된 non-generic functions와 같으며 function 선언과 비슷하다.

```js
function identity<Type>(arg: Type): Type {
	return arg;
}

// Type 지정: <Type>(arg: Type) => Type
let myIdentity: <Type>(arg: Type) => Type = identity;
```

우리는 type에서 generic type parameter를 Type 변수들의 수만큼 그리고 얼만큼 type 변수들이 사용되었는지에 따라 다른 이름으로 지정할 수도 있다.

```js
function identity<Type>(arg: Type): Type {
	return arg;
}

let myIdentity: <Input>(arg: Input) => Input = identity;
```

우리는 또한 generic types를 ojbect literal type의 call signature 형태로도 쓸 수 있다.

```js
function identity<Type>(arg: Type): Type {
	return arg;
}

let myIdentity: { <Type>(arg: Type): Type } = identity;
```

상기의 예시는 우리로 하여금 첫번째 generic interface를 쓰도록 이끈다. 직전의 예시에서 object literal을 interface로 바꾸어 보자.

```js
interface GenericIdentityFn {
	<Type>(arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
	return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

비슷한 예시로 우리는 generic parameter가 전체 interface의 parameter가 되기를 바랄 수도 있다. 이것은 우리로 하여금 어떤 type을 전체 generic type으로 지정할 것인지 직관하게 한다(e.g. `Dictionary`라고 단순히 적는 것보다 `Dictionary<string>`라고 명명).

```js
interface GenericIdentityFn<Type> {
	(arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
	return arg;
}

// number 형태로 전체 generic over 됨.
let myIdentity: GenericIdentityFn<number> = identity;
```

### B. Generic Classes

Generic class는 generic interface와 비슷한 형태를 가지고 있다. Generic classes는 generic type parameter list를 꺾쇠괄호(`<>`)를 통해 표현한다.

```js
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

`GenericNumber` class의 꽤 일반적인 사용방식에서 당신은 오직 `number` type을 사용하는 것 말고는 어느 것도 제한하지 않는다는 것을 눈치챘을 수도 있다. 우리는 대신 `string`을 사용할 수도 있고 혹은 더 복잡한 objects를 사용할 수도 있다.

```js
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

Interface와 마찬가지로 특정 class에 type parameter를 넣는 것은 class의 모든 properties를 같은 type 형태로 사용할 수 있다는 확신을 준다.

**Class는 type에 있어 두 가지 면을 가지고 있다: static side와 instance side. Generic classes는 그들의 static side보다는 instance side를 전체 generic화 한다. 그렇기에 classes 작업을 할 때, static members는 class의 type parameter로서 사용할 수 없다.**

### C. Generic Constraints

Any와 all types의 형태로 작업하는 것보다 우리는 어느 함수가 `.length` property*도* 가지고 있는 any 그리고 all types로 제한하여 작업을 하고 싶다고 해보자. Type이 이 member를 가지고 있는 한 적어도 이 member를 가지고 있어야 함을 요구하는 형태로 우리는 허용할 것이다. 그렇게 하기 위해서 우리는 `Type`이 될 수 있는 것에 제한을 어떤 형태로 걸 것인지 정렬해야만 한다.

그렇게 하기 위해서 우리는 우리의 제한을 서술할 interface를 생성할 것이다. 여기에서 우리는 `.length`라는 property를 가진 interface를 생성할 것이고 **이 interface를 `extends` 키워드를 사용하여 제한을 나타낼 것이다.**

```js
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  console.log(arg);
  return arg;
}
```

Generic function은 이제 제한되었기에 더이상 any와 all types의 범위에서 작동하지 않을 것이다.

```js
// ❌
loggingIdentity(3);
// 🚫 Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.

// ⭕️
loggingIdentity({ length: 10, value: 3 });
```

### D. Using Type Parameters in Generic Constraints (Generic 제한에서 Type Parameters 사용하기)

당신은 다른 type parameter에 의해 제한된 type parameter을 선언할 수 있다. 예를 들어, 이름이 주어진 object에서 property를 얻고 싶다고 해보자. 우리는 잘못하여 `obj`에 존재하지 않는 property를 가져오고 싶지 않으므로 두 types 간에 제한을 걸 것이다:

```js
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m");
// 🚫 Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
```

### E. Using Class Types in Generics(Generics에서 Class Types 사용하기)

TypeScript에서 generics를 이용해 factories를 생성할 때, constructor functions로 class types를 참조할 필요가 있다. 예를 들면,

```js
function create<Type>(c: { new(): Type }): Type {
	return new c();
}
```

좀 더 진보된 예시는 constructor function과 class types의 instance side 사이에서 관계를 제한하고 의미를 나타내기 위해 prototype property를 사용한다.

```js
class BeeKeeper {
  hasMask: boolean = true;
}

class ZooKeeper {
  nametag: string = "Mikle";
}

class Animal {
  numLegs: number = 4;
}

class Bee extends Animal {
  keeper: BeeKeeper = new BeeKeeper();
}

class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

## 3. Keyof Type Operator

### A. The `keyof` type operator

`keyof` operator는 object type을 취하고 key로서 string 혹은 numeric literal union을 생산한다. 다음의 `type P`는 `"x" | "y"`와 같다:

```js
type Point = { x: number; y: number };
type P = keyof Point;

// ❗️type P = keyof Point
```

만약 type이 `string` 혹은 `number` index signature를 가지고 있으면, `keyof`는 그 types를 다음과 같이 표기할 것이다:

```js
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;

// ❗️type A = number

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;

// ❗️type M = string | number
```

**위 예시에서, `M`은 `string | number`이다 - 이것은 JavaScript object keys는 항상 `string`으로 강제되기 때문인데 따라서 `obj[0]`는 항상 `obj["0"]과 같게 된다.**

**`keyof` types는 mapped types와 결합되었을 때 특히 유용하다.**

## 4. Typeof Type Operator

```js
function f() {
	return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;

// ❗️type P = { x: number, y: number };
```

### 한계

TypeScript는 의도적으로 `typeof`의 사용을 제한하고 있다.

구체적으로 identifiers(i.e. 변수 명칭) 혹은 그들의 properties에서만 사용이 가능하다. 이것은 실제로 실행되지 않음에도 실행되고 있다고 착각하는 혼란을 피하기 위함이다.

```js
// Meant to use = ReturnType<typeof msgbox>
// ❌
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
// ',' expected.
```

## 5. Indexed Access Types

우리는 *indexed access type*을 다른 type에 대해 특정 property를 찾는데 사용할 수 있다.

```js
type Person = { age: number, name: string, alive: boolean };
type Age = Person["age"];

// ❗️type Age = number
```

Indexing type은 그 자체로 type이어서 union을 사용할 수도, `keyof`를 사용할 수도 혹은 다른 types를 사용할 수도 있다:

```js
type I1 = Person["age" | "name"];

// ❗️type I1 = string | number

type I2 = Person[keyof Person];

// ❗️type I2 = string | number | boolean

type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];

// ❗️type I3 = string | boolean
```

당신이 만약 존재하지 않는 property를 색인하려고 하면 error를 보게 될 것이다.

```js
// ❌
type I1 = Person["alve"];
// 🚫 Property 'alve' does not exist on type 'Person'.
```

임시 type으로 색인하는 예로 array elements의 type을 얻기 위해 `number`를 사용하는 것이다. 우리는 array literal의 element type을 알맞게 붙잡기 위해 `typeof`와 결합하여 통합할 수 있다.

```js
const MyArray = [
	{ name: "Alice", age: 15 },
	{ name: "Bob", age: 23 },
	{ name: "Eve", age: 38 },
];

type Person = (typeof MyArray)[number];

// ❗️type Person = { name: string; age: number; }
type Age = (typeof MyArray)[number]["age"];

// ❗️type Age = number
// Or
type Age2 = Person["age"];

// ❗️type Age2 = number
```

당신은 색인을 할 때에만 types를 사용할 수 있는데 이것은 당신이 `const`를 변수 참조했을 경우에는 사용할 수 없다는 뜻이다.

```js
// ❌
const key = "age";
type Age = Person[key];
// 🚫 Type 'key' cannot be used as an index type.
// 🚫 'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?

// ⭕️
type key = "age";
type Age = Person[key];
```

## 6. Conditional Types

가장 유용한 programs의 심부에서 우리는 input(입력)에 기반한 결정을 내려야 한다. JavaScript programs도 차이가 없지만 values를 쉽게 검사할 수 있다는 사실과 그러한 결정은 inputs의 types에 기반한다. *Conditional types*는 inputs와 outputs의 types 간 관계를 기술하는 것을 돕는다.

```js
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string;

// ❗️type Example1 = number

type Example2 = RegExp extends Animal ? number : string;

// ❗️type Example2 = string
```

Conditional types는 JavaScript에서 다소 조건 표현과 같은 형태이다(`condition ? trueExpression : falseExpressjion`).

```js
SomeType extends OtherType ? TrueType : FalseType;
```

**`extends`의 왼쪽에 있는 type이 오른쪽에 있는 type에(`extends` 기준: 여기에서는 `OtherType`) 할당 가능할 때 첫 번째 branch("true" branch)를 얻을 것이다. 그렇지 않으면 당신은 좀 더 뒷쪽에 위치한 branch("false" branch)를 얻게 된다.**

위의 예시와 같이 conditional types는 즉각적으로 유용한 것처럼 보이지는 않는다. 우리는 `Dog extends Animal`에 해당하는지 아닌지를 우리 스스로 말할 수 있고 `number` 혹은 `string`을 선택할 수 있다! 하지만 conditional types의 힘은 generics와 함께 사용할 때 나타난다.

다음의 `createLabel` function을 살펴보자.

```js
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

`createLabel`의 overloads는 inputs의 types에 기초한 선택을 하게 하는 하나의 JavaScript function을 기술한다. 몇 가지를 메모해 두자:

> 1. 만약 library가 같은 종류의 선택을 넘어서고 API의 범위 밖이라면 번잡해질 수 있다.
> 2. 우리는 세 가지의 overloads를 생성해야 한다: 하나는 우리가 확신할 수 있는 type(하나는 `string` 그리고 다른 하나는 `number`), 그리고 가장 일반적인 경우(`string | number`). `createLabel`이 관리 가능한 모든 새로운 type을 위해 overloads의 수는 기하급수적으로 증가하게 된다.

대신에, 우리는 conditional type으로 logic을 짤 수 있다.

```js
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

우리의 overloads는 더이상의 overloads 없이, 하나의 function에서 간소화 된 상태로 위에서 작성한 conditional type을 사용할 수 있다.

```js
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}

let a = createLabel("typescript");

// ❗️let a: NameLabel

let b = createLabel(2.8);

// ❗️let b: IdLabel

let c = createLabel(Math.random() ? "hello" : 42);
// ❗️let c: NameLabel | IdLabel
```

### A. Conditional Type Constraints

종종 conditional type에서 (type을) 확인하는 것은 우리에게 새로운 정보를 제공할 것이다. Type guards를 지닌 Narrowing(좁히기)과 같이 우리에게 좀 더 구체적인 type을 줄 수 있다. Conditional type의 true branch(조건문에서의 "true" 영역)는 우리가 가까이서 확인하는 type에 의해 더 나아간 형태로 generics를 제한할 것이다.

다음 예시를 보자:

```js
type MessageOf<T> = T["message"];
// 🚫 Type '"message"' cannot be used to index type 'T'.
```

이 예시에서 TypeScript는 error가 나는데 `T`는 `message`라고 불리는 property가 존재하는지 몰랐기 때문에 발생한다. 우리는 `T`를 제한할 수 있고 TypeScript는 더 이상 error를 발생시키지 않을 것이다:

```js
// message의 type은 정해지지 않았지만 존재함을 미리 선언함.
type MessageOf<T extends { message: unknown }> = T["message"];

interface Email {
  message: string;
}

type EmailMessageContents = MessageOf<Email>;

// ❗️type EmailMessageContents = string
```

하지만, 만약 우리가 `MessageOf`를 any type으로 설정하고 `message` property가 사용 불가능할 때 `never`을 default로 설정하길 원한다면 어떨까? 우리는 제한을 conditional type 형태로 기술하는 것으로 기능을 구현할 수 있다.

```js
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>;

// ❗️type EmailMessageContents = string

type DogMessageContents = MessageOf<Dog>;

// ❗️type DogMessageContents = never
```

True branch 안에서 TypeScript는 `T`가 `message` property를 가지고 있을 것이라는 것을 알게 된다.

다른 예시를 보자, 우리는 array types를 array types의 element types로 쪼개는 `Flatten`이라는 type을 쓸 수도 있다. 하지만 False의 경우에는 그대로 둔다.

```js
type Flatten<T> = T extends any[] ? T[number] : T;

// Extracts out the element type.
type Str = Flatten<string[]>;

// ❗️type Str = string

// Leaves the type alone.
type Num = Flatten<number>;

// ❗️type Num = number
```

`Flatten`이 array type으로 주어졌을 때, `string[]`의 element type으로 꺼내기 위해 `Flatten`은 `number`로 색인된 접근을 한다. 그렇지 않으면 주어진 상태의 type을 반환한다.

### B. Inferring Within Conditional Types

우리는 방금까지 conditional types를 이용해 제한을 적용하고 types를 추출하는 방법에 관해 살펴보았다. 이것은 conditional types를 좀 더 쉽게 하는 일반적 방법으로 이어지게 된다.

Conditional types는 우리가 비교하는 types로부터 추론하는 방법을 제공하는데 true branch의 옆에서 `infer` 키워드를 사용함으로써 실행 가능하다. 예를 들어, `Flatten`의 element type을 indexed access type을 통해 _수동으로_ 꺼내는 대신에 *infer(추론)*을 하고 있다고 해보자:

```js
// Array element의 type을 추론형으로 넣고 만약 Array 형태가 오면 element type을 추출해 true branch에 반환한다.
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
// Array 내부의 element type은 string이므로 string이 true branch에 위치한다.
type Str = Flatten<string[]>;
// Array 내부의 element type은 number이므로 number가 true branch에 위치한다
type Num = Flatten<number[]>;
// False branch에 number가 Type으로 할당 된다.
type SimpleNum = Flatten<number>;
```

여기에서 우리는 true branch에서 `T`의 element type을 되찾는 방법을 구체화하기 보다는 `Item`이라고 명명한 새로운 generic type 변수를 선언적으로 접하게 하기 위해 `infer` 키워드를 사용하였다. 이것은 우리가 흥미있어 하는 types의 구조를 면밀히 살피고 파내는 것에서 자유롭게 한다.

우리는 `infer` 키워드를 이용해 유용한 helper type aliases를 기술(write)할 수 있다. 예를 들어, 간단한 경우에 우리는 function types로부터 return type을 추출해낼 수 있다.

```js
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;

type Num = GetReturnType<() => number>;

// ❗️type Num = number

type Str = GetReturnType<(x: string) => string>;

// ❗️type Str = string

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;

// ❗️type Bools = boolean[]
```

Overloaded function의 type처럼 다수의 call signatures를 지닌 type으로부터 추론할 때, 추론은 짐작컨대 가장 관대한 catch-all case의 *last signature*로부터 만들어질 것이다. argument types의 list에 기초한 overload 해결을 행하는 것은 불가능하다.

```js
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;

type T1 = ReturnType<typeof stringOrNum>;

// ❗️type T1 = string | number
```

### C. Distributive Conditional Types

Conditional types가 generic type에 적용됐을 때, union type 형태가 입력되면 *분배적*으로 적용된다. 다음 예시를 보자:

```js
type ToArray<Type> = Type extends any ? Type[] : never;
```

만약 union type을 `ToArray`에 꽂게 되면, conditional type은 union 각각의 member에 적용될 것이다.

```js
type ToArray<Type> = Type extends any ? Type[] : never;

type StrArrOrNumArr = ToArray<string | number>;

// ❗️type StrArrOrNumArr = string[] | number[]
```

일반적으로 분배성은 훌륭한 행위이다. 만약, 이것을 피하고 싶다면 당신은 각각의 side를 대괄호와 함께 `extends` 키워드로 둘러주면 된다.

```js
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

// 'StrArrOrNumArr' is no longer a union.
type StrArrOrNumArr = ToArrayNonDist<string | number>;

// ❗️type StrArrOrNumArr = (string | number)[]
```

## 7. Mapped Types

당신이 작업을 반복하고 싶지 않을 때, 때때로 type은 다른 type에 기초해 판단해야 할 필요가 있다.

Mapped types는 index signatures에서 사용하기 위한 syntax로 만들어졌으며 아직 선언된 적이 없는 properties의 types의 선언에 사용된다.

```js
type OnlyBoolsAndHorses = {
	[key: string]: boolean | Horse,
};

const conforms: OnlyBoolsAndHorses = {
	del: true,
	rodney: false,
};
```

Mapped type은 주로 `keyof`를 사용해 keys를 추출해 각각 적용(union 형태)하는 generic type이다.

```js
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

이 예시에서 `OptionFlags`는 `Type`으로부터 모든 properties를 가져와 values를 모두 boolean으로 바꾼다.

```js
type FeatureFlags = {
	darkMode: () => void,
	newUserProfile: () => void,
};

type FeatureOptions = OptionsFlags<FeatureFlags>;

// ❗️type FeatureOptions = { darkMode: boolean; newUserProfile: boolean; }
```

### A. Mapping Modifiers

Mapping 될 동안 적용될 수 있는 두 가지 modifiers가 있다: 하나는 `readonly` 다른 하나는 `?`이고 각각 변동성과 선택성에 영향을 준다.

당신은 `-` 혹은 `+`를 접두에 붙임으로써 이 modifiers를 추가하거나 제거할 수 있다. 만약 접우사를 붙이지 않으면 기본적으로 `+`가 붙게 된다.

```js
//  ☑️ Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;

// ❗️type UnlockedAccount = { id: string; name: string; }

//  ☑️ Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type User = Concrete<MaybeUser>;

// ❗️type User = { id: string; name: string; age: number; }
```

### B. Key Remapping via `as`

TypeScript 4.1 부터 mapped type에서 `as` 절을 이용해 key를 다시 매핑(re-map)할 수 있게 되었다.

```js
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

당신은 [template literal types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)와 같은 특성을 지렛대 삼아 이전의 property에 새로운 property 명칭을 생성할 수 있게 되었다.

```js
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
    name: string;
    age: number;
    location: string;
}

type LazyPerson = Getters<Person>;

// ❗️type LazyPerson = { getName: () => string; getAge: () => number; getLocation: () => string; }
```

당신은 conditional type을 거쳐 `never`을 생산함으로써 keys를 걸러낼 수 있다.

```js
// Remove the 'kind' property
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};

interface Circle {
    kind: "circle";
    radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;

// ❗️type KindlessCircle = { radius: number; }
```

당신은 임의의 unions를 매핑하는데 있어 `stirng | number | symbol` 뿐만 아니라 어떤 type의 union이라도 매핑할 수 있다.

```js
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}

type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };

type Config = EventConfig<SquareEvent | CircleEvent>

// ❗️type Config = { circle: (event: CircleEvent) => void; square: (event: SquareEvent) => void; }
```

### C. Further Exploration

Mapped types는 type 조작 부분에서 다른 특성들과도 잘 작동한다. 예를 들어 conditional type을 사용한 mapped type이 object가 `pii`라는 property를 가지고 있는지 여부에 따라 `true` 혹은 `false`를 반환한다고 가정하자.

```js
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};

type DBFields = {
  id: { format: "incrementing" };
  name: { type: string; pii: true };
};

type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;

// ❗️type ObjectsNeedingGDPRDeletion = { id: false; name: true; }
```

## 8. Template Literal Types

Teplate literal types는 [string literal types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types) 위에 만들어진 types이고 unions를 거쳐 많은 strings로 확장할 수 있는 능력을 가지고 있다.

Template literal types는 [template literal string in Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)와 같은 문법을 가지고 있지만 type position에서 사용한다는 차이가 있다. 구체적인 문자 types 사용과 함께 template literal은 선언된 구체적 문자 types에 이어 string literal type을 생산한다.

```js
type World = "world";

type Greeting = `hello ${World}`;

// ❗️type Greeting = "hello world"
```

삽입된 위치에 union이 사용되었을 때, type은 각각의 union memeber에 붙을 수 있는 모든 string literal에 붙여져 결과물을 산출한다.

```js
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;

// ❗️type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

Template literal에서 각각의 덧붙여진 위치에서 unions는 교차하며 곱해진다.

```js
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";

type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;

// 결과물 12개: 3 * 2 * 2
// ❗️type LocaleMessageIDs = "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" | "en_footer_sendoff_id" | "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id" | "ja_footer_sendoff_id" | "pt_welcome_email_id" | "pt_email_heading_id" | "pt_footer_title_id" | "pt_footer_sendoff_id"
```

우리는 일반적으로 큰 string unions에서 사전 생성하는 것을 추천했지만 이것은 더 작은 경우에서도 유용하다.

### A. String Unions in Types

Template literals의 힘은 type 안에 존재하는 string과 이제는 다른 새로운 string을 정의할 때 발휘된다.

예를 들어, JavaScript의 일반적 패턴은 최근에 가졌던 fields에 기초해 object를 확장하는 것이다. 우리는 value가 바뀌면 당신이 알아채도록 하는 `on`이라는 function을 지원하는 function을 위해 type 정의를 제공할 것이다.

```js
declare function makeWatchedObject(obj: any): any;

const person = makeWatchedObject({
	firstName: "Saoirse",
	lastName: "Ronan",
	age: 26,
});

person.on("firstNameChanged", (newValue) => {
	console.log(`firstName was changed to ${newValue}!`);
});
```

`on`이 `"firstName"` 뿐만 아니라 `"firstNameChange"` event를 듣고 있음을 주의 깊게 보라, template literals는 type system 안에서 이러한 종류의 string 조작 방법을 제공하고 있다.

```js

type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

/// Create a "watched object" with an 'on' method
/// so that you can watch for changes to properties.
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```

이것으로 우리는 잘못된 property가 제공되었을 때 에러를 발생시키도록 할 수 있다.

```js
const person = makeWatchedObject({
	firstName: "Saoirse",
	lastName: "Ronan",
	age: 26,
});

person.on("firstNameChanged", () => {});

// ❌ Prevent easy human error (using the key instead of the event name)
person.on("firstName", () => {});
// 🚫 Argument of type '"firstName"' is not assignable to parameter of type '"firstNameChanged" | "lastNameChanged" | "ageChanged"'.

// ❌ It's typo-resistant
person.on("frstNameChanged", () => {});
// 🚫 Argument of type '"frstNameChanged"' is not assignable to parameter of type '"firstNameChanged" | "lastNameChanged" | "ageChanged"'.
```

### B. Inference with Template Literals

마지막 예시(상기의 `type PropEventSource<Type>`)가 본래 value의 type을 재사용하지 않았다는 점에 주목하라. `callback`은 `any`를 사용하였다. Template literal types는 대체 positions로부터 추정할 수 있다.

우리는 연관된 property를 확인하기 위한 `eventName` string의 부분들로 generic이 추측할 수 있도록 할 수 있다.

```js
// ❗️ 이전 예시
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

// 💯 새로운 예시
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
        (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void ): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", newName => {

    // ❗️(parameter) newName: string
    console.log(`new name is ${newName.toUpperCase()}`);
});

person.on("ageChanged", newAge => {

    // ❗️(parameter) newAge: number
    if (newAge < 0) {
        console.warn("warning! negative age");
    }
})
```

여기서 우리는 `on`을 generic method로 만들었다.

사용자가 `"firstNameChange"` string을 호출하게 될 때, TypeScript는 `Key`에 적합한 type을 추정하려 할 것이다. 그렇게 하기 위해서 `"Changed"`에 앞서 content 옆에서 `Key`를 매치시키고 `"firstName"` string을 추정할 것이다. TypeScript가 확인을 완료하면 `on` method는 본래의 object에서 `string`이라는 `firstName`의 type을 불러올 수 있게 된다. 비슷하게, `"ageChanged"`가 호출되었을 때, TypeScript는 `number`인 `age` property에 적합한 type을 찾는다.

Inference(추정, 추론)은 다른 방식들로도 통합될 수 있는데, 종종 strings를 해체하고 다른 방식들로 다시 재결합한다.

### C. Intrinsic String Manipulation Types (본질적 String 조작 types)

String 조작을 돕기 위해 TypeScript는 string 조작에 사용되는 일련의 types를 제공한다. 이 types는 퍼포먼스를 위해 컴파일러에 built-in 형태로 제공되며 TypeScript의 `.d.ts` 파일들 안에서 발견되지 않는다.

- **a. `Uppercase<StringType>`**

String 안의 각각의 문자들을 대문자로 변환한다.

```js
type Greeting = "Hello, world"
type ShoutyGreeting = Uppercase<Greeting>
           
// ❗️type ShoutyGreeting = "HELLO, WORLD"
 
type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<"my_app">
       
// ❗️type MainID = "ID-MY_APP"
```

- **b. `Lowercase<StringType>`**

String 안의 각각의 문자들을 소문자로 변환한다.

```js
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
          
// ❗️type QuietGreeting = "hello, world"
 
type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
       
// ❗️type MainID = "id-my_app"
```

- **c. `Capitalize<StringType>`**

String 안의 첫 번째 문자를 대문자로 변환한다.

```js
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
        
// ❗️type Greeting = "Hello, world"
```

- **d. `Uncapitalize<StringType>`**

String 안의 첫 번째 문자를 소문자로 변환한다.

```js
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
              
// ❗️type UncomfortableGreeting = "hELLO WORLD"
```