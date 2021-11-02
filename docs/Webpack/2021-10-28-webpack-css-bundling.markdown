---
layout: post
title:  "Webpack css bundling에 관해"
date:   2021-10-28 20:08:05 +0900
parent: Webpack
categories: Webpack
nav_order: 2
---
## Webpack css의 bundling은 어떤 형태로 이루어져야 할까?
최근 Babel을 공부하다 자연스레 Webpack을 공부하게 되었다. Webpack을 공부하며 Javascript, CSS, HTML, MEDIA 파일이 각각 통합된다는 것을 알게 되었고 여기서 고민이 생겼다. 

#### 만약, CSS 파일이 하나로 통합된다면, 각각의 CSS 파일에서 선언한 Style 값들이 하나로 통합될 테고, 각각의 고유 `ClassName`이 존재하지 않는 이상 Style 값이 섞여 예상하는 UI가 구현되지 않을 수 있겠지?

```js
// a.js
p {
    color: blue; 
}

// b.js
p {
    font-size: 2rem;
}

// Bundling 후 적용될 p 태그
p {
    color: blue;
    font-size: 2rem;
}
```

*a*라는 js 파일에서 `p` 태그에 색을 입히고 *b*라는 Js 파일에서 `p` 태그의 글씨 크기를 조정했다고 하면 하나로 bundling된 css에서는 Module로 구분되지 않고 하나로 통합된다. 즉, `p = 각각의 모듈 p 태그 Style의 합`라고 할 수 있다.

#### 그렇다면 CSS를 하나로 통합해 Bundling 하는 것은 좋지 않은 선택일까?

그렇지 않다. 만약, 소규모의 프로젝트라면 CSS를 하나로 통합해 Bundling 할 수도 있다. 그러나 하나로 Bundling 하겠다면 각각의 고유 `ClassName`을 통합적으로 관리할 수 있는 rule을 만들어야 한다. 또, 하나로 Bundling 된 이 거대한 Chunk는 브라우저 입장에서도 읽는데 시간이 걸릴 것이기에 당연히 **로딩 시간이 길어질 것**이라는 점을 생각해야 한다.

실제로 **Create React App**에서는 하나의 CSS 파일로 Bundling이 되고 각각의 `ClassName`에 따라 구분이 된다. 아마 이런 부분을 생각하면 Framework는 전반적으로 특정 Configuration에서만 작동하는 형태가 되어서는 안 되기 때문에 성능을 어느 정도 포기하는 경우도 있다고 생각한다. 그래서 어느 기업들은 직접 Framework를 만들어 쓰기도 하는 듯 한데 전문가라면 분명 더 나은 Framework 퍼포먼스를 보여줄 수 있는 툴을 만들 수 있을 것이다.

#### 그럼 나누어서 Bundling 하는 게 이점이 있다는 건 알겠어, 그럼 어떻게 얼마나 무엇을 나눠야 할까?

이 문제가 내 고민이었다. Page 단위로 Chunk를 만들어야 할까? 아니면 모듈 단위로 Chunk를 만들어야 할까? 무엇을 나누어야 할까? Stackoverflow나 여러 사이트를 많이 찾아보니 **그건 개발자의 선택 영역에 있다고 했다.** 각자 관리하고자 하는 단위가 다를 테니 맞는 대답이지만 초심자에게는 너무 두루뭉술한 얘기처럼 느껴졌다.

대부분의 질문들은 외부 Style과 내부 Style을 관리하는 것에 대한 물음들이었다. 개발하는 어플리케이션 내부의 CSS를 분산하여 Bundling하게 되면 개별 Bundling된 파일이 결국에는 내부 `global css` 단위에서 통합되게 되고 즉, css 파일은 중복되어 생산된다.(global css의 통합 css에서 존재하는 css가 분리된 css 파일에서도 존재)

이것은 CSS의 Tree구조(transmission structure)와도 연관이 있기 때문에 global css가 하부의 css와 중복되어 Bundling 되는 것은 피할 수 없는 문제 같다.

> 찾아본 바에 따르면 Javascript 파일이 child module과 결합된다고 하더라도 child module을 load 한 뒤에 parent 파일이 완전히 load 된다고 한다. 
> 뿐만 아니라 chunk 단위로 나누어지게 되면 Webpack은 parent에서 중복되는 chunk가 발견된다고 하더라도 instance를 여러 개 생성하는 것이 아니라 하나만 생성한다고 한다.

이런 것들을 고려해보았을 때, chunk를 너무 많이 나누게 되면 bundling 폴더의 크기가 증가할 것이라고 생각한다. 따라서 페이지 단위 혹은 Vendor(바뀌지 않는 module: node_modules)와 사용자 Javascript 파일 내지 css 파일로 나누는 것이 loading enhancement에도 좋은 영향을 주고 FOUC(Flash Of Unstyled Content)를 방지하며 bundling 파일도 적절하게 caching(캐싱: splitChunks를 통한 캐싱 방법에 대해 좀 더 알아볼 것)할 수 있지 않나 생각한다. 그리고 이러한 부분들은 Load velocity나 측정치를 보며 직접 확인해야 할 것 같다.