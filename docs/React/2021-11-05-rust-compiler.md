---
layout: post
title: "Rust 컴파일러와 Framework의 방향에 대한 생각"
date: 2021-11-05 23:20:00 +0900
parent: React
categories: react, rust, nextjs, compiler
nav_order: 2
comments: false
---

*2021-11-05 23:20 작성*

# Rust 컴파일러와 Framework의 방향에 대한 생각
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

## 1. Babel

최신 ES를 오래된 브라우저에서도 적용할 수 있는 Transpiler **Babel**을 통해 브라우저 환경을 세세하게 고려하지 않고도 개발을 하게 된 점에서 **Babel**은 개발에 있어서 큰 기여를 했다. 하지만 여전히 JavaScript의 속도 한계를 가지고 있었고 그런 가운데 Rust의 기능이 부상했다. Rust는 Lower-level 코드로 JavaScript보다 빠른 속도로 처리를 할 수 있고 JavaScript와 Rust 간에 Transcompiling이 가능하다면 성능을 향상시킬 수 있을 것이다. 

## 2. SWC 

그리고 최근 Next JS에서는 **SWC**라는 Transpiler를 도입했다. Next JS는 **SWC** 기반의 Transcompiler 위에 Rust Compiler를 도입했고 최대 3배 빠른 refresh 그리고 최대 5배 빠른 builds 퍼포먼스를 선보였다.

## 3. Rome

Babel을 만든 Sebastian은 현재 Babel이 아닌 **Rome**이라는 Toolchain을 개발하고 있다. Babel의 주요 Contributor들이 **Rome** 개발에 몰두하고 있고 linter, compiler, bundler 등 다양한 역할을 수행하는 Toolchain으로 만들기 위해 개발에 박차를 가하고 있다. 그리고 **Rome** 또한 최근에 Rust로 다시 쓰여졌는데 이것을 고려할 때 **Rome**도 강한 퍼포먼스를 보여줄 것으로 생각된다. 이러한 발전 양상을 봤을 때, 앞으로  다른 프레임워크에서도 Rust를 이용한 Compiler를 도입할 것으로 보이고 수 년 내에 Javascript를 이용한 개발에 큰 성능향상이 있을 듯 하다.

## 4. 생각

앞으로 더 나은 개발 환경, 더 빠른 개발 환경에서 일한다는 것은 기쁜 일이지만 Rust compiler에 대해 이해하고 Rust에 대비할 필요성이 있을 듯 하다. 최근에 잠시 취업준비로 Rust를 중지하기는 했지만 이내 곧 다시 시작해야만 한다.

{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}