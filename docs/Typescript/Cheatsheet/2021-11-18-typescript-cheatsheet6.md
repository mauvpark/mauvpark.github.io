---
layout: post
title: "6. Classes"
date: 2021-11-18 22:04:00 +0900
parent: Cheatsheet
grand_parent: Typescript
categories: typescript, cheat-sheet
nav_order: 2
comments: false
---

*2021-11-18 22:04 작성*

# Typescript 메모 6 (Classes)
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

## 1. Class Members

**--strictPropertyInitialization**

[strictPropertyInitialization](https://www.typescriptlang.org/tsconfig#strictPropertyInitialization) setting은 class fields가 `constructor` 안에서 초기화될 필요가 있는지 없는지를 통제한다.

```js
// ❌
class BadGreeter {
	name: string;
	// 🚫 Property 'name' has no initializer and is not definitely assigned in the constructor.
}

// ⭕️
class GoodGreeter {
	name: string;

	constructor() {
		this.name = "hello";
	}
}
```

Field는 *constructor*에서 초기화될 필요가 있다. TypeScript는 초기화 탐지에 있어서 constructor에서 적용한 methods를 분석하지 않는다. 그 이유는 파생 class는 그러한 methods를 무시할 뿐만 아니라 해당 members를 초기화하는데 실패하기 때문이다.

만약 당신이 constructor 보다 다른 방법들을 통해 field를 초기화할 의도가 있다면 `!` operator를 사용할 수 있다(예를 들어, 외부 library가 당신의 class 일부로 채워져 있다고 할 때):

```js
class OKGreeter {
    // Not initialized, but no error
    name!: string;
}
```

**readonly**

Fields는 `readonly` modifier를 접두어로 가질 수 있다. 이것은 contructor 외부에서 field에 할당되는 것을 방지한다.

```js
// @errors: 2540 2540
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }
}

// 💯
const g = new Greeter("hello"); // ✅ constructor 내부에서 변경 됨
// ❌
g.name = "why"; // 🚫 constructor 외부에서 변경이 불가능 함
```

### Constructors

Class constructor는 functions와 매우 비슷하다. 당신은 type 주석, default values,그리고 overloads와 함께 parameters를 추가할 수 있다.

```js
class Point {
	x: number;
	y: number;

	// Normal signature with defaults
	constructor(x = 0, y = 0) {
		this.x = x;
		this.y = y;
	}
}
```

```js
class Point {
  a: number;
  b: string;
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  // ❗️두 가지가 할당되면 x, y가 활성화 되고 한 가지만 할당되면 s만 활성화되며 y는 undefined 상태가 된다.
  constructor(xs: any, y?: any) {
    // TBD
    this.a = xs;
    this.b = y;
  }
}

const test1 = new Point(5, "world");
const test2 = new Point("hello");
console.log(test1); // ☑️ { "a": 5, "b": "world" }
console.log(test2); // ☑️ { "a": "hello", "b": undefined }
```

function signatures와 class constructor signatures 사이에 몇 가지 차이가 있다.

-   Constructors는 type parameters를 가질 수 없다 - 이것들은 외부 class 선언에만 해당된다.
-   Constructors는 type 주석을 반환할 수 없다 - class 객체 type은 항상 반환되는 그것이다.

**Super Calls**

Javascript에서처럼 만약 당신이 기반 class가 있다면 `this.` members를 사용하기 전에 contructor body 안에서 `super();`을 부를 필요가 있을 것이다:

```js
class Base {
	k = 4;
}

class Derived extends Base {
	constructor() {
		// Prints a wrong value in ES5; throws exception in ES6
		console.log(this.k);
		// 🚫 'super' must be called before accessing 'this' in the constructor of a derived class.
		super();
	}
}
```

JavaScript에서 저지르는 쉬운 실수 중 하나는 `super`를 부르지 않는 것이다. 하지만 TypeScript는 필요할 때 당신에게 알려줄 것이다.

### Methods

Class에서 function property는 *method*라고 불린다. Methods는 functions와 constructors에서 쓰는 똑같은 type 주석을 사용할 수 있다.

```js
class Point {
	x = 10;
	y = 10;

	scale(n: number): void {
		this.x *= n;
		this.y *= n;
	}
}
```

표준 Type 주석과 달리 TypeScript는 methods에 다른 새로운 것을 추가하지 않는다.

Method body 안에서 `this.`를 통해 fields와 다른 methods에 접근하는 것은 여전히 의무적이다. Method body 안의 자격이 확인되지 않은 명칭은 항상 둘러진 scope 안에서 참조가 가능하다.

```js
let x: number = 0;

class C {
	x: string = "hello";

	m() {
		// ❌ This is trying to modify 'x' from line 1, not the class property
		x = "world";
		// 🚫 Type 'string' is not assignable to type 'number'.
	}
}
```

### Getters / Setters

Classes는 *accessors(접근자)*를 가질 수 있다.

```js
class C {
	_length = 0;
	get length() {
		return this._length;
	}
	set length(value) {
		this._length = value;
	}
}
```

> _JavaScript에서 다른 기타 logic 없이 field를 지고 있는 get/set 쌍은 아주 드물게 유용하다. Get/set 동작 동안 만약 당신이 추가적인 logic을 더할 필요가 없다면 public fields를 노출시켜도 괜찮다._

TypeScript는 accessors에 대해 몇 가지 특별한 추론을 가진다.

-   만약 `get`이 `set` 없이 존재한다면 `get` property는 자동적으로 `readonly` 상태가 된다.
-   만약 setter parameter의 type이 구체화 되지 않으면 getter의 반환 type으로부터 추론된다.
-   Getters와 setters는 같은 [Member Visibility](https://www.typescriptlang.org/docs/handbook/2/classes.html#member-visibility)를 가져야만 한다.

TypeScript 4.3 이후로 getting과 setting은 서로 다른 types를 가질 수 있다.

```js
class Thing {
	_size = 0;

	get size(): number {
		return this._size;
	}

	set size(value: string | number | boolean) {
		let num = Number(value);

		// Don't allow NaN, Infinity, etc

		if (!Number.isFinite(num)) {
			this._size = 0;
			return;
		}

		this._size = num;
	}
}
```

### Index Signatures

Classes는 index signatures를 선언할 수 있다; 이것들은 [다른 object types에서의 Index Signatures](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures)와 동일하게 작동한다.

```js
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);

  check(s: string) {
    return this[s] as boolean;
  }
}
```

Index signature type은 methods의 types도 차지하는데 필요하기 때문에 이러한 types는 유용하게 사용하기가 쉽지는 않다. 일반적으로 indexed data(색인된 데이터)를 class 객체 그 자체보다 다른 곳에 저장하는 것이 더 좋다.

## 2. Class Heritage

객체 지향 특성을 사용하는 다른 언어들과 같이 JavaScript에서의 classes는 기반 classes로부터 상속받는다.

### `implements` Clauses

당신은 class가 특정 `interface`를 충족하는지 확인하기 위해 `implements` 절을 사용할 수 있다. 만약 class가 적절하게 실행되는데 실패하면 error가 발생한다.

```js
interface Pingable {
	ping(): void;
}

class Sonar implements Pingable {
	ping() {
		console.log("ping!");
	}
}

// ❌
class Ball implements Pingable {
	// 🚫 Class 'Ball' incorrectly implements interface 'Pingable'.
	// 🚫   Property 'ping' is missing in type 'Ball' but required in type 'Pingable'.
	pong() {
		console.log("pong!");
	}
}
```

Classes는 다중 interfaces를 실행할 수도 있다. 예시. `class C implements A, B {}`.

**Cautions**

`implements` 절은 class가 interface type으로 다루어질 수 있는지만을 확인한다는 점을 이해해야 한다. 이것은 class나 methods의 type을 _전혀_ 변경하지 않는다. Error의 일반적 근원은 `implements` 절이 class type을 변경할 것이라고 가정한다는 것이다. 실제로는 그렇지 않다!

```js
interface Checkable {
	check(name: string): boolean;
}

// ❌
class NameChecker implements Checkable {
	check(s) {
		// 🚫 Parameter 's' implicitly has an 'any' type.
		// Notice no error here
		return s.toLowerCase() === "ok";

		// 🚫 s.toLowercase = any
	}
}

// ⭕️
class NameChecker implements Checkable {
	check(s: string) {
		return s.toLowerCase() === "ok";
	}
}
```

위의 Error 예시에서 우리는 아마도 `s`의 type이 interface `check`의 `name: string` parameter로부터 영향을 받을 거라고 예측하고 있을 것이다. 그러나 그렇지 않다 - `implements` 절은 어떻게 class body가 확인 받고 type이 추론될 것인지에 대해 영향을 미치지 않는다.

비슷하게, *optional property*가 포함된 interface를 실행해도 해당 property를 생성하지는 않는다.

```js
interface A {
	x: number;
	y?: number;
}
class C implements A {
	x = 0;
}
const c = new C();
c.y = 10;
// 🚫 Property 'y' does not exist on type 'C'.
```

### `extends` Clauses

Classes는 기반 class에서 `extend`할지도 모른다. 파생 class는 기반 class의 모든 properties와 methods를 갖는다. 그리고 members를 추가하여 정의한다.

```js
class Animal {
	move() {
		console.log("Moving along!");
	}
}

class Dog extends Animal {
	woof(times: number) {
		for (let i = 0; i < times; i++) {
			console.log("woof!");
		}
	}
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

**Overriding Methods**

또한 파생 class는 기반 class의 field나 property를 덮어쓸(override) 수 있다. `super.` syntax를 사용해서 기반 class의 methods에 접근할 수 있다. JavaScript classes는 간편한 검색(lookup) object라서 "super field"의 개념이 없다.

TypeScript는 파생 class가 항상 기반 class의 부분 type이 되도록 강제한다.

예를 들어, 여기 method를 덮어쓸 수 있는 적합한 방식이 있다.

```js
class Base {
	greet() {
		console.log("Hello, world!");
	}
}

class Derived extends Base {
	greet(name?: string) {
		if (name === undefined) {
			super.greet();
		} else {
			console.log(`Hello, ${name.toUpperCase()}`);
		}
	}
}

const d = new Derived();
d.greet(); // ☑️ "Hello, world!"
d.greet("reader"); // ☑️ "Hello, READER"
```

파생 class가 기반 class의 계약을 따른다는 것은 중요하다. 기반 class 참조를 통해 파생 class 객체를 참조하는 것은 항상 적합하며 매우 일반적이라는 것을 기억하자:

```js
// Alias the derived instance through a base class reference
const b: Base = d;
// No problem
b.greet(); // ⭕️
b.greet("Hello"); // ❌
```

만약 `Derived`가 `Base`의 계약을 따르지 않는다면 어떻게 될까?

```js
class Base {
	greet() {
		console.log("Hello, world!");
	}
}

class Derived extends Base {
	// Make this parameter required
	greet(name: string) {
		// 🚫 Property 'greet' in type 'Derived' is not assignable to the same property in base type 'Base'.
		// 🚫   Type '(name: string) => void' is not assignable to type '() => void'.
		console.log(`Hello, ${name.toUpperCase()}`);
	}
}
```

만약 우리가 error가 발생함에도 이 코드를 compile하게 되면 이 sample을 충돌이 날 것이다.

```js
const b: Base = new Derived();
// ❌ Crashes because "name" will be undefined
b.greet();
```

**Initialization Order(초기화 순서)**

JavaScript classes가 초기화하는 순서는 몇몇의 경우에는 놀라울 수 있다. 다음의 코드를 살펴보자:

```js
class Base {
	name = "base";
	constructor() {
		console.log("My name is " + this.name);
	}
}

class Derived extends Base {
	name = "derived";
}

// Prints "base", not "derived"
const d = new Derived();
```

여기서 무슨 일이 일어난 것일까?

JavaScript에 의해 정희된 class 초기화의 순서:

-   기반 class fields가 초기화 됨
-   기반 class constructor가 동작함
-   파생 class fields가 초기화 됨
-   파생 class constructor가 동작함

이것은 기반 class constructor가 자체 constructor가 동작하는 중에 자체 value인 `name`을 보았다는 뜻이다. 그리고 파생 class field 초기화는 그 당시에 아직 동작하지 않았다.

```js
// 예상한대로 값이 나오게 하려면 다음과 같이 시도해볼 수 있다.
class Base {
	name = "base";
	change() {
		console.log("My name is " + this.name);
	}
}

class Derived extends Base {
	name = "derived";
}

const d = new Derived();
d.change(); // "My name is derived"
```

**Inheriting Built-in Types(deprecated 된 부분이 많으므로 삭제 표시 없는 곳만 참고)**

> _만약 당신이 Array, Error, Map 등과 같은 built-in types(이미 default로 정해진 type을 가지고 있는 것들)를 상속하는 것이 예정에 없거나 혹은 당신의 컴파일 target이 명확하게 ES6/ES2015 혹은 그 이상으로 setting 돼 있는 경우 이 section을 건너 뛰어도 된다._

> ES2015에서 object를 반환하는 constructors는 암시적으로 `super(...)`의 어느 호출자에 대해서나 `this`의 value를 대체한다. 이것은 `super(...)`의 어느 잠재적 return value라도 잡아내어 this를 이용해 대체하기 위해 필요하다.

~~결과적으로 `Error`, `Array` 그리고 다른 것들의 하부 class로 만들어도 더이상 기대한 대로 작동하지 않게 된다. 이것은 constructor가 `Error`, `Array` 그리고 prototype chain을 조절하기 위해 ECMAScript 6의 `new.target`을 사용하는 것처럼 기능한다는 사실 때문이다;
그러나, ECMAScript 5의 constructor를 적용할 때 `new.target`에 대한 value를 보장할 수 있는 방법이 없다. 다른 downlevel compilers도 일반적으로 같은 한계를 기본적으로 가지고 있다.~~

~~다음과 같은 하위 class를 보자:~~

```js
class MsgError extends Error {
	constructor(m: string) {
		super(m);
	}
	sayHello() {
		return "hello " + this.message;
	}
}

// 테스트용
let d = new MsgError("error!");

console.log(d.sayHello()); // "hello error!"
```

~~당신은 아마 다음과 같은 사실들을 발견할 것이다:~~

-   ~~Methods는 이러한 하위 classes를 구성하는 것에 의해 반환되는 objects에 대해 `undefined`일 수 있다.~~
-   ~~`instanceof`는 하위 class의 객체들과 그들의 객체들 사이에서 부숴질 것이다. 그래서 `(new MsgError()) instanceof MsgError`은 `false`를 반환할 것이다.~~

~~추천하자면, 당신은 손수 `super(...)` 호출 다음에 즉시 prototype를 적용할 수 있다.~~

// ❌ deprecated

```js
class MsgError extends Error {
	constructor(m: string) {
		super(m);

		// Set the prototype explicitly.
		Object.setPrototypeOf(this, MsgError.prototype);
	}

	sayHello() {
		return "hello " + this.message;
	}
}
```

~~그러나 `MsgError`의 어떤 하위 class라도 손수 prototype을 설정해주어야 한다. 런타임에서 [Object.setPrototyeOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)(**퍼포먼스 Issue가 있으므로 신중히 접근할 것!**)를 지원하지 않기도 하므로 [\_\_proto\_\_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)를 사용할 수도 있다.~~

## 3. Member Visibility

TypeScript를 이용해 특정 methods나 properties를 class 바깥에서 보이게 할 수 있다.

### A. `public`

Class members의 default 가시성(visibility)은 `public`이다. `public` member는 어디에서든 접근할 수 있다.

```js
class Greeter {
	public greet() {
		console.log("hi!");
	}
}
const g = new Greeter();
g.greet();
```

`public`은 이미 기본 가시성 수식어(default visibility modifier)이기 때문에 class member에 서술할 필요는 없다. 그러나 스타일/가독성의 이유로 선택될 수도 있다.

### B. `protected`

`protected` members는 선언된 class의 하위 classes에서만 가시적이다(보인다).

```js
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
  }
}
const g = new SpecialGreeter();
g.greet(); // ✅ OK
g.getName(); // ❌
// 🚫 Property 'getName' is protected and only accessible within class 'Greeter' and its subclasses.
```

**protected members의 노출(exposure)**

파생 classes는 그들의 기반 class 계약들을 따를 필요가 있다. 그러나 더 많은 능력을 위해 기반 class의 하부 type을 노출시킬 수도 있다. 이것은 `protected` members를 `public`으로 만드는 것을 포함한다.

```js
class Base {
	protected m = 10;
}
class Derived extends Base {
	// No modifier, so default is 'public'
	m = 15;
}
const d = new Derived();
console.log(d.m); // OK
```

`Derived`는 이미 `m`을 자유롭게 읽고 쓸 수 있다는 점을 주목해라. 그래서 이것은 이 상황에서의 "보안"을 의미있게 변경하지는 않는다. 여기에서 주목할 점은 파생 class에서 노출이 의도적이지 않을 경우 `protected` 수식어를 세심하게 반복할 필요가 있다.

**교차 계층 protected 접근**

다른 OOP(객체 지향 프로그래밍) 언어들은 기반 class 참조를 통해 `protected` member에 접근하는 것이 허용되는지 아닌지에 대해 서로 다른 견해를 가진다.

```js
class Base {
  protected x: number = 1;
}
class Derived1 extends Base {
  protected x: number = 5;
}
class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10;
  }
  f2(other: Base) {
	// ❌
    other.x = 10;
	// 🚫 Property 'x' is protected and only accessible through an instance of class 'Derived2'. This is an instance of class 'Base'.
  }
}
```

예를 들어 Java에서는 이것을 허용하는데 반해 C# 그리고 C++에서는 이 코드가 허용되지 않는다.

_TypeScript는 C#과 C++의 방향성을 가지고 있다. `Derived2`안의 `x`에 접근하는 것은 `Derived2`의 하위 classes에서만 허용된다._ 게다가 `Derived1` 참조를 통해 `x`에 접근하는 것은 허용되지 않는다(분명히 그래야 함에도!). 그리하여 기반 class 참조를 통해 접근하는 것은 상황을 개선시키지 못할 것이다.

더 자세한 정보를 원한다면 [왜 파생 Class에서 Protected Member에 접근할 수 없을까?](https://docs.microsoft.com/ko-kr/archive/blogs/ericlippert/why-cant-i-access-a-protected-member-from-a-derived-class)를 참고하라. C# 기준에서의 이유를 더욱 자세하게 설명해 줄 것이다.

### C. `private`

`private`은 `protected`와 비슷하지만, 하위 classes에서 조차 접근을 불허한다.

```js
class Base {
  private x = 0;
}
const b = new Base();
// ❌ Can't access from outside the class
console.log(b.x);
// 🚫 Property 'x' is private and only accessible within class 'Base'.
```

```js
class Derived extends Base {
	showX() {
		// ❌ Can't access in subclasses
		console.log(this.x);
		// 🚫 Property 'x' is private and only accessible within class 'Base'.
	}
}
```

`private` members는 파생 classes에서 보이지 않기 때문에 파생 class는 가시성을 높일 수 없다.

```js
class Base {
  private x = 0;
}
class Derived extends Base {
// 🚫 Class 'Derived' incorrectly extends base class 'Base'.
// 🚫   Property 'x' is private in type 'Base' but not in type 'Derived'.
  x = 1;
}
```

**교차 객체 private 접근**

어떤 다른 OOP 언어들은 같은 class의 서로 다른 객체들이 서로의 `private` members에 접근 가능하도록 하는 것을 허용하지 않는다. Java, C#, C++, Swift 그리고 PHP는 이것을 허용하지만 Ruby는 허용하지 않는다.

_TypeScript는 교차 객체 `private` 접근을 허용한다:_

```js
// ☑️ 예시 1
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}

