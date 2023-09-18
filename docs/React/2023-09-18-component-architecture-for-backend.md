---
layout: post
title: "백엔드 개발자를 위한 공통 컴포넌트 구조 설계"
date: 2023-09-18 12:46:00 +0900
parent: React
categories: react, architecture
nav_order: 2
comments: false
---

*2023-09-18 12:46 작성*

# 백엔드 개발자를 위한 공통 컴포넌트 구조 설계
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

## 문제
백엔드 개발자가 React와 CSS에 대한 지식이 얕다고 가정했을 때, 공통 컴포넌트 개발을 위해 어떤 노력을 해야 할까?

## 문제 해결을 위한 조건
1. 백엔드 개발자가 CSS를 수정할 일이 없도록 해야 한다.
2. 주어진 컴포넌트가 이해하기 쉬워야 한다(직관적인 컴포넌트 이름과 Property의 이름).
3. 타입 혹은 JSDOC을 이용해 참고가 쉬워야 한다.
4. 폴더 구조가 이해하기 쉬워야 한다.
5. 백엔드에서 데이터만 넣어 사용할 수 있도록 적절히 chunk화 되어야 한다.

## 문제 해결 방안
1. `Atomic design pattern`을 사용해 공통 컴포넌트를 분리한다(`atoms, molecules, origanisms, templates`).
2. 특정 페이지에서 사용하는 컴포넌트는 Atomic folder에 저장하지 않고 컴포넌트의 페이지 폴더 단위로 만들어 구분한다(`Components/Login/Login.tsx`).

## 정리
1. 공통 컴포넌트의 `organisms` 단위 혹은 `templates`는 chunk 형태의 컴포넌트를 제공해 관련 데이터만 넣으면 UI가 자동으로 출력 되므로 백엔드 개발자 입장에서 접근이 용이하고, `atoms`와 `molecules`를 활용해 **데이터만으로** 블록을 짜맞출 수 있다.
2. 특정 페이지에서 유일하게 사용하는 컴포넌트는 컴포넌트의 페이지 폴더에 저장되므로 유일함을 보장할 수 있고, 따라서 백엔드 개발자는 공통 컴포넌트와 특정 페이지 컴포넌트를 구분할 수 있게 된다.

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
