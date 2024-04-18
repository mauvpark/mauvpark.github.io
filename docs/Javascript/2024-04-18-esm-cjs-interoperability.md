---
layout: post
title: (번역)ESM과 CJS의 상호작용성에 관해
date: 2024-04-18 15:00:00 +0900
parent: Javascript
categories: javascript, esm, cjs, interoperability
nav_order: 3
comments: false
---

_2024-04-18 작성_

# (번역)ESM과 CJS의 상호운용성에 관해

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

링크: [Modules - ESM/CJS Interoperability](https://www.typescriptlang.org/docs/handbook/modules/appendices/esm-cjs-interop.html)

2015년 당신은 ESM에서 CJS로 변환하는 트랜스파일러를 작성하고 있습니다. 그러나 이것을 어떻게 할지에 대한 기술 명세는 존재하지 않습니다. 있는 거라고는 ES 모듈이 서로 어떻게 상호작용하는지, CommonJS 모듈이 서로 어떻게 상호작용하는지 그리고 어떻게 해결할지에 대한 요령 뿐입니다. ES 모듈을 export 한다고 생각해봅시다.

```javascript
export const A = {};
export const B = {};
export default "Hello, world!";
```

당신이라면 위의 ES 모듈을 어떻게 CommonJS 모듈로 바꾸겠습니까? export 되는 것들은 기본적으로 **특별한 문법**과 함께 이름 지어진다는 것으로 볼 때, 오직 한 가지 선택만이 존재하는 것처럼 보입니다.

```javascript
exports.A = {};
exports.B = {};
exports.default = "Hello, world!";
```

훌륭한 아날로그 방식입니다. 자, 그럼 import 하는 방식에도 비슷하게 적용해보도록 합시다.

```javascript
import hello, { A, B } from "./module";
console.log(hello, A, B);

// 아래와 같이 변환된다.

const module_1 = require("./module");
console.log(module_1.default, module_1.A, module_1.B);
```

지금까지는 모든 CJS-세계에 있는 것들이 ESM-세계에 있는 것들과 1:1 대응하고 있습니다. 그럼, 이 동등함을 한 단계 더 확장해보도록 합시다. 그렇다면 우리는 다음과 같은 것들도 할 수 있습니다.

```javascript
import * as mod from "./module";
console.log(mod.default, mod.A, mod.B);

// 아래와 같이 변환된다.

const mod = require("./module");
console.log(mod.default, mod.A, mod.B);
```

당신은 이러한 scheme에서는 function, class 혹은 primitive가 할당 된 `exports` 결과물을 ESM export 형태로 생산할 수 있는 방법이 없다는 것을 인지할 수 있을 겁니다.

```javascript
// @File 이름: exports-function.js
module.exports = function hello() {
  console.log("Hello, world!");
};
```

하지만 CommonJS 모듈들은 종종 위와 같은 형식으로 존재하고는 합니다. 트랜스파일러로 처리되어, ESM import 형태로 이 모듈에 접근하는 방법은 무엇이 있을까요? 우리는 namespace import(`import *`)를 기본적인 `require` 호출 방식으로 변환하는 것을 규명했습니다. 그래서 우리는 다음과 같이 입력을 지원할 수 있다고 생각할 수 있습니다.

```javascript
import * as hello from "./exports-function"; // INFO 결과값: [Module: null prototype] { default: [Function: doSomething] }
hello();

// 아래와 같이 변환된다.

const hello = require("./exports-function");
hello();
```

우리의 결과물은 런타임에서 보면 작동합니다. 하지만 우리에게는 적합성에 대한 문제가 있습니다. 자바스크립트 기술 명세에 따르면, namespace import는 항상 Module Namespace Object(주석: Namespace란? export 된 모든 구성요소들의 통합 컨테이너로 생각할 수 있음)로 풀어내므로 한 오브젝트의 구성요소들은 모두 해당 모듈에서 export 된 것들일 것입니다. 이 경우 `require`는 `hello`라는 함수를 반환하겠지만, `import *`는 절대 함수를 반환하지 않습니다. 즉, **위에서 우리가 가정한 둘 사이의 관계성은 유효하지 않습니다**.

여기서 한 발짝 물러서서 *목표*가 무엇인지에 대해 명확히 해봅시다. 모듈이라는 개념이 ES2015 기술 명세에서 소개되자마자 ESM에서 CJS로 그 능력을 저하시키는 지원을 하는 트랜스파일러들이 떠올랐습니다. 그리고 이는 사용자들로 하여금 새로운 문법이 런타임에서 본격적으로 지원하기 이전에 채택할 수 있도록 했습니다. ESM 코드로 작성하는 것은 미래에 사용할 새로운 프로젝트들에게 있어서도 좋은 방법일 것이기에 상식처럼 받아들여졌습니다. 이것이 실제로 구현되기 위해서, 트랜스파일러들의 CJS 결과물들이 ESM으로 매끄럽게 이행되도록 할 필요가 있었고, ESM이 런타임에서 본격적으로 지원 될 때에 기능에 문제없이 ESM의 기술 스펙 그대로 적용될 필요가 있었습니다. 그렇기에 목표는 변환된 결과물들의 일부 혹은 전체 중 ESM에서 CJS로 변환된 코드들이 실제 ESM을 런타임에서 지원하게 되었을 때, 가시적인 변화 없이 변경 및 적용이 되도록 할 방법을 찾는 것이었습니다.

기술 명세를 따르는 트랜스파일러들은 변환된 CommonJS 결과물(원래 ESM 코드였던 것들)의 의미를 특정 ESM 입력값의 의미와 일치하도록 쉽게 조정할 수 있습니다(화살표는 import를 가리킴).

<img src="https://www.typescriptlang.org/d789cdf9a8f7f88a40e9e703e0698245/esm-cjs-interop.md-1.svg" alt="ESM and transpiled CJS">

그러나 CommonJS 모듈들(ESM에서 CommonJS로 변환된 것이 아니라 순수하게 CommonJS로 쓰여진 것들)은 이미 Node.js ecosystem에서 잘 정립되어 있어서 ESM으로 쓰여진 모듈과 변환된 CJS가 CommonJS로 쓰여진 모듈들을 "불러오기"를 하는 것은 불가피합니다. 상호 운용을 위한 방식은 ES2015에서 명시되지 않았고, 실제 런타임에서도 아직 존재하지 않았습니다.

<img src="https://www.typescriptlang.org/f935787a9210481bf751b445c081f07b/esm-cjs-interop.md-2.svg" alt="esm-cjs-interop">

트랜스파일러 작성자가 아무것도 하지 않았을지라도, 작성자들이 변환된 코드에서 emit한 `require` 호출과 존재하는 CJS 모듈에서 정의된 `exports` 사이에 존재하는 특정 의미로부터 어떤 동작이 나타날 것입니다. 그리고 사용자들에게 ESM이 런타임에서 지원 될 때, 변환된 ESM에서 true(완전한 ESM 방식의 ESM) ESM으로 매끄럽게 이행하게 하기 위해 이 동작은 런타임이 실행하기로 한 것과 일치해야 합니다.

상호운용 방식 런타임들이 지원하고자 했던 것들이 무엇이었는지 추측해보면, ESM은 "true(완전한 CJS 방식의 CJS) CJS" 모듈로만 가져오도록 하는 것은 아니었을 것입니다. ESM이 CJS로부터 변환된 ESM을 CJS와 별개로 인식할 수 있는지 그리고 CJS가 ES 모듈을 `require` 할 수 있는지 역시 불명확했습니다. ESM 가져오기(`import`) 방식이 CJS `require` 호출 방식과 같은 module resolution algorithm을 사용할지 여부도 알 수 없었습니다. 이러한 모든 변수들은 트랜스파일러가 사용자로 하여금 native ESM으로 매끄럽게 이행할 수 있도록 예측 가능하고 정확해야 합니다.

## `allowSyntheticDefaultImports`과 `esModuleInterop`

`import *`을 `require`로 변환하는 기술 명세 적합성 문제로 돌아가봅시다.

```javascript
// 기술 명세 상 유효하지 않은 경우
import * as hello from "./exports-function";
hello();

// 잘 작동하는 변환 예제
const hello = require("./exports-function");
hello();
```

타입스크립트가 ES 모듈을 작성하고 변환하는 것에 대해 처음 지원을 시작했을 때, 컴파일러는 namespace import 방식의 모듈에 대해 해당 모듈의 `exports`가 namespace와 같은 object를 반환하지 않는 경우 에러로 취급했습니다.

```javascript
import * as hello from "./exports-function";
// TS2497              ^^^^^^^^^^^^^^^^^^^^
// External module '"./exports-function"' resolves to a non-module entity
// and cannot be imported using this construct.
```

이것에 대한 유일한 해결책은 사용자로 하여금 CommonJS `require`로 대변되는 오래된 타입스크립트 import 문법을 사용하도록 하는 것이었습니다.

```javascript
import hello = require("./exports-function");
```

사용자로 하여금 non-ESM 문법으로 되돌리는 것은 근본적으로 "우리는 어떻게 해야 할지도 `"./exports-function"`과 같은 CJS 모듈이 미래에 언젠가 ESM import 형식으로 접근이 가능할지도 확신하지 못합니다. 그러나 우리가 사용하고 있는 변환 scheme 안에 있는 런타임을 통해 작동할 것이며, `import *`로 이용하는 형태는 아닐 겁니다."라고 시인하는 것이나 마찬가지입니다. 이는 위 파일을 어떤 변화 없이 소위 진짜 ESM으로 전환한다는 목표를 충족시키지 못할 뿐만 아니라 함수로의 연결에 있어서 `import *`을 대체하는 어떤 대안이 되지는 못합니다. 현재까지도 타입스크립트에서 `allowSyntheticDefaultImports`와 `esModuleInterop`이 disabled 상태일 때 이런 식으로 작동합니다.

> 불행하게도 이것은 좀 과도하게 간소화 되어 있습니다-타입스크립트는 이 문제에 대한 적합성 문제를 완전히 피하고 있지는 않았습니다. 왜냐하면 namespace 선언으로 선언된 함수가 병합되는 한, 함수의 namespace import를 통해 작동하게 했고, call signature를 유지했기 때문입니다-namespace가 비어있더라도 말이죠. bare 함수(주석: namespace 없이 직접 export 하는 함수)를 export 하는 한 "모듈이 아닌 개체"로 인식됩니다.
>
> ```javascript
> declare function $(selector: string): any;
> export = $; // `import *` 할 수 없음!👍
> ```
>
> 아래와 같이 의미가-없어야 하는 변경 방식은 유효하지 않은 import지만 타입 체크 결과 에러를 반환하지 않게 합니다.
>
> ```javascript
> declare namespace $ {}
> declare function $(selector: string): any;
> export = $; // `import *`가 가능하고 호출 할 수 있음! 😱
> ```

한편, 다른 트랜스파일러들은 같은 문제를 푸는 다른 방법을 찾아냈습니다. 사고 과정은 이런 거죠:

1. 함수 혹은 원시값을 export 하는 CJS 모듈을 import 하기 위해, 우리는 확실하게 default import를 사용할 필요가 있습니다. namespace import 방식은 규칙에 반하는 것이고 named import 방식은 여기서는 상식에 벗어납니다.
2. 아마도 ESM/CJS interop(상호운용)을 실행하는 런타임들은 CJS 모듈의 default import로 하여금 항상 모든 `exports`에 직접 연결하는 방법을 선택할 것입니다.
3. 그래서, true CJS 모듈의 default import는 `require` 호출 처럼 작동해야만 합니다. 하지만 우리는 true CJS 모듈과 transpiled CJS 모듈을 명확히 구분할 방법이 필요하기 때문에, `export default "hello"`를 `exports.default = "hello"`로 변환할 수 있고 _그_ 모듈의 default import를 `export.default`로 연결할 수 있습니다. 기본적으로, 우리의 변환된 모듈 중 하나 그리고 그것의 default import는 한 방식(ESM에서 ESM import처럼 보이는 것)처럼 작동해야 할 필요가 있고, 다른 CJS 모듈의 default import는 다른 방식(ESM에서 CJS import처럼 작동하는 것)으로 작동할 필요가 있습니다.
4. ESM 모듈을 CJS로 변환할 때, 결과값에 특별한 추가적 필드를 더해봅시다.

```javascript
exports.A = {};
exports.B = {};
exports.default = "Hello, world!";
// 스페셜 플래그!
exports.__esModule = true;
```

default import를 변환할 때 확인할 수 있습니다:

```javascript
// import hello from "./module";
const _mod = require("./module");
const hello = _mod.__esModule ? _mod.default : _mod;
```

`__esModule` 플래그는 Traceur에서 처음 쓰였고 그 이후 Babel, SystemJS 그리고 Webpack에서 쓰였습니다. 타입스크립트는 1.8 버전에서 `allowSyntheticDefaultImports`를 도입해 type checker가 `exports.default`보다는 `export default` 선언이 결여된 어떤 모듈 타입에서 사용할 수 있도록 `exports`에 직접 default import를 연결할 수 있도록 했습니다. 이 플래그는 import나 export 방식을 수정하는게 아닌 다른 트랜스파일러들이 default imports를 다루는 방식을 반영했습니다. 즉, default import가 "모듈이 아닌 개체들"을 다룰 수 있도록 했습니다. `import *` 에러를 확인할 수 있습니다:

```javascript
// 에러:
import * as hello from "./exports-function";

// 오래된 해결방법
import hello = require("./exports-function");

// 새로운 방식, `allowSyntheticDefaultImports`와 함께 사용
import hello from "./exports-function";
```

타입스크립트 없이도 이미 Babel과 Webpack에서는 지원을 했기 때문에 해당 시스템을 사용하는 사용자들에게 코드를 쓰도록 하는 것은 대개 괜찮았습니다. 그러나 이것은 부분적인 해결책이었고 몇 가지 문제들을 풀지 못한 채 남겨 놓았습니다:

1. Babel과 다른 시스템들은 `__esModule` 프로퍼티를 타겟 모듈에서 찾을 수 있는지 여부에 따라 default import 방식을 구별했습니다. 하지만 `allowSyntheticDefaultImports`는 타겟 모듈의 타입에서 default export가 없을 경우에만 _대체(fallback)_ 동작을 하도록 했습니다. 이것은 타겟 모듈의 default export가 _없을_ 경우에 비일관성을 만들어냈습니다. 트랜스파일러와 번들러는 여전히 default import를 모듈의 `exports.default`로 연결할 것이고, 이는 `undefined` 상태일 것이고 예상컨대 타입스크립트에서 에러를 발생시킬 것입니다. 진짜(real) ESM import는 연결할 수 없다면 에러를 발생시킬 것이기 때문이죠. 하지만 `allowSyntheticDefaultImports`가 적용되어 있다면, 타입스크립트는 그러한 import의 default import를 전체 `exports` 오브젝트로 연결시켜 named exports를 각각의 프로퍼티로써 접근할 수 있도록 허용할 것입니다.
2. `allowSyntheticDefaultImports`는 둘 다 사용 되어 질 수도 있고 같은 타입을 가질 수도 있는, 이처럼 이상한 비일관성을 생성하는 namespace imports가 타입 지어지는 방식을 변경하지는 않았습니다.

```javascript
// @파일 이름: exportEqualsObject.d.ts
declare const obj: object;
export = obj;

// @파일 이름: main.ts
import objDefault from "./exportEqualsObject";
import * as objNamespace from "./exportEqualsObject";

// 런타임에서 참이어야 하지만 타입스크립트는 에러를 발생시킵니다:
objNamespace.default === objDefault;
//           ^^^^^^^ Property 'default' does not exist on type 'typeof import("./exportEqualsObject")'.
```

3. 가장 중요한 것은, `allowSyntheticDefaultImports`는 `tsc`에 의해 emit되는 자바스크립트를 변경하지 않았습니다(주석: `allowSyntheticDefaultImports`에 의해 자바스크립트가 변경 됐다면 objDefault로 import 했을 때, export 된 모듈은 default가 되어야 하고 런타임 값은 참이어야 합니다). 플래그(주석: `allowSyntheticDefaultImports`)가 Babel이나 Webpack과 같은 다른 도구에 물린 코드 만큼 정확한 확인을 허용했지만, `tsc`를 사용해 `--module commonjs`로 emit하는 사용자들과 Node.js에서 돌아가는 것에 진짜 위험을 만들어냈습니다. 만약 위험에 처한 경우들이 `import *` 사용으로 에러를 마주했다면, `allowSyntheticDefaultImports` 플래그를 허용하는 것이 마치 에러들을 고쳐줄 것처럼 보이지만 사실은 Node에서 충돌 날 코드들을 emit하면서 빌드 타임 에러만 조용히 지나갈 뿐입니다.

타입스크립트는 2.7 버전에서 `esModuleInterop` 플래그를 도입하여 모듈 가져오기 관련 타입 검사를 개선했습니다. 이는 기존 트랜스파일러 및 번들러와의 호환성을 높이고 Node.js 환경에서의 문제 해결을 위해 필요했습니다. 그리고 트랜스파일러들이 몇 년 전에 가져온 같은 방식의 `__esModule`-조건부 CommonJS emit을 도입했습니다(새로운 emit helper는 `import *`에 대해 결과값이 항상 오브젝트가 되도록 보장했고, call signature는 빠졌으며, 회피하기 힘들었던 "resolves to a non-module entity" 에러가 나타나던 기술명세에 대한 준수 문제를 완벽히 해결했습니다). 마침내 새로운 플래그가 허용되고 타입스크립트의 코드 분석과 emit 그리고 CJS/ESM 상호운용 scheme을 수용하는 기타 트랜스파일링과 번들링 에코시스템은 Node에 의해 꽤 받아들여질만 하게 되었습니다.

## Node.js에서의 상호운용

Node.js v12부터 ES 모듈을 별도로 설정하거나 플래그를 사용하지 않고도 기본적으로 사용할 수 있게 되었습니다. 번들러와 트랜스파일러들이 몇 년 전부터 하기 시작한 것과 같이 Node.js는 CommonJS 모듈들에 `exports` 오브젝트의 "synthetic default export"를 제공했고, 모든 모듈 콘텐츠로 하여금 ESM으로부터 default import로 접근할 수 있게 했습니다:

```javascript
// @파일 이름: export.cjs
module.exports = { hello: "world" };

// @파일 이름: import.mjs
import greeting from "./export.cjs";
greeting.hello; // "world"
```

매끄럽게 마이그레이션이 이루어졌군요! 불행하게도, 이러한 공통점들은 여기서 마무리 됩니다.

### `__esModule` 탐색 없음("이중(double) default" 문제)

Node.js는 default import 방식을 분리하기 위해 `__esModule` 표시자를 존중할 수는 없었습니다. 그래서 "default export"로 변환된 모듈은 다른 변환된 모듈에 의해 "가져온(imported)" 된 상태일 때, 한 가지 방식으로 동작하게 되었습니다. 그리고 Node.js에서 true ES 모듈을 가져올 때는 다른 방식이 적용되었습니다:

```javascript
// @파일 이름: node_modules/dependency/index.js
exports.__esModule = true;
exports.default = function doSomething() {
  /*...*/
};

// @파일 이름: transpile-vs-run-directly.{js/mjs}
import doSomething from "dependency";
// 변환 후 잘 작동합니다. 그러나 Node.js의 ESM에서는 함수가 아닙니다:
doSomething();
// 변환 후에 존재하지 않습니다, 하지만 Node.js의 ESM에서는 잘 작동합니다.
doSomething.default();
```

변환된 default import는 타겟 모듈이 `__esModule` 플래그를 가지고 있지 않을 때에만 synthetic default export를 만들지만, Node.js는 _항상_ default export를 종합하여 변환된 모듈에 "이중 default"를 생성합니다.

> 주석: 이중 default에 대한 이해
>
> ```javascript
> // Node.js 환경
> import doSomething from "dependency";
> doSomething; // 호출
> // doSomething = {
> //  __esModule: true,
> //  default: function doSomething() {}
> // }
> // Default 1: Synthetic default export인 오브젝트 doSomething
> // Default 2: 오브젝트 doSomething의 default인 함수 doSomething
> ```

### 신뢰할 수 없는 named exports

CommonJS 모듈의 `exports` 오브젝트를 default import로 사용할 수 있게 만드는 것에 더해, Node.js는 named import로써도 사용 가능하도록 `exports`의 프로퍼티들을 찾으려고 시도합니다. 이러한 방식은 작동하는 한 번들러 그리고 트랜스파일러와 일치합니다; 그러나, 다른 코드를 실행하기 전에 named exports를 구문 분석하는 Node.js와 달리 변환된 모듈은 named imports는 런타임에서 해결합니다. 결과적으로 변환된 모듈에서 잘 작동하는 CJS 모듈의 imports는 Node.js에서는 작동하지 않을 것입니다.

```javascript
// @파일 이름: named-exports.cjs
exports.hello = "world";
exports["worl" + "d"] = "hello";

// @파일 이름: transpile-vs-run-directly.{js/mjs}
import { hello, world } from "./named-exports.cjs";
// `hello`는 작동합니다, 그러나 `world`는 Node.js에서 찾을 수 없습니다. 💥

import mod from "./named-exports.cjs";
mod.world;
// default에서 접근하는 프로퍼티는 항상 작동합니다. ✅
```

### True ES 모듈을 `require` 할 수 없음

True CommonJS 모듈은 ESM에서-CJS로-변환된 모듈을 `require`할 수 있습니다. 둘 다 런타임에서는 CommonJS이기 때문이죠. 하지만 Node.js에서 `require`를 통해 ES 모듈을 풀어내려고 시도하면 충돌이 나게 됩니다. 이는 출시된 라이브러리는 그 라이브러리들의 CommonJS(true 혹은 transpiled)를 사용하는 것들을 깨부수지 않고서는 변환된 모듈에서 true ESM으로 마이그레이션 할 수 없다는 것을 의미합니다.

```javascript
// @파일 이름: node_modules/dependency/index.js
export function doSomething() {
  /* ... */
}

// @파일 이름: dependent.js
import { doSomething } from "dependency";
// ✅ dependent와 dependency가 모두 변환된 상태라면 작동합니다.
// ✅ dependent와 dependency가 모두 true ESM이라면 작동합니다.
// ✅ dependent가 true ESM이고 dependency가 변환되었다면 작동합니다.
// 💥 dependent가 변환 되었고 dependency가 true ESM이라면 충돌이 납니다.
```

### 다른 모듈 해결 알고리즘

Node.js는 `require` 호출을 해결하는데 있어 오랫동안 사용하던 알고리즘과는 중대하게 다른, ESM imports를 해결하기 위한 새로운 모듈 해결 알고리즘을 소개했습니다. CJS와 ES 모듈 사이의 상호운용과 직접적으로 관계되지는 않았지만, 이 차이가 변환된 모듈에서 ESM으로 매끄럽게 마이그레이션 되는 것을 어렵게 만드는 한 요인이 되었습니다.

```javascript
// @파일 이름: add.js
export function add(a, b) {
  return a + b;
}

// @파일 이름: math.js
export * from "./add";
//            ^^^^^^^
// CJS로 변환 될 때 작동합니다, 하지만 Node.js ESM에서는 "./add.js"가 되어야만 합니다.
```

## 결론

확실히, 변환된 모듈에서 ESM으로 매끄럽게 마이그레이션 하는 것은 Node.js에서는 불가능한 것처럼 보입니다. 이것은 우리에게 어떤 것을 시사할까요?

### 올바른 `module` 컴파일러 옵션을 설정하는 것은 중요합니다.

상호운용에 관한 규칙은 호스트 마다 다르므로, 각각의 파일이 바라보는 모듈이 무슨 종류의 모듈인지, 그 파일들에 적용할 규칙을 무엇을 적용할 것인지에 대한 이해가 있지 않은 이상 타입스크립트가 올바른 코드 분석(checking behavior)을 제안할 수는 없습니다. 그리고 이것이 `module` 컴파일러 옵션의 목적이기도 합니다.(특정한 경우, Node.js에서 실행하도록 의도된 코드들이 번들러에 의해 접근되는 코드들 보다 더 까다로운 규칙의 대상이 됩니다. 컴파일러의 결과값은 `module`이 `node16` 혹은 `nodenext`로 되어 있지 않은 이상, Node.js 호환성에 대해 확인 되지 않습니다.)

### CommonJS 코드로 쓰여진 애플리케이션은 `esModuleInterop`을 항상 허용해야 합니다.

자바스크립트 파일을 emit하는 `tsc`를 사용하는 타입스크립트 _애플리케이션_ (다른 사용자들이 소비할지도 모르는 라이브러리에 반하는)에서 `esModuleInterop`이 허용 되어 있는지 여부가 큰 결과를 초래하지는 않습니다. 당신이 작성하는 특정 종류의 모듈을 import 하기 위한 방식이 변하겠지만, 타입스크립트의 분석(checking)과 emit이 동기화(sync) 상태에 있다면 에러에서-자유로운 코드는 양쪽 모드에서 실행되는데 안전할 것입니다. 이 경우 `esModuleInterop`을 비활성화 한 상태로 남겨두는 것의 단점은, 이를 비활성화 하는 것으로 인해 ECMAScript 기술명세에 반하는 문법을 가진 자바스크립트 코드를 사용자에게 허용하게 한다는 것입니다. Namespace imports에 대한 직관적 이해를 혼란스럽게 만들고, 미래에 작동 가능한 ES module로 마이그레이션하는 것에 어려움을 주게 됩니다.

반면에 써드파티 트랜스파일러 혹은 번들러에 의해 접근을 받는 애플리케이션은 `esModuleInterop`을 허용하는 것은 더욱 중요합니다. 모든 주요 번들러와 트랜스파일러들은 `esModuleInterop`-같은 emit 전략을 가지고 있습니다. 그래서 타입스크립트는 이 같은 전략에 일치 시킬 분석 방법에 적응할 필요가 있습니다. (컴파일러는 항상 `tsc`가 emit할 자바스크립트 파일에서 무엇이 일어날지 추론합니다, 그래서 `tsc` 대신 다른 도구를 사용한다 할지라도, emit에-영향을 미치는(emit-affecting) 컴파일러 옵션들은 사용하는 도구(주석: `tsc`와 비슷한 것들)의 결과물과 가능한 일치되게 설정 되어야 합니다.)

`esModuleInterop`이 없는 `allowSyntheticDefaultImports`는 지양 되어야 합니다. 이것은 `tsc`에 의해 emit되는 코드들의 변경 없이 컴파일러의 코드 분석을 변경하며, 잠재적으로 안전하지 않은 자바스크립트를 emit 시킬 수 있습니다. 뿐만 아니라, 해당 도구가 도입하는 방식으로 변경 사항을 확인하는 것은 `esModuleInterop`에 의해 도입되는 버전보다 더 불완전합니다.

어떤 사람들은 `esModuleInterop`이 허용 되었을 때, `tsc`의 자바스크립트 결과물로 산출되는 `__importDefault`와 `__importStart` 도우미 함수(helper functions)가 포함되는 것에 대해 반대 의견을 냅니다. 왜냐하면 디스크에서 결과물 크기를 미미하게 증가시킨다고 여기거나 `__esModule`을 적용함으로써 이 도우미들(helpers)에 의해 차용되는 상호운용 알고리즘이 앞서 논의했던 위험성으로 이끌 수 있으므로 이는 곧 Node.js의 상호운용 방식을 잘못 표현하고 있는 것이라고 여깁니다. 적어도 부분적으로 `esModuleInterop`을 비활성화 하는 것으로 나타나는 결함이 있는 코드 분석을 수용하지 않는 것을 제외하고는 두 가지 반대 주장 모두 나올 법 합니다. 첫째, `importHelpers` 컴파일러 옵션은 도우미 함수(helper functions)들을 필요로 하는 각각의 파일에 직접 삽입(inlining)하는 것보다는 `tslib`으로부터 도우미 함수들을 가져와 사용하는 옵션입니다. 두 번째 반대 의견에 대해 논의하기 위해 마지막 예시를 한 번 보도록 합시다:

```javascript
// @파일 이름: node_modules/transpiled-dependency/index.js
// INFO 주석: ESM-CJS 변환 모듈
exports.__esModule = true;
exports.default = function doSomething() {
  /* ... */
};
exports.something = "something";

// @파일 이름: node_modules/true-cjs-dependency/index.js
// INFO 주석: 순수 CJS 모듈
module.exports = function doSomethingElse() {
  /* ... */
};

// @파일 이름: src/sayHello.ts
// INFO 주석: 순수 ESM 모듈
export default function sayHello() {
  /* ... */
}
export const hello = "hello";

// @파일 이름: src/main.ts
import doSomething from "transpiled-dependency";
import doSomethingElse from "true-cjs-dependency";
import sayHello from "./sayHello.js";
```

Node.js에서 사용하기 위해 `src`를 CommonJS로 컴파일링 한다고 가정해봅시다. `allowSyntheticDefaultImports` 혹은 `esModuleInterop` 없이, `true-cjs-dependency`에서 `doSomethingElse`를 가져오는 것은 에러를 발생시키고, 나머지는 에러가 발생하지 않습니다. 어떤 컴파일러 옵션도 건들지 않고 이 에러를 수정하기 위해서 `import doSomethingElse = require("true-cjs-dependency")`와 같은 형태로 가져올 수 있을 것 입니다. Namepace import 방식으로 쓰고 호출하는 것을 통해 해결할 수도 있겠지만, 해당 모듈(보이지 않는)이 어떤 타입으로 쓰여졌는지에 의존하는 것은 언어-수준의 기술 명세 위반을 만들어 냅니다. `esModuleInterop`을 사용하면, 위의 모든 경우에서 에러를 발생하지 않을 것이고(그리고 호출 가능함), 유효하지 않은 namespace import는 잡힐 것입니다.

만약 우리가 Node.js에서 `src`를 true ESM으로 마이그레이션하기로 결정한다면 무엇이 변할까요(`"type": "module"`을 root package.json에 추가한다고 해봅시다)? 첫 번째 import인 `"transpiled-dependency"`에서 `doSomething`으로 가져오는 것은 더이상 호출 불가능 할 것입니다--이것은 "이중 default"를 드러낼 것이고 `doSomething()` 형식으로 가져오기 보다는 `doSomething.default()` 형식으로 호출해야 할 것입니다.(타입스크립트는 `--module node16`과 `nodenext` 하에서 이러한 이중 default 문제를 잡아내고 이해할 수 있습니다.) 하지만 특히, `doSomethingElse`인 두 번째 import에서는 CommonJS로 컴파일링 할 때 작동하도록 하기 위해 `esModuleInterop`이 필요하고, true ESM에서 잘 작동합니다.

만약 여기서 이의를 제기할 만한 것이 있다면, 두 번째 import에서 `esModuleInterop`이 하는 것과 관계된 것은 아닐 것입니다. default import를 허용하고 호출 가능한 namespace imports를 방지하는 `esModuleInterop`이 만들어내는 변화는 Node.js의 진짜(real) ESM/CJS 상호운용(interop) 전략에 의거한 것이고 진짜 ESM으로 좀 더 쉽게 마이그레이션 될 수 있도록 만듭니다. 한 가지 문제가 있다면, `esModuleInterop`이 _첫 번째_ import에 대한 매끄러운 마이그레이션 경로(path)를 제공하는데 실패하는 것 정도겠죠. 하지만 이 문제는 `esModuleInterop`을 활성화하는 것으로 인해 나타나는 문제는 아닙니다; 첫 번째 import는 완전히 `esModuleInterop`으로부터 영향을 받지 않죠. 불행하게도, 이 문제는 `main.ts`와 `sayHello.ts` 사이의 의미론적 약속(semantic contract)을 깨부수지 않는 이상 해결 될 것 같지는 않습니다. 왜냐하면 `sayHello.ts`의 CommonJS 결과물은 `transpiled-dependency/index.js`와는 구조적으로 동일한 것처럼 보이기 때문이죠. 만약 `esModuleInterop`이 `doSomething`의 변환된 import가 Node.js ESM에서도 동일한 방식으로 작동하도록 방식을 변화시킨다면 `sayHello`를 import하는 방식도 동일하게 변경할 것이고 입력 코드를 만드는 것은 ESM 의미론에 반하게 됩니다 (그리하여 `src` 디렉토리를 변경사항 없이 ESM으로 마이그레이션 하는 것을 여전히 막고 있는 것입니다).

우리가 봐왔듯이, 변환된 모듈에서 true ESM으로 매끄럽게 마이그레이션하는 경로는 없습니다. 하지만 `esModuleInterop`은 올바른 방향으로 가는 한 걸음입니다.모듈 문법의 변형과 import 도우미 함수를 포함하는 것을 최소화 하고 싶은 사용자들은 `esModuleInterop`을 비활성화 하는 것보다는 `verbatimModuleSyntax`를 활성화 하는 것이 좀 더 나은 선택지입니다. `verbatimModuleSyntax`는 CommonJS를-emit하는 파일들에서 `import mod = require("mode")`와 `export = ns`가 사용되게끔 강제해 우리가 지금까지 논의해왔던 import 모호성을 모두 피하고 true ESM으로 마이그레이션 하는 비용을 줄입니다.

### 라이브러리 코드는 특별한 고려가 필요합니다.

라이브러리들(선언 파일(Declaration files)를 추가한)은 넓은 범위의 컴파일러 옵션들 아래에서 에러에서-자유롭도록 타입에 대한 특별한 케어가 필요합니다. 예를 들어, 다른 interface를 상속하는 interface를 작성하는 방식으로 `strictNullChecks`가 비활성화 되었을 때만 성공적으로 컴파일 할 수 있게끔 할 수 있습니다. 만약 어느 라이브러리가 타입을 그러한 방식으로 퍼블리싱한다면, 모든 사용자로 하여금 `strictNullChecks`를 비활성화 하도록 강제할 것입니다. `esModuleInterop`은 타입 선언으로 비슷하게 default imports를 "전염시키는" 형식으로 포함할 수 있습니다.

```javascript
// @파일 이름: /node_modules/dependency/index.d.ts
import express from "express";
declare function doSomething(req: express.Request): any;
export = doSomething;
```

이 default import가 `esModuleInterop`이 _오직_ 활성화 되었을 때만 작동하고, 사용자가 `esModuleInterop` 옵션 없이 이 파일을 참조하려고 할 때 에러를 발생시킨다고 가정해봅시다.
그 사용자는 _아마도_ `esModuleInterop`을 어떤 식으로든 활성화 시켜야 할 것이나 일반적으로 환경설정에 이런 식으로 전염을 시키는 것은 좋지 않은 형식으로 보여질 것입니다.
라이브러리의 선언 파일(Declaration files)을 다음과 같이 작성하는 것이 더 나을 것입니다:

```javascript
import express = require("express");
// ...
```

이런 예시들은 라이브러리는 `esModuleInterop`을 활성화 해서는 *안 된다*는 관습적인 지혜에 따른 것입니다. 이러한 조언은 의미 있는 시작점입니다. 그러나 앞서 namespace import의 타입이 변하는 예시를 보았듯이, `esModuleInterop`을 활성화 했을 때, 잠재적으로 에러를 _가져옵니다_. 그래서 라이브러리들을 `esModuleInterop`과 함께 컴파일 할지 안 할지는, 라이브러리 개발자의 선택지를 사용자들에게 감염시킬지 여부를 선택하는 것과 같으므로 유의해야 합니다.

라이브러리 작성자들은 최대의 호환성을 보장하려고 하기 때문에 컴파일러 옵션을 쓰지 않는 방식으로 declaration files를 검증하려고 할 것입니다. 하지만 `verbatimeModuleSyntax`를 사용하는 것은 CommonJS로-emit하는 파일들을 CommonJS-스타일로 import하고 export하도록 하는 문법을 강제하는 것으로써 `esModuleInterop`과 관련된 문제를 완전히 피하는 것입니다. 게다가 `esModuleInterop`은 오직 CommonJS에만 영향을 미치므로 점점 더 많은 라이브러리들이 오직-ESM만 퍼블리싱하는 것으로 옮겨감에 따라 이 문제의 관계성은 점점 줄어들 것입니다.