let a = new A();
let b = new A();

a.sameAs(b); // OK

// ☑️ 예시 2
class A {
  private x = 10;

  constructor(k: number) {
    this.x = k;
  }

  public sameAs(other: A) {
    // No error
    return other.x === this.x; // false
  }
}

let a = new A(2);
let b = new A(5);

a.sameAs(b); // OK
```

**Caveats(경고)**

TypeScript의 type system의 다른 측면들과 마찬가지로 `private`과 `protected`는 [type 확인 동안만 강화된다](https://www.typescriptlang.org/play?removeComments=true&target=99&ts=4.3.4#code/PTAEGMBsEMGddAEQPYHNQBMCmVoCcsEAHPASwDdoAXLUAM1K0gwQFdZSA7dAKWkoDK4MkSoByBAGJQJLAwAeAWABQIUH0HDSoiTLKUaoUggAW+DHorUsAOlABJcQlhUy4KpACeoLJzrI8cCwMGxU1ABVPIiwhESpMZEJQTmR4lxFQaQxWMm4IZABbIlIYKlJkTlDlXHgkNFAAbxVQTIAjfABrAEEC5FZOeIBeUAAGAG5mmSw8WAroSFIqb2GAIjMiIk8VieVJ8Ar01ncAgAoASkaAXxVr3dUwGoQAYWpMHBgCYn1rekZmNg4eUi0Vi2icoBWJCsNBWoA6WE8AHcAiEwmBgTEtDovtDaMZQLM6PEoQZbA5wSk0q5SO4vD4-AEghZoJwLGYEIRwNBoqAzFRwCZCFUIlFMXECdSiAhId8YZgclx0PsiiVqOVOAAaUAFLAsxWgKiC35MFigfC0FKgSAVVDTSyk+W5dB4fplHVVR6gF7xJrKFotEk-HXIRE9PoDUDDcaTAPTWaceaLZYQlmoPBbHYx-KcQ7HPDnK43FQqfY5+IMDDISPJLCIuqoc47UsuUCofAME3Vzi1r3URvF5QV5A2STtPDdXqunZDgDaYlHnTDrrEAF0dm28B3mDZg6HJwN1+2-hg57ulwNV2NQGoZbjYfNrYiENBwEFaojFiZQK08C-4fFKTVCozWfTgfFgLkeT5AUqiAA).

이것은 JavaScript runtime이 `in` 혹은 `private` 혹은 `protected` member에 접근 가능한 간단한 property 색인을 구성한다는 의미이다.

```js
class MySafe {
  private secretKey = 12345;
}

// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```

`private`은 또한 type 확인 동안 괄호 표기를 사용하는 접근을 허용한다. 이러한 fields는 _부드러운 private_ 그리고 privacy를 엄격하게 강제하지 않는다는 결점을 가지고 있음에도 `private`으로 선언된 fields를 잠재적으로 unit tests와 같은 것들에 쉽게 접근이 가능하도록 만들기 때문에 사용된다.

```js
class MySafe {
  private secretKey = 12345;
}

const s = new MySafe();

// ❌ Not allowed during type checking
console.log(s.secretKey);
// 🚫 Property 'secretKey' is private and only accessible within class 'MySafe'.

// ⭕️ OK
console.log(s["secretKey"]);
```

TypeScript의 `private`과 달리 JavaScript의 [private fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields)(`#`)는 컴파일 후에도 private한 상태로 남는다. 그리고 이전에 언급되었던 *괄호 표기 접근*처럼 도피 수단을 제공하지 않으므로 *hard private(<->soft private)*한 상태가 된다.

```js
class Dog {
	// Hard private
	#barkAmount = 0;
	personality = "happy";

	constructor() {}
}
```

```js
"use strict";
class Dog {
	#barkAmount = 0;
	personality = "happy";
	constructor() {}
}
```

ES2021 혹은 그보다 덜한 버전으로 컴파일링 할 때, TypeScript는 `#`를 WeakMaps로 대체하여 사용할 것이다.

