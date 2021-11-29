---
layout: post
title: "7. Modules"
date: 2021-11-19 15:52:00 +0900
parent: Typescript
categories: typescript, cheat-sheet
nav_order: 2
comments: false
---

_2021-11-19 15:52 작성_

# Typescript 메모 7 (Module)

본 내용은 [Typescript 공식 메뉴얼](https://www.typescriptlang.org/docs/)을 참고해 나오는 내용 중 필요한 내용만 발췌해 공부 목적으로 **번역 및 재구성**하였습니다.

## 도입

JavaScript는 모듈화 코드를 조작하는 여러 방식들에 관해 긴 역사를 가지고 있다. TypeScript는 2012년을 시작으로 적용되기 시작했기 때문에 이러한 형태의 많은 formats를 지원해왔다. 하지만 시간이 흐르면서 커뮤니티와 JavaScript 설명서도 ES Modules(혹은 ES6 Modules)라고 불리는 구성 방식(format)으로 수렴하기 시작했다. 당신도 `import/export` 문법이라는 것을 통해 이미 알고 있을 것이다.

ES Modules는 2015년 JavaScript 사양에 추가되었고 2020년 경 대부분의 웹 브라우저와 JavaScript runtimes에서 지원하기 시작했다.

이 핸드북은 ES Modules와 유명한 선도자격 CommonJS의 `module.exports =` 문법에 초점을 맞출 것이다. 만약 당신이 다른 module 패턴에 관심이 있다면 [Modules](https://www.typescriptlang.org/docs/handbook/modules.html) 섹션을 참고하면 된다.

## 1. JavaScript Modules가 정의되는 방식에 대해

ECMAScript 2015와 마찬가지로 TypeScript에서도 최상위 level에서 `import` 혹은 `export`를 포함하는 파일은 module로 여겨진다.

역으로, 최상위 level에서의 `import` 혹은 `export` 선언이 없는 파일은 global scope(global 범위)에서 사용할 수 있는 script의 contents로써 여겨진다.

Modules는 global scope가 아닌 그들 자신만의 scope 내에서 실행된다. 이것은 module 안에서 선언된 variables, functions, classes 등이 `export` 형태 중 하나를 사용하여 명확하게 외부로 선언된 것들이(exported) 아닌 이상 보이지 않는다는 말이다. 역으로, 다른 module로부터 외부로 선언된 variable, function, class, interface 등을 소비하기 위해서는 `import` 형태 중 하나를 이용하여 수입되어야(imported) 한다.

## 2. Non-Modules

시작하기 전에 TypeScript가 module을 고려한다는 점을 이해해야 한다. JavaScript 설명서에 따르면 `export` 혹은 최상위 `await`이 존재하지 않는 JavaScript 파일들이 module이 아닌 script로 여겨져야 한다고 설명한다.

Script 파일 안에 존재하는 variables와 types는 공유된 global scope 안에 존재하도록 선언된다. 그리고 이것은 당신이 다수의 입력 파일들을 하나의 결과 파일로 합치기 위한 [outFile](https://www.typescriptlang.org/tsconfig#outFile) compiler option으로 사용하거나 다수의 `<script>` 태그를 HTML 파일 안에 사용하여 파일들을 load한다.

만약 최근에 `import` 혹은 `export`를 사용한 적이 없는 파일이 있고 module로서 이용하고 싶다면 다음 line을 추가하면 된다:

```js
export {};
```

이것은 파일이 아무것도 exporting하지 않는 module로 변환한다. 이 문법은 당신의 module target에 관계없이 작동한다.

## 3. TypeScript의 Modules

TypeScript에서 module에 기반한 코드를 작성할 때는 세 가지를 고려해야 한다.

-   Syntax: import와 export 하기 위해 무슨 문법을 사용하기를 원하는가?
-   Module Resolution: module 명칭(혹은 경로)과 디스크의 파일들의 관계는 무엇인가?
-   Module Output Target: 내가 내놓은 JavaScript module은 무슨 모습이어야 하는가?

### Additional Import Syntax

`import {old as new}`를 사용하여 명칭을 새로 지정할 수 있다.

```js
import { pi as π } from "./maths.js";

console.log(π);

// ❗️(alias) var π: number
import π
```

`* as name`을 사용해 모든 export된 objects를 하나의 명칭 안에 넣을 수 있다.

```js
// @filename: app.ts
import * as math from "./maths.js";

console.log(math.pi);
const positivePhi = math.absolute(math.phi);

// ❗️const positivePhi: number
```

다른 변수를 집어넣지 않고도 파일을 import할 수도 있다.

```js
// @filename: app.ts
import "./maths.js";

console.log("3.14");
```

이 경우에는 `import`는 아무것도 하지 않는다. 그러나 `maths.ts`안에 존재하는 모든 code가 평가되고 다른 objects에 영향을 끼치는 부작용(부정적 의미 X)을 일으킨다.

**TypeScript Specific ES Module Syntax**

Types는 JavaScript 값들과 같은 문법을 사용해 import, export 될 수 있다.

```js
// @filename: animal.ts
export type Cat = { breed: string, yearOfBirth: number };

export interface Dog {
	breeds: string[];
	yearOfBirth: number;
}

// @filename: app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
```

`import type`은 types만 import하는 문법이다.

```js
// @filename: animal.ts
export type Cat = { breed: string, yearOfBirth: number };
export type Dog = { breeds: string[], yearOfBirth: number };
export const createCatName = () => "fluffy";
// 🚫 'createCatName' cannot be used as a value because it was imported using 'import type'.

// @filename: valid.ts
import type { Cat, Dog } from "./animal.js";
export type Animals = Cat | Dog;

// @filename: app.ts
// ❌
import type { createCatName } from "./animal.js";
const name = createCatName();
```

TypeScript 4.5부터는 개별 import가 허용되며 type의 경우 `type` 접두사를 붙여 다른 것들과 구분할 수 있다.

```js
// @filename: app.ts
import { createCatName, type Cat, type Dog } from "./animal.js";

export type Animals = Cat | Dog;
const name = createCatName();
```

non-TypeScript 트랜스파일러인 Babel, swc 혹은 esbuild는 모두 어떤 imports가 안전하게 사라져야할지 알기 위해 상기와 같은 형태를 허용한다.

## 4. CommonJS Syntax

CommonJS는 npm 대부분의 modules가 배달되는 형식이다. 만약 당신이 ES Modules 문법을 작성하더라도 CommonJS 문법이 어떻게 작동하는지 간단하게나마 이해하는 것은 debug에 도움을 줄 것이다.

**Exporting**

global 단위로 `exports` property을 설정하는 것을 통해 export 된 식별자들은 `module`이라고 불린다.

```js
function absolute(num: number) {
	if (num < 0) return num * -1;
	return num;
}

module.exports = {
	pi: 3.14,
	squareTwo: 1.41,
	phi: 1.61,
	absolute,
};
```

그리고 이 파일들은 `require` statement(구문)를 통해 import 될 수 있다.

```js
const maths = require("maths");
maths.pi; // ❗️any

// destructuring feature
const { squareTwo } = require("maths");
squareTwo; // ❗️const squareTwo: any
```

### CommonJS와 ES Modules intereop

default import와 module namespace object import 간의 차이에 대해 CommonJS와 ES Modules 간 features에 mis-match가 있다. TypeScript는 두 가지 방법의 서로 다른 제약 마찰을 줄이기 위해 [esModuleInterop](https://www.typescriptlang.org/tsconfig#esModuleInterop)과 함께 컴파일러 플래그를 가지고 있다.

{% if site.disqus.shortname %}
{% include disqus_comments.html %}
{% endif %}
