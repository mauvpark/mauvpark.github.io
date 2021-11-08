---
layout: post
title: "NPM Packging"
date: 2021-11-08 17:13:00 +0900
parent: NodeJs
categories: node-js, npm
nav_order: 2
comments: true
---

# NPM Packaging

## Pre-requisite

1. 최신 Node JS
2. NPM cli

## How-To

1. package.json
- `npm init -y` package.json 초기화
- `name`: 폴더 이름과 같고 publishing name이 됨
- `main`에서 패키징 시작점 정하기
- `type: module` import, export 및 기타 유용한 다른 syntax를 사용할 수 있게 함(Node version 14 이상) 

2. main file
- e.g. index.js

```js
const randomNumberGenerator = (min = 0, max = 100) => {
    return Math.round((Math.random() * (max - min) + min));
}

export default randomNumberGenerator;
```

3. terminal `npm login`: username, password

4. `npm whoami`: check logged in

5. root folder **README.md** 파일 생성해 설명
- 설명
- 설치 방법
- 사용 방법 (Params, import 방법 등) 
- 라이센스 등

6. `npm publish`: npm 사이트 퍼블리싱

7. npm site profile에서 확인

8. 실행 가능한지 `npm install 모듈이름` (테스트 시 package.json 내 `type:module` 설정 필요: import 사용)

## 참고
1. [JavaScript Mastery](https://youtu.be/8FziherTC8M)
2. [Published example](https://www.npmjs.com/package/mauv_rng)

{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}