```js
"use strict";
var _Dog_barkAmount;
class Dog {
	constructor() {
		_Dog_barkAmount.set(this, 0);
		this.personality = "happy";
	}
}
_Dog_barkAmount = new WeakMap();
```

만약 악의적인 시도들로부터 당신의 class 안에 존재하는 values를 보호할 필요가 있다면 당신은 closures, WeakMaps 혹은 private fields와 같은 단단한(hard) runtime privacy를 제공하는 메커니즘을 사용해야 한다. Runtime 동안 이러한 추가된 privacy checks를 하는 것은 퍼포먼스에 영향을 끼친다는 점에 주목하라.

## 4. Static Members

Classes는 `static` members를 가질 수도 있다. 이 속성을 가진 members는 class의 특정 개체에 속하지 않는다. 이 members는 object 그 자체의 constructor을 통해 접근할 수 있다.

```js
class MyClass {
	static x = 0;
	static printX() {
		console.log(MyClass.x);
	}
}
console.log(MyClass.x);
MyClass.printX();
```

Static members는 또한 `public`, `protected` 그리고 `private`가시성 수식어와 함께 사용할 수 있다.

```js
class MyClass {
  private static x = 0;
}
// ❌
console.log(MyClass.x);
// 🚫 Property 'x' is private and only accessible within class 'MyClass'.
```

