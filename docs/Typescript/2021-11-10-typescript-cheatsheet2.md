---
layout: post
title: "2. Narrowing"
date: 2021-11-10 21:50:00 +0900
parent: Typescript
categories: typescript, cheat-sheet
nav_order: 2
comments: false
---

*2021-11-10 21:50 작성*

# Typescript 메모 2 (Narrowing)
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

본 내용은 [Typescript 공식 메뉴얼](https://www.typescriptlang.org/docs/)을 참고해 나오는 내용 중 필요한 내용만 발췌해 공부 목적으로 재구성하였습니다.

## 1. `typeof` type guards

As we’ve seen, JavaScript supports a `typeof` operator which can give very basic information about the type of values we have at runtime. TypeScript expects this to return a certain set of strings:

-   `"string"`
-   `"number"`
-   `"bigint"`
-   `"boolean"`
-   `"symbol"`
-   `"undefined"`
-   `"object"`
-   `"function"`

Like we saw with padLeft, this operator comes up pretty often in a number of JavaScript libraries, and TypeScript can understand it to narrow types in different branches.

In TypeScript, checking against the value returned by `typeof` is a type guard. Because TypeScript encodes how `typeof` operates on different values, it knows about some of its quirks in JavaScript. For example, notice that in the list above, `typeof` doesn’t return the string null. Check out the following example:

```js
function printAll(strs: string | string[] | null) {
	if (typeof strs === "object") {
		for (const s of strs) {
			// 🚫 Object is possibly 'null'.
			console.log(s);
		}
	} else if (typeof strs === "string") {
		console.log(strs);
	} else {
		// do nothing
	}
}
```

In the printAll function, we try to check if strs is an object to see if it’s an array type (now might be a good time to reinforce that arrays are object types in JavaScript). **But it turns out that in JavaScript, `typeof` null is actually "object"!** This is one of those unfortunate accidents of history.

Users with enough experience might not be surprised, but not everyone has run into this in JavaScript; luckily, TypeScript lets us know that strs was only narrowed down to `string[] | null` instead of just string[].

This might be a good segue into what we’ll call “truthiness” checking.

## 2. Truthiness narrowing

In JavaScript, constructs like if first “coerce” their conditions to booleans to make sense of them, and then choose their branches depending on whether the result is true or false. Values like

-   `0`
-   `NaN`
-   `""` (the empty string)
-   `0n` (the bigint version of zero)
-   `null`
-   `undefined`

all coerce to false, and other values get coerced true. You can always coerce values to booleans by running them through the Boolean function, or by using the shorter double-Boolean negation. (The latter has the advantage that TypeScript infers a narrow literal boolean type true, while inferring the first as type boolean.)

```js
// both of these result in 'true'
Boolean("hello"); // type: boolean, value: true
!!"world"; // type: true,    value: true
```

It’s fairly popular to leverage this behavior, especially for guarding against values like null or undefined. As an example, let’s try using it for our printAll function.

```js
function printAll(strs: string | string[] | null) {
	if (strs && typeof strs === "object") {
		for (const s of strs) {
			console.log(s);
		}
	} else if (typeof strs === "string") {
		console.log(strs);
	}
}
```

Keep in mind though that truthiness checking on primitives can often be error prone. As an example, consider a different attempt at writing printAll ⬇️

```js
function printAll(strs: string | string[] | null) {
	// !!!!!!!!!!!!!!!!
	//  DON'T DO THIS!
	//   KEEP READING
	// !!!!!!!!!!!!!!!!
	if (strs) {
		if (typeof strs === "object") {
			for (const s of strs) {
				console.log(s);
			}
		} else if (typeof strs === "string") {
			console.log(strs);
		}
	}
}
```

**We wrapped the entire body of the function in a truthy check, but this has a subtle downside: we may no longer be handling the empty string case correctly.**

TypeScript doesn’t hurt us here at all, but this is behavior worth nothing if you’re less familiar with JavaScript. TypeScript can often help you catch bugs early on, but if you choose to do nothing with a value, there’s only so much that it can do without being overly prescriptive. If you want, you can make sure you handle situations like these with a linter.

One last word on narrowing by truthiness is that Boolean negations with `!` filter out from negated branches.

```js
function multiplyAll(
	values: number[] | undefined,
	factor: number
): number[] | undefined {
	if (!values) {
		return values;
	} else {
		return values.map((x) => x * factor);
	}
}
```

## 3. Equality narrowing

TypeScript also uses switch statements and equality checks like ===, !==, ==, and != to narrow types.

Checking against specific literal values (as opposed to variables) works also. In our section about truthiness narrowing, we wrote a `printAll` function which was error-prone because it accidentally didn’t handle empty strings properly. **Instead we could have done a specific check to block out `null`s, and TypeScript still correctly removes `null` from the type of `strs`.**

```js
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {

(parameter) strs: string[]
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);

(parameter) strs: string
    }
  }
}
```

JavaScript’s looser equality checks with == and != also get narrowed correctly. If you’re unfamiliar, checking whether something **`== null` actually not only checks whether it is specifically the value `null` - it also checks whether it’s potentially `undefined`. The same applies to `== undefined`: it checks whether a value is either `null` or `undefined`.**

```js
interface Container {
	value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
	// Remove both 'null' and 'undefined' from the type.
	if (container.value != null) {
		console.log(container.value);
		//❗️(property) Container.value: number

		// Now we can safely multiply 'container.value'.
		container.value *= factor;
	}
}
```

## 4. The `in` operator narrowing

JavaScript has an operator for determining if an object has a property with a name: the in operator. TypeScript takes this into account as a way to narrow down potential types.

For example, with the code: `"value" in x`. where `"value"` is a string literal and `x` is a union type. The “true” branch narrows `x`’s types which have either an optional or required property value, and the “false” branch narrows to types which have an optional or missing property value.

```js
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
	if ("swim" in animal) {
		return animal.swim();
	}

	return animal.fly();
}
```

To reiterate optional properties will exist in both sides for narrowing, for example a human could both swim and fly (with the right equipment) and thus should show up in both sides of the in check:

```js
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void, fly?: () => void };

function move(animal: Fish | Bird | Human) {
	if ("swim" in animal) {
		animal;

		//❗️(parameter) animal: Fish | Human
	} else {
		animal;

		//❗️(parameter) animal: Bird | Human
	}
}
```

## 5. `instanceof` narrowing

JavaScript has an operator for checking whether or not a value is an “instance” of another value. More specifically, in JavaScript x `instanceof` Foo checks whether the prototype chain of x contains Foo.prototype. While we won’t dive deep here, and you’ll see more of this when we get into classes, they can still be useful for most values that can be constructed with `new`. As you might have guessed, `instanceof` is also a type guard, and TypeScript narrows in branches guarded by `instanceof`s.

```js
// new Date() 형태의 객체가 param이 할당될 것이며 Date 객체 확인을 통해 타입을 확인한다.
function logValue(x: Date | string) {
	if (x instanceof Date) {
		console.log(x.toUTCString());

		//❗️(parameter) x: Date
	} else {
		console.log(x.toUpperCase());

		//❗️(parameter) x: string
	}
}
```

## 6. Control flow analysis

This analysis of code based on reachability is called _control flow_ analysis, and TypeScript uses this flow analysis to narrow types as it encounters type guards and assignments. When a variable is analyzed, control flow can split off and re-merge over and over again, and that variable can be observed to have a different type at each point.

```js
function example() {
	let x: string | number | boolean;

	x = Math.random() < 0.5;

	console.log(x);

	//❗️let x: boolean;

	if (Math.random() < 0.5) {
		x = "hello";
		console.log(x);

		//❗️let x: string;
	} else {
		x = 100;
		console.log(x);

		//❗️let x: number;
	}

	return x;

	//❗️let x: string | number;
}
```

## 7. Using type predicates

We’ve worked with existing JavaScript constructs to handle narrowing so far, however sometimes you want more direct control over how types change throughout your code.

To define a **user-defined type guard**, we simply need to define a function whose return type is a **_type predicate_**:

```js
// pet is Fish: type predicate(서술할 때의 술부), pet을 Fish 타입으로 강제
function isFish(pet: Fish | Bird): pet is Fish {
    // pet 객체 안에 swim이 존재하는지 여부를 boolean 형태로 return
  return (pet as Fish).swim !== undefined;
}
```

`pet is Fish` is our type predicate in this example. A predicate takes the form `parameterName` is Type, where `parameterName` must be the name of a parameter from the current function signature.

Any time `isFish` is called with some variable, TypeScript will narrow that variable to that specific type if the original type is compatible.

```js
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

// pet이 Fish인지를 확인하는 user-defined type guard가 실행 됨
if (isFish(pet)) {
	pet.swim();
} else {
	pet.fly();
}
```

Notice that TypeScript not only knows that `pet` is a `Fish` in the `if` branch; it also knows that in the else branch, you don’t have a `Fish`, so you must have a `Bird`.

You may use the type guard `isFish` to filter an array of `Fish | Bird` and obtain an array of `Fish`:

```js
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];

// The predicate may need repeating for more complex examples
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

### Class "This is Type" example

You can use `this is Type` in the return position for methods in classes and interfaces. When mixed with a type narrowing (e.g. `if` statements) the type of the target object would be narrowed to the specified `Type`.

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

// FileRep class를 constant에 할당하고 FileSystemObject 클래스 타입을 지정하면
// constant는 타입이 허용하는 기능에 접근할 수 있다.
const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

// FileRep의 객체를 가진 fso는 해당 객체(FileRep)를 바탕으로 아래의 명령어를 통해
// type predicates를 포함한 메서드를 수행하며 Type Checking을 수행한다.

// true
if (fso.isFile()) {
  fso.content;
  //❗️const fso: FileRep

// false
} else if (fso.isDirectory()) {
  fso.children;
  //❗️const fso: Directory

// false
} else if (fso.isNetworked()) {
  fso.host;
  //❗️const fso: Networked & FileSystemObject
}
```

A common use-case for a this-based type guard is to allow for lazy validation of a particular field. For example, this case removes an `undefined` from the value held inside box when `hasValue` has been verified to be true:

```js
class Box<T> {
  value?: T;

  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}

const box = new Box();
box.value = "Gameboy";

box.value;

// 🚫 (property) Box<unknown>.value?: unknown

if (box.hasValue()) {
  box.value;

  // 🚫 (property) value: unknown
}
```

## 8. Discriminated unions

```js
interface Shape {
	kind: "circle" | "square";
	radius?: number;
	sideLength?: number;
}

// error 1 - radius might not be defined
function getArea(shape: Shape) {
	return Math.PI * shape.radius ** 2;
	// 🚫 Object is possibly 'undefined'.
}

// error 2 - radius might not be defined.
function getArea(shape: Shape) {
	if (shape.kind === "circle") {
		return Math.PI * shape.radius ** 2;
		// 🚫 Object is possibly 'undefined'.
	}
}
```

Hmm, TypeScript still doesn’t know what to do here. We’ve hit a point where we know more about our values than the type checker does. We could try to use a non-null assertion (a `!` after `shape.radius`) to say that `radius` is definitely present.

```js
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

But this doesn’t feel ideal. We had to shout a bit at the type-checker with those non-null assertions (`!`) to convince it that `shape.radius` was defined, but those assertions are error-prone if we start to move code around. Additionally, outside of _strictNullChecks_ we’re able to accidentally access any of those fields anyway (since optional properties are just assumed to always be present when reading them). We can definitely do better.

The problem with this encoding of `Shape` is that **the type-checker doesn’t have any way to know whether or not `radius` or `sideLength` are present based on the `kind` property.** We need to communicate what we know to the type checker. With that in mind, let’s take another swing at defining `Shape`.

```js
interface Circle {
	kind: "circle";
	radius: number;
}

interface Square {
	kind: "square";
	sideLength: number;
}

type Shape = Circle | Square;
```

Here, we’ve properly separated `Shape` out into two types with different values for the `kind` property, but `radius` and `sideLength` are declared as required properties in their respective types.

```js
function getArea(shape: Shape) {
	switch (shape.kind) {
		case "circle":
			return Math.PI * shape.radius ** 2;

		//❗️(parameter) shape: Circle
		case "square":
			return shape.sideLength ** 2;

		//❗️(parameter) shape: Square
	}
}
```

**사용예시: They’re good for representing any sort of messaging scheme in JavaScript, like when sending messages over the network (client/server communication), or encoding mutations in a state management framework.**

## 9. The `never` type

When narrowing, **you can reduce the options of a union to a point where you have removed all possibilities and have nothing left.** In those cases, TypeScript will use a`never` type to represent a state which shouldn’t exist.

## 10. Exhaustiveness checking

For example, adding a `default` to our `getArea` function which tries to assign the shape to `never` will raise when every possible case has not been handled.

```js
type Shape = Circle | Square;

function getArea(shape: Shape) {
	switch (shape.kind) {
		case "circle":
			return Math.PI * shape.radius ** 2;
		case "square":
			return shape.sideLength ** 2;
		default:
			const _exhaustiveCheck: never = shape;
			return _exhaustiveCheck;
	}
}

// 🚫 error
interface Triangle {
	kind: "triangle";
	sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
	switch (shape.kind) {
		case "circle":
			return Math.PI * shape.radius ** 2;
		case "square":
			return shape.sideLength ** 2;
		default:
			const _exhaustiveCheck: never = shape;
			// 🚫 Type 'Triangle' is not assignable to type 'never'.
			return _exhaustiveCheck;
	}
}
```

{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}