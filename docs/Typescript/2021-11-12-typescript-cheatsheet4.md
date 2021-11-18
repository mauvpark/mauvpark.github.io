---
layout: post
title: "4. Object Types"
date: 2021-11-12 17:00:00 +0900
parent: Typescript
categories: typescript, cheat-sheet
nav_order: 2
---

*2021-11-12 17:00 작성*

# Typescript 메모 4 (Object Types)

본 내용은 [Typescript 공식 메뉴얼](https://www.typescriptlang.org/docs/)을 참고해 나오는 내용 중 필요한 내용만 발췌해 공부 목적으로 **번역 및 재구성**하였습니다.

## 1. Property Modifiers

```js
// Destructuring pattern
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
	console.log("x coordinate at", xPos);

	// ❗️(parameter) xPos: number
	console.log("y coordinate at", yPos);

	// ❗️(parameter) yPos: number
	// ...
}

// 🚫 Error
function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
	render(shape);
	// 🚫 Cannot find name 'shape'. Did you mean 'Shape'?
	render(xPos);
	// 🚫 Cannot find name 'xPos'.
}
```

object destructuring pattern 안에서 `shape: Shape`은 `shape` property를 붙잡아 `Shape`이라는 이름의 **_지역 변수로 재정의_**하라는 의미이다. 마찬가지로 `xPos: number`는 `number`라는 이름의 `xPos` parameter의 value에 기반한 변수를 생성한다.

### `readonly` Properties

Properties는 TypeScript에서 `readonly`로도 표시될 수 있다. 이것은 runtime 동안 어떤 행동도 바꾸지 않을 것이며 `readonly`로 표시된 property는 type-checking 시간 동안 쓰여질 수 없다.(can't be written.)

```js
interface SomeType {
  readonly prop: string;
}

function doSomething(obj: SomeType) {
  // We can read from 'obj.prop'.
  console.log(`prop has the value '${obj.prop}'.`);

  // But we can't re-assign it.
  obj.prop = "hello";
// 🚫 Cannot assign to 'prop' because it is a read-only property.
}
```

`readonly` modifier를 사용하는 것은 필연적으로 value가 **_완전히 변경 불가능하다는 것을 의미하지는 않는다._** 달리 말해, 내부 contents는 변경될 수 없다. **_단지 property 자체가 다시 재할당 될 수 없다는 의미이다._**(can't be re-written to.)

```js
// @errors: 2540
interface Home {
  readonly resident: { name: string; age: number };
}

function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
  // ❗️16 -> 17: 완전히 재할당이 된 것이 아니라면 변경이 가능하다.
  console.log(home.resident.age);
}

function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  // 🚫 Error: 재할당은 불가능하다.
  home.resident = {
    name: "Victor the Evictor",
    age: 42,
  };
}

const victor = {
  resident: {
    name: "victor",
    age: 16
  }
}

visitForBirthday(victor);
```

`readonly`가 암시하는 것이 무엇인지 예상하여 관리하는 것은 중요하다. 이것은 TypeScript의 devlopment time 동안 object가 어떻게 사용될 것인지에 대한 신호로서 유용하다. TypeScript는 두 개 타입의 properties가 checking 당시 `readonly`인지 타입들이 호환 가능한지에 대한 요소가 아니기 때문에 `readonly` properties는 가명(aliasing)을 통해 변경 가능하다.

```js
interface Person {
  name: string;
  age: number;
}

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
// ❗️aliasing에서 변경
writablePerson.age++;
writablePerson.name = "Novic";
console.log(readonlyPerson.age); // prints '43'
console.log(readonlyPerson.name); // prints "Novic"
```

> **_참고: 만약 `readonly` attributes를 없애고 싶다면 [mapping modifiers](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers)를 사용한다._**

```js
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;

// 다음과 같이 변경 된다.
/* type UnlockedAccount = {
    id: string;
    name: string;
} */
```

### Index Signatures

때때로 당신은 type properties의 모든 명칭을 알지 못 할 것이다. 하지만 values의 형태는 알고 있을 것이다.

그런 경우에 당신은 가능할 법한 values의 types를 기술하는데 있어 *index signature*를 사용할 수 있다.

```js
interface StringArray {
	[index: number]: string;
}

const myArray: StringArray = getStringArray();
const secondItem = myArray[1];

// ❗️const secondItem: string
```

위와 같이 우리는 index signature를 가지고 있는 `StringArray` interface를 가지고 있다. 이 index signature는 `StringArray`가 `number` 단위로 indexed(색인)되었을 때 `string`을 return할 것이라고 설명해준다.

Index signature property type은 `string` 혹은 `number` 형태여야만 한다.<br/>
(e.g. `[index: string]: return type`, `[index: number]: return type`)

> _두 가지 types를 indexer의 types로 지원하는 것도 가능하다._

`string` index signatures는 "dictionary" 패턴을 기술하는데 강력한 방법이지만 또한 모든 properties가 return type에 매칭되도록 강제하기도 한다.이것은 `string` index가 `obj.property`는 `obj["property"]`로도 이용 가능하다는 것을 선언하기 때문이다. 다음 예시와 같이 `name`의 type은 `string` index type과 매칭되지 않기에 type checker에서 error를 일으킨다.

```js
interface NumberDictionary {
	[index: string]: number;

	length: number; // ok
	name: string;
	// 🚫 Property 'name' of type 'string' is not assignable to 'string' index type 'number'.
}
```

상기와 같이 모든 Properties는 `string` index signature에 의해 타입이 강제되며 따라서 `name`의 return type도 `number` 형태가 되어야 한다.

그러나 만약 index signature의 return type이 union 형태라면 union types의 범위 안에서 type이 허용된다.

```js
interface NumberOrStringDictionary {
	[index: string]: number | string;
	length: number; // ok, length is a number
	name: string; // ok, name is a string
}
```

마침내 당신은 indices에 value가 할당되는 것을 예방하기 위한 `readonly`를 index signatures에 적용할 수 있게 되었다.

```js
interface ReadonlyStringArray {
  readonly [index: number]: string;
}

let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
// 🚫 Index signature in type 'ReadonlyStringArray' only permits reading.
```

## 2. Extending Types (Types의 확장)

```js
interface BasicAddress {
	name?: string;
	street: string;
	city: string;
	country: string;
	postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
	unit: string;
}

// multiple extending
interface Colorful {
	color: string;
}

interface Circle {
	radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
	color: "red",
	radius: 42,
};
```

## 3. Intersection Types

`interface`는 확장을 통해 다른 types를 이용한 새로운 types를 생성할 수 있도록 한다. TypeScript는 *intersection types*라고 하는 또 다른 생성 방법을 제공하는데 주로 이미 존재하는 object types을 통합하는데 사용된다.

*intersection type*은 `&` operator를 이용해 정의할 수 있다.

```js
interface Colorful {
	color: string;
}
interface Circle {
	radius: number;
}

type ColorfulCircle = Colorful & Circle;

function draw(circle: Colorful & Circle) {
	console.log(`Color was ${circle.color}`);
	console.log(`Radius was ${circle.radius}`);
}

// okay
draw({ color: "blue", radius: 42 });

// oops
draw({ color: "red", raidus: 42 });
// 🚫 Argument of type '{ color: string; raidus: number; }' is not assignable to parameter of type 'Colorful & Circle'.
// 🚫 Object literal may only specify known properties, but 'raidus' does not exist in type 'Colorful & Circle'. Did you mean to write 'radius'?
```

## 4. Interfaces vs. Intersections

우리는 방금 types를 통합하는 두 가지 방법을 확인했고 비슷하다는 점을 알게 되었다. 그러나 실제로는 미묘하게 다르다. interfaces를 사용할 때, 우리는 `extends` 절을 사용했다. 그리고 비슷한 동작을 intersections에서 할 수 있었고 그 결과치를 type alias로 명칭을 지었다. 두 가지 방법의 원칙적 차이는 어떻게 conflicts(충돌)이 관리된다는 점이다. 그리고 주된 일반적 차이는 당신이 interface를 이용하느냐 type alias에 기반한 intersection type을 이용하느냐에 따라 달려있다.

### Generic Object Types

```js
interface NumberBox {
  contents: number;
}

interface StringBox {
  contents: string;
}

interface BooleanBox {
  contents: boolean;
}

// ❌ type에 따라 매 번 다른 function을 불러와야 함.
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}

// Generic type
// 💯 Type을 쉽게 바꿀 수 있으며 재사용이 가능함.
interface Box<Type> {
  contents: Type;
}

// Type alias
// Equivalent as...
type Box<Type> = {
  contents: Type;
};

interface Apple {
  // ....
}

// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;

let box: Box<string>;

function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}

// 중첩 예시
type OrNull<Type> = Type | null;

type OneOrMany<Type> = Type | Type[];

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;

type OneOrManyOrNull<Type> = OneOrMany<Type> | null

type OneOrManyOrNullStrings = OneOrManyOrNull<string>;

type OneOrManyOrNullStrings = OneOrMany<string> | null
```

### The `ReadonlyArray` Type

`ReadonlyArray`는 변경할 수 없는 배열을 나타내는 특별한 type이다.

```js
function doStuff(values: ReadonlyArray<string>) {
	// We can read from 'values'...
	// slice()는 배열복사, 원본 배열은 바뀌지 않음.
	const copy = values.slice();
	console.log(`The first value is ${values[0]}`);

	// ...but we can't mutate 'values'.
	values.push("hello!");
	// 🚫 Property 'push' does not exist on type 'readonly string[]'.
}
```

properties에서 사용하는 `readonly` modifier와 비슷하며 특정한 의도를 가지고 사용할 수 있다. `ReadonlyArray`를 return하는 `function`을 보면 우리는 contents를 변경하지 않기로 약속되어 있다고 말해준다. 그리고 `ReadonlyArray`를 소비하는 `function`을 보면 contens를 변경할 것이라는 주의없이 어떤 array라도 function 안에 집어 넣을 수 있다고 말해준다.

```js
new ReadonlyArray("red", "green", "blue");
// 🚫 'ReadonlyArray' only refers to a type, but is being used as a value here.

// 오류 없이 할당 가능
let roArray: ReadonlyArray<string> = ["red", "green", "blue"]; // ✅
roArray = ["blue", "yellow"]; // ✅
roArray.push("purple"); // 🚫 Error! Property 'push' does not exist on type 'readonly string[]'.
```

TypeScript에서 `Array<Type>`를 속기(shorthand) syntax `Type[]`형태로 제공하는 것처럼 `ReadonlyArray<Type>`은 `readonly Type[]` 형태로 제공한다.

**마지막으로 주의할 점은 `readonly` property modifier와 다르게 `Array`와 `ReadonlyArray` 간 양방향(bidirectional) 할당은 불가능하다.**

```js
let x: readonly string[] = [];
let y: string[] = [];

x = y;
// y error!
y = x;
// 🚫 The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
```

### Tuple Types

*Tuple type*은 `Array`의 다른 종류로 정확히 얼만큼의 elements가 포함되는지 정해진다.(fixed) 그리고 정확히 특정 positions에 해당 types들이 포함된다.

```js
type StringNumberPair = [string, number];
```

여기에서 `StringNumberPair`는 `string`과 `number`로 구성된 *tuple type*이다. `ReadonlyArray`와 마찬가지로 runtime이 진행될 동안은 나타내는 것이 없는데 이것은 TypeScript에 있어서 매우 중요하다. Type system에게 `StringNumberPair`는 `0` index에 `string`이 포함되어 있고 `1` index에는 `number`가 포함되어 있다고 기술한다.

```js
function doSomething(pair: [string, number]) {
	const a = pair[0];

	//❗️const a: string
	const b = pair[1];

	//❗️const b: number
	// ...
}

doSomething(["hello", 42]);

// Destructuring pattern
function doSomething(stringHash: [string, number]) {
	const [inputString, hash] = stringHash;

	console.log(inputString);

	//❗️const inputString: string

	console.log(hash);

	//❗️const hash: number
}
```

> _`Tuple types`는 각각의 element의 의미가 "명확"하다는 전제에서 철저하게 작성된 convention-based APIs를 사용할 때 유용하다. 이것은 우리가 tuple type을 destructure할 때 원하는 형태의 변수를 줄 수 있다는 점에서 융통성이 있다. 위와 같이 `0`와 `1`이라고 element를 명명할 수 있었으며 다른 형태로도 가능하다._

> _그러나 모든 사용자가 명확하다는 것에 같은 view를 공유하고 수긍하는 것은 아니므로 descriptive property를 사용하는 object 형태를 사용하는 것이 더 나은 선택일지도 모른다._

Tuple에 있어서 또 다른 흥미로운 점은 tuple은 `?`를 element type 바로 뒤에 붙이는 형태로 optional properties를 구현할 수 있다는 점이다. **Optional tuple elements는 오직 끝에 왔을 경우에만 작동하며 `length`에도 영향을 미친다.**

```js
type Either2dOr3d = [number, number, number?];

function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;

const z: number | undefined

  console.log(`Provided coordinates had ${coord.length} dimensions`);

  //❗️(property) length: 2 | 3
}
```

또한 Tuples는 rest elements도 가질 수 있다.

```js
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

-   `StringNumberBooleans`의 처음 두 개의 elements는 각각 `string`과 `number`이고, 그 다음 여러 개의 `boolean` elements가 나열된다.
-   `StringBooleansNumbver`의 첫 element는 `string`이며 여러 개의 `boolean` elements가 위치하고 마지막으로 `number` elements가 오게 된다.
-   `BooleansStringNumber`은 처음 여러 개의 `boolean` elements가 오고 그 후에 각각 `string`과 `number` elements가 오게 된다.

Tuple의 rest element에는 "length"가 고정되어 있지 않다. rest element의 수가 얼마나 존재 하는지에 따라 달라진다.

```js
type StringNumberBooleans = [string, number, ...boolean[]];
// ---cut---
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];

console.log(a.length); // 2
console.log(b.length); // 3
console.log(c.length); // 7
```

왜 optioanl 그리고 rest elements가 유용할까? 음, 이것은 TypeScript로 하여금 tuples가 parameter lists와 통신하게끔 하기 때문이다. Tuple types는 [rest parameters와 arguments](https://www.typescriptlang.org/docs/handbook/2/functions.html#rest-parameters-and-arguments)에 사용될 수 있다.

```js
// 복잡한 형태
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
// 간편한 형태, 한 개의 param으로 공통 관리
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

**이것은 당신이 rest parameter가 포함된 여러 개의 arguments를 하나의 변수로 압축하여 사용할 때 간편하다. 이로써 당신은 최소한의 elements를 적용하게 되고 중간 매개 변수를 할당할 필요가 없게 된다.**

### `readonly` Tuple Types

Tuple types에 관한 마지막 메모 - tuples types는 `readonly` variants를 가지고 있다. 그리고 tuples types 앞에 `readonly` modifier를 붙임으로써 구체화된다. array 속기(shorthand) syntax와 동일하다.

```js
function doSomething(pair: readonly [string, number]) {
  // ...
}
```

당신도 예상했겠지만 `readonly` tuple에 어떤 property를 쓰더라도 TypeScript에서 허용하지 않는다.

```js
function doSomething(pair: readonly [string, number]) {
  pair[0] = "hello!";
  // 🚫 Cannot assign to '0' because it is a read-only property.
}
```

Tuples는 대부분의 코드에서 생성되고 수정되지 않은 상태로 존재하는 경향이 있다. 그래서 가능하다면 `readonly` tuples라고 types에 주석을 다는 것은 좋은 default라고 볼 수 있다. 이것은 또한 `readonly` tuple types를 암시하는 `const` assertions가 적용된 array로도 볼 수 있다.

```js
// 암시적 readonly tuples, 실제로는 array에 const assertion이 적용되었다.
let point = [3, 4] as const;
 
function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}
 
distanceFromOrigin(point);
// 🚫 Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
// 🚫 The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.
```

{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}