Static members는 상속될 수도 있다.

```js
class Base {
	static getGreeting() {
		return "Hello world";
	}
}
class Derived extends Base {
	myGreeting = Derived.getGreeting();
}
```

### 특수한 Static 명칭들

일반적으로 `Function` prototype의 properties를 덮어쓰는 것은 안전하지도 가능하지도 않다. 그 이유는 classes는 특정 `static` 명칭들이 사용될 수 없고 `new`를 이용해 적용될 수 있는 그 자체로서의 functions이기 때문이다. `name`, `length` 그리고 `call`과 같은 Function properties는 `static` members로서 정의될 수 없다.

```js
class S {
	// ❌
	static name = "S!";
	// 🚫 Static property 'name' conflicts with built-in property 'Function.name' of constructor function 'S'.
}
```

### 왜 Static Classes가 안되는가?

TypeScript(그리고 JavaScript)는 C#과 Java도 그렇듯이 `static class`라고 불리는 구조체를 가지고 있지 않다.

그러한 구조체(`static class`)들은 그러한 언어(`static class`를 허용하는 언어)들이 모든 데이터와 functions를 하나의 class 안에 넣도록 강제하기 때문에 오직 존재한다; TypeScript에서는 제한이 존재하지 않기 때문에 `static class`와 같은 구조체는 필요하지 않다. 하나의 객체만을 가지는 class는 일반적으로 JavaScript/TypeScript에서는 *일반(normal) object*라고 표현한다.

