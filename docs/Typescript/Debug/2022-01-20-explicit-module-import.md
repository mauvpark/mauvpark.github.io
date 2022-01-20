---
layout: post
title: d.ts에서의 명시적 module import
date: 2022-01-20 18:09:00 +0900
parent: Debug
grand_parent: Typescript
categories: typescript, debug, module-import, d.ts
nav_order: 1
comments: false
---

*2022-01-20 18:09 작성*

# `d.ts`에서의 명시적 Module import에 관해
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

## 도입

```javascript
// can not find module
import example from 'assets/Tiger.png';
```

<div class="code-example">
    <h3>
        Cannot find module 'assets/protein.png' or its corresponding type declarations.
    </h3>
</div>

```json
// tsconfig
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
        "app/*": ["./src/app/*"],
        "config/*": ["./src/app/_config/*"],
        "environment/*": ["./src/environments/*"],
        "shared/*": ["./src/app/_shared/*"],
        "helpers/*": ["./src/helpers/*"],
        "tests/*": ["./src/tests/*"],
        "assets/*": ["./assets/*"],
    },
  }
}
```

위와 같이 `tsconfig.json`이 설정 되어 있다고 해도 assets 폴더의 png, jpeg 등의 resources를 import 할 때 에러가 발생할 수 있다. 이 문제를 해결하기 위해 다음의 두 가지 사례를 기술해둔다.

## 실패 사례

```json
// tsconfig
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
        "app/*": ["./src/app/*"],
        "config/*": ["./src/app/_config/*"],
        "environment/*": ["./src/environments/*"],
        "shared/*": ["./src/app/_shared/*"],
        "helpers/*": ["./src/helpers/*"],
        "tests/*": ["./src/tests/*"],
        "assets/*": ["./assets/*"],
    },
  },
  // Test 1
  "include": ["assets/*"],
  // Test 2
  "include": ["**/*.png"],
}
```

[TypeScript의 Include](https://www.typescriptlang.org/tsconfig#Include)를 이용해 TypeScript가 적용되는 범위를 지정해도 제대로 적용되지 않았고 똑같은 에러 메시지가 출력 되었다.

## 성공 사례

```javascript
// d.ts
declare module 'assets/*.png' {
  const value: any;
  export = value;
}
```

[Stackoverflow - declare module](https://stackoverflow.com/questions/51100401/typescript-image-import/51163365#51163365)에서 `d.ts`에서 직접 Modul을 선언하는 방법을 제시했고 위와 같이 문구를 집어넣었다. 그 결과 TypeScript에서 찾지 못하던 Reference error가 해결 되었다.

## 고찰

그러나 이 부분은 정확히 말해 해결이라고 볼 수는 없을 것이다. `d.ts`에서 선언된 이 부분은 `assets/*.png`에 대해서 **'내가 해당 파일들이 있음을 보장'**하며, 해당 File들은 어떤 Type을 가져야 함을 기술하는 것이기 때문이다. 그렇기에 fallback을 적용하는데 있어 다음의 두 가지가 선행 되어야 한다.

1. `babel.config.js` File이 있는 경우 `plugins`의 `module-resolver`, 그 안의 `root`와 `alias` 부분이 실제적으로 적용되어 build 할 때에도 reference check가 확실해야만 한다.

2. `tsconfig.json`에서 `babel.config.js`와 동일한 relative path에 대한 reference check가 존재해야 한다.

위의 두 가지가 제대로 적용됐음에도 module error가 발생할 때에만 fallback을 사용할 때 기대하는 Performance가 나올 것이다. 만약 위의 두 가지가 선행되지 않으면 type check는 문제없는데 build 에러가 나게 되므로 유의한다(Bug가 확인 되지 않으므로 찾는데 시간을 허비할 수 있다는 의미).

또한, 상기의 declare module은 사용하는 Library나 기존 components의 hard한 type 지정 및 타입 에러 수정에 응용 될 수 있다. 다음은 react-navigation에서 `navigate` Method type이 존재하지 않아 발생하는 Type error를 수정하는 방법이다.

> 아래의 방법을 통해 매번 `useNavigation<StackNavigationProp<any>>()` 할 수고를 덜 수 있게 된다.

```javascript
// d.ts
// navigate type error가 발생했을 때 수정 방안
import '@react-navigation/native';
import { useNavigation } from '@react-navigation/native';
import { StackNavigationProp } from '@react-navigation/stack';

declare module '@react-navigation/native' {
  export function useNavigation(): StackNavigationProp<any>;
}
```

## 참조

1. [Stackoverflow](https://stackoverflow.com/questions/51100401/typescript-image-import/51163365#51163365)

2. [Typescript](https://www.typescriptlang.org/tsconfig#Include)

## 저작권

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="크리에이티브 커먼즈 라이선스" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />이 저작물은 <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">크리에이티브 커먼즈 저작자표시-동일조건변경허락 4.0 국제 라이선스</a>에 따라 이용할 수 있습니다.
