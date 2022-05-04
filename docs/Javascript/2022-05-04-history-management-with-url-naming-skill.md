---
layout: post
title: URL 이름짓기(naming skill)를 통한 history 관리
date: 2022-05-04 16:05:00 +0900
parent: Javascript
categories: javascript, url, naming, history, flow
nav_order: 3
comments: false
---

*2022-05-04 16:05 작성*

# URL 이름짓기(naming skill)를 통한 history 관리
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

## URL 이름짓기(naming skill)는 왜 필요한가?

React 프로젝트를 경험해본 사람들이라면 react-router-dom을 한 번 쯤은 사용해봤을 법 하다. 화면 간의 이동은 물론, url을 통해 원하는 값을 넘겨줄 수도 있는 등 다양한 편의성을 제공하기 때문에 내게도 아주 유용한 라이브러리였다. 이 라이브러리를 사용할 때, 라우팅을 위한 컴포넌트 주소값을 할당하고 이 주소값을 바탕으로 다양한 조작을 시행한다.

그리고 이 조작에는 단순히 화면을 이동하는 것 뿐만이 아닌 페이지 간 상하 관계를 만드는 흐름(flow)도 포함한다.

숙련된 개발자들은 이미 이 중요성에 대해 잘 알고 있겠지만, 나와 같은 신입 내지 주니어 개발자들에게는 이런 부분이 생소하게 느껴진다. 지금 당장 개발하는 상황에서 상하 관계에 대해 생각해볼 수 있는 기회가 적기 때문에 충분한 고민 끝에 적용하기도 사실 어렵다고 생각한다. 그렇기에 통일되지 않은 url로 인해 flow 구성이 쉬워질 것도 어려워지게 된다.

만약 어느정도 개발이 지난 시점에 상하 flow 조정을 하게 됐다고 생각해보자. 물론 이 때, url에 통일감이 전혀 없다고도 해보자.

```
http://localhost:3000/main

http://localhost:3000/profile

http://localhost:3000/chat

http://localhost:3000/chatroom

http://localhost:3000/shop

http://localhost:3000/seller
```

위에서 상하 관계를 짜야 한다면 history pathname에 따라 강제 할당을 할 수도 있을 것이다. 불가능한 얘기도 아니고 가능하다. 그러나 다음에 특정 페이지에 조건이 생겨 다른 페이지로 넘겨야 하는 등의 동작이 생긴다면 일일히 코드 분석을 해야 하기에 필요한 시간이 두 배, 세 배 그 이상이 될 것이다.

단적인 예를 들어, 하나의 폴더 안에 숙제 문서, 일기 문서, 과업 문서, 회사 문서, 계약 문서 등을 전부 집어 넣는다고 해보자. 내가 여기서 필요한 문서를 찾는데 걸리는 시간이 불필요하게 커질 것이다. 그래서 우리는 각각의 문서를 숙제 폴더, 일기 폴더, 과업 폴더, 회사 폴더, 계약 폴더 등으로 나누어 저장한다. 그렇게 되면 폴더 구조가 한 눈에 들어오고 필요한 문서를 찾는데 걸리는 시간이 훨씬 짧아질 테니 말이다.

## 파악할 수 있는 형태의 URL 이름짓기(naming skill)

위의 예에서 `chat`과 `chatroom`, 그리고 `shop`과 `seller` 정도는 다음과 같이 묶을 수도 있을 것이다.

```
http://localhost:3000/chat

http://localhost:3000/chat/chatroom

http://localhost:3000/shop

http://localhost:3000/shop/seller
```

우리가 흔히 정리하는 폴더와 같이 `chatroom`의 상위 폴더는 `chat`이고, `seller`의 상위 폴더는 `shop`인 것이다. 그렇기에 flow를 짤 때에도 구조를 파악해 상위 폴더로 간단하게 강제할 수 있게 된다.

예를 들어, `chat` 폴더 이하의 모든 파일들은 `chat` 폴더로 돌아오게 할 수 있고, `shop` 이하의 모든 파일들은 `shop`으로 돌아오게 할 수 있게 되는 것이다. 그리고 이러한 flow를 짜는데 있어 처음 소개했던 방식에서는 관련된 모든 url을 하드 코딩으로 식별하여 돌아와야 하는 폴더 지점을 할당해야 했지만, 이러한 방식에서는 `chat` 혹은 `shop` 이하의 모든 폴더 혹은 파일을 모두 해당 지점으로 돌아오게 하면 되므로 코딩이 훨씬 보기 쉽고 간편해진다.

코딩을 직접 할 때는 시간에 쫓겨서 바빠서 이런 부분들을 놓치고는 한다. 그러나, 잘 정리된 폴더 구조는 색인도 편하다는 사실을 인지하면 좋을 것 같다.

## 라이센스

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="크리에이티브 커먼즈 라이선스" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />이 저작물은 <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">크리에이티브 커먼즈 저작자표시-동일조건변경허락 4.0 국제 라이선스</a>에 따라 이용할 수 있습니다.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