예를 들어 TypeScript에서 "static class" 문법은 필요없는데 흔히 사용하는 object(혹은 최상위 function)로 이미 역할을 잘 수행할 수 있기 때문이다:

```js
// ❌ Unnecessary "static" class
class MyStaticClass {
	static doSomething() {}
}

// ☑️ Preferred (alternative 1)
function doSomething() {}

// ☑️ Preferred (alternative 2)
const MyHelperObject = {
	dosomething() {},
};
```

## 5. Classes 안에서의 `static` Blocks

Static blocks는 private fields를 포함하고 있는 class의 private fields에 접근할 수 있게 하는 자체 scope를 가지고 일련의 문장을 서술할 수 있게 한다. 이는 Static blocks 안에는 코드문을 쓸 수 있고 변수의 누출 걱정을 하지 않아도 되며 그리고 class 내부에 위치한 모든 것에 접근할 수 있다는 의미이다. 그리고 이 능력들을 충분히 활용해 초기화 코드를 작성할 수 있다.

```js
class Foo {
	// JavaScript private: #
	static #count = 0;

	get count() {
		return Foo.#count;
	}

	static {
		try {
			const lastInstances = loadLastInstances();
			Foo.#count += lastInstances.length;
		} catch {}
	}
}
```

## 6. Generic Classes

어떻게 보면 Interfaces와 같은 classes는 generic이 될 수 있다. Generic class가 `new`를 통해 객체화 되었을 때, type parameters는 function 호출과 같은 방식으로 추론된다.

```js
class Box<Type> {
	contents: Type;
	constructor(value: Type) {
		this.contents = value;
	}
}

const b = new Box("hello!");

// ❗️const b: Box<string>
```

Classes는 interface와 같은 방식으로 generic 제약들과 기본값들을 사용할 수 있다.

### Static Members에서의 Type Parameters

아래와 같은 코드는 허용되지 않는다.

```js
class Box<Type> {
	// ❌
	static defaultValue: Type;
	// 🚫 Static members cannot reference class type parameters.
}
```

Types는 항상 완전히 삭제된다는 점을 기억하라! Runtime에서는 오직 `Box.defaultValue` property slot만이 남는다. 이것은 `Box<string>.defaultValue`(만약 가능했다면)가 `Box<number>.defaultValue`로도 변할 수 있다는 의미이며 이것은 좋지 않다. _Generic class의 `static` members는 절대 class의 type parameters를 참조할 수 없다._

## 7. Classes 안에서의 Runtime 그리고 `this`

TypeScript는 JavaScript의 runtime 동작을 변경하지 않는다는 것을 기억하고 JavaScript는 특유의 몇 가지 runtime 동작을 가지고 있다는 점을 유념하라.

JavaScript의 `this`를 조작하는 것은 사실상 드물다:

```js
class MyClass {
	name = "MyClass";
	getName() {
		return this.name;
	}
}
const c = new MyClass();
const obj = {
	name: "obj",
	getName: c.getName,
};

// Prints "obj", not "MyClass"
// 참조로 보았을 때 this가 obj 단위에서 선언되어 "obj" 값을 반환한다.
console.log(obj.getName());
```

기본적으로 긴 얘기를 짧게 말하자면 한 function 내의 `this`의 값은 *어떻게 function이 호출되었는지*에 의존한다. 여기에서 function은 `obj` 참조를 통해 호출되었는데 `this`의 값은 class 객체이기보다는 `obj`이다.

이게 당신이 바라던 바는 아니겠지만! TypeScript는 이러한 종류의 error를 방지하고 경감시키는 몇 가지 방법을 제공한다.

### Arrow Functions

만약 당신이 종종 `this` context를 상실하는 방식으로 호출되는 function을 가지고 있다면 method 정의 보다 arrow function property를 사용하는 것이 더 효과적이다.

```js
class MyClass {
	name = "MyClass";
	// 💯 Arrow function
	getName = () => {
		return this.name;
	};
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
```

Arrow function은 몇 가지 교환점을 가진다.

-   `this` 값은 TypeScript에서 코드가 확인되지 않아도 runtime에서 고쳐지도록 보장된다.
-   Arrow function은 더 많은 메모리를 사용한다. 그 이유는 각각의 clss 객체는 이러한 방식으로 정의된 각각의 function 복사본을 가지게 되기 때문이다.
-   당신은 파생 class에서 `super.getName`을 사용할 수 없다. 그 이유는 prototype chain에 기반 class method를 불러올 entry가 존재하지 않기 때문이다.

### `this` parameters

Method 혹은 function 정의에서 `this`라는 이름을 가진 초기 parameter는 TypeScript에서 큰 의미를 갖는다. 이러한 parameters는 컴파일할 동안 삭제된다.

```js
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
	/* ... */
}

// JavaScript output
function fn(x) {
	/* ... */
}
```

TypeScript는 `this` parameter를 가진 function이 올바른 context를 가지고 호출하는지를 확인한다. Arrow function을 사용하는 대신에 우리는 method가 적절하게 호출되었다는 것을 정적으로 강제하기 위해 `this` parameter를 method 정의에 추가한다.

```js
// @errors: 2684
class MyClass {
	name = "MyClass";
	getName(this: MyClass) {
		return this.name;
	}
}
const c = new MyClass();
// OK
c.getName();

let obj = {
	name: c.getName(),
};

// OK
console.log(obj.name);
const g = c.getName;
// Error, would crash
// ❌
console.log(g());
// 🚫 The 'this' context of type 'void' is not assignable to method's 'this' of type 'MyClass'.
```

이 방법은 arrow function 접근의 반대 교환점을 갖는다:

-   JavaScript 호출자는 깨닫지 못하고 부정확하게 class method를 사용한다.
-   Class 객체 당 한 개가 아닌 Class 정의 당 단 한 개의 function만 할당한다.
-   기반 metohd 정의는 `super`을 통해 호출할 수 있다.

## 7. `this` Types

Classes에서 `this`라는 특수한 type은 최근 class의 type을 _역동적으로_ 참조한다.얼마나 유용한지 같이 보도록 하자:

```js
class Box {
	contents: string = "";
	set(value: string) {
		// ❗️(method) Box.set(value: string): this
		this.contents = value;
		return this;
	}
}
```

여기서 TypeScript는 `set`의 반환 type이 `Box`보다는`this`가 될 것이라고 추측했다. 이제 `Box`의 하위 class를 만들어보자:

```js
class ClearableBox extends Box {
	clear() {
		this.contents = "";
	}
}

const a = new ClearableBox();
const b = a.set("hello");

// ❗️const b: ClearableBox
```

당신은 parameter type 주석에서 `this`를 사용할 수도 있다.

```js
class Box {
	content: string = "";
	sameAs(other: this) {
		return other.content === this.content;
	}
}
```

이것은 `other: Box`라고 적는 것과는 다르다 - 만약 당신이 파생 class를 가지고 있다면 `sameAs` method는 같은 추상 class의 다른 객체들만을 받아들일 것이다.

```js
class Box {
	content: string = "";
	sameAs(other: this) {
		return other.content === this.content;
	}
}

class DerivedBox extends Box {
	otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
// ❌
derived.sameAs(base);
// 🚫 Argument of type 'Box' is not assignable to parameter of type 'DerivedBox'.
// 🚫  Property 'otherContent' is missing in type 'Box' but required in type 'DerivedBox'.
```

### `this`에 기초한 type guards

당신은 classes의 methods와 interfaces의 반환 position에서 `this is Type`을 사용할 수 있다. Type 좁히기(e.g. `if`문)와 섞였을 때 target(목표) object의 type은 구체화된 `Type`으로 좁혀질 것이다.

```js
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
  fso.content;

// ❗️const fso: FileRep
} else if (fso.isDirectory()) {
  fso.children;

// ❗️const fso: Directory
} else if (fso.isNetworked()) {
  fso.host;

// ❗️const fso: Networked & FileSystemObject
}
```

`this`에 기초한 type guard의 흔한 사용 유형은 특정 field의 lazy validation을 허용하는 것이다. 예를 들어, 아래의 경우에 `hasValue`가 `true`로 확인됐을 때 box 안에 잠긴 value로부터 `undefined`를 제거한다.

## 8. Parameter Properties

TypeScript는 constructor parameter를 같은 명칭과 같은 값을 가진 class property로 변환하는 특수한 문법을 제공한다. 이것들은 *parameter properties*라고 불리는데 constructor argument에 `public`, `private`, `protected` 혹은 `readonly` 중 하나의 가시성 수식어를 접두사를 달아 생성한다. 그 결과로 초래된 field는 아래와 같은 수식어를 얻는다.

```js
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3);
console.log(a.x);
             
// ❗️(property) Params.x: number
// ❌
console.log(a.z);
// 🚫 Property 'z' is private and only accessible within class 'Params'.
```

## 9. Class Expressions

Class expressions는 class 선언과 비슷하다. 차이가 있다면 class expressions는 명칭이 필요없다는 점이며 그럼에도 불구하고 식별자에 맞춰 참조한다.

```js
const someClass = class<Type> {
  content: Type;
  constructor(value: Type) {
    this.content = value;
  }
};
 
const m = new someClass("Hello, world");
     
// ❗️const m: someClass<string>
```

## 10. `abstract` Classes와 Members

TypeScript에서의 Classes, methods 그리고 fields는 *추상적(abstract)* 일 수 있다.

*abstract method* 혹은 *abstract field*는 구체적 실행(implementation) 형태가 제공된 적이 없었던 것을 뜻한다. 이러한 members는 직접적으로 객체화 될 수 없는 *abstract class* 안에 존재해야만 한다. 

Abstract classes의 역할은 모든 abstract members를 실행하는 하위 classes의 기반 class를 받는 것이다. 한 class가 어떤 abstract members도 갖지 않을 때 *concrete(구체적인)*하다고 표현한다.

예시를 보자:

```js
abstract class Base {
  abstract getName(): string;
 
  printName() {
    console.log("Hello, " + this.getName());
  }
}
 
const b = new Base();
// 🚫 Cannot create an instance of an abstract class.
```

위와 같이 `Base`는 추상 class이기 때문에 `new`를 이용해 객체화할 수 없다. 대신에, 파생 class를 만들어 추상 members를 실행할 수 있다.

```js
class Derived extends Base {
  getName() {
    return "world";
  }
}
 
const d = new Derived();
d.printName(); // ✅ "Hello, world" 
```

만약 기반 class의 추상 members를 실행하는 것을 잊으면 error가 발생한다는 점을 알아두자:

```js
class Derived extends Base {
// 🚫 Non-abstract class 'Derived' does not implement inherited abstract member 'getName' from class 'Base'.
  // forgot to do anything
}
```

### Abstract Construct Signatures

때때로 당신은 어떤 추상 class로부터 파생하는 class의 객체를 생산하는 어떤 class constructor function을 받아들이길 원할 것이다.

예를 들어 다음과 같은 코드를 작성하길 원한다고 하자:

```js
function greet(ctor: typeof Base) {
  const instance = new ctor();
// 🚫 Cannot create an instance of an abstract class.
  instance.printName();
}
```

TypeScript는 정확하게 당신이 추상 class를 객체화하고 있다고 말해줄 것이다. 결국에 `greet`의 주어진 정의는 추상 class를 만들어내는 것으로 끝날 것이다.

```js
// Bad!
greet(Base);
```

대신에 construct signature를 이용해 허용하는 function을 작성할 수 있다.

```js
function greet(ctor: new () => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived); // ⭕️
greet(Base); // ❌
// 🚫 Argument of type 'typeof Base' is not assignable to parameter of type 'new () => Base'.
// 🚫   Cannot assign an abstract constructor type to a non-abstract constructor type.
```

이제 TypeScript는 어떤 class constructor functions가 적용될지에 대해 알맞게 알려줄 것이다 - `Derived`는 *concrete(구체적인)*하기 때문에 가능하지만 `Base`는 불가능하다.

## 11. Classes 간의 관계

대부분의 경우에서 TypeScript의 classes는 다른 types와 마찬가지로 구조적으로 비교된다.

예를 들어 여기 두 classes는 서로를 대체해 사용할 수 있다. 그 이유는 둘 다 똑같기 때문이다.

```js
class Point1 {
  x = 0;
  y = 0;
}
 
class Point2 {
  x = 0;
  y = 0;
}
 
// OK
const p: Point1 = new Point2();
```

비슷하게 classes 간의 하위 type 관계는 명확한 상속이 없다고 하더라도 존재한다.

```js
class Person {
  name: string;
  age: number;
}
 
class Employee {
  name: string;
  age: number;
  salary: number;
}
 
// OK
const p: Person = new Employee();
```

간단하게 들리지만, 다른 것들보다 좀 더 수상한 몇 가지 경우가 있다.

Members를 가지고 있지 않는 Empty classes가 있다. 구조적 type system에서 members가 존재하지 않는 type은 일반적으로 supertype(어떤 것이든 가능한 type)이 된다. 그래서 만약 당신이 빈 class를 작성한다면(하지 마라!) 어떤 것이든 위치시킬 수 있다.

```js
class Empty {}
 
function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}
 
// All OK!
fn(window);
fn({});
fn(fn);
```

{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}