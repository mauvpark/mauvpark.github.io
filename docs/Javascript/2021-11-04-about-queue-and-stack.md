---
layout: post
title: "Queue와 Stack의 개념과 예시"
date: 2021-11-04 19:37:00 +0900
parent: Javascript
categories: javascript, queue, stack, heap
nav_order: 2
comments: true
---

# Queue와 Stack의 개념과 예시(+ Heap)

## 1. 도입

**Queue**와 **Stack** 개념은 단순하다. 그러나 두 개념을 어디에 사용하면 좋을지에 대해서 생각해보는 것은 나중의 응용력을 강화할 수 있다. 그렇기에 두 개념을 정리하고 좋은 예시를 써 두기로 했다.

## 2. Queue와 Stack의 개념

<p align="center">
  <img src="../../assets/js/the_javascript_runtime_environment_example.svg" width="350" title="queue & stack example">
</p>

**Queue**는 **_FIFO(First In First Out)_**으로 정리할 수 있다. 예를 들어 가로로 길게 늘여진 통 안에 구슬을 차례로 넣고 입구에 강한 바람을 넣으면 반대쪽으로 구슬이 쏟아져 나올 것이다. 당연히 그 순서는 가장 먼저 넣은 구슬이 가장 먼저 나올 것이고, 가장 마지막에 넣은 구슬이 마지막에 나올 것이다.

> _처음 넣은 것이 가장 먼저 나온다. 그것이 **Queue**이다._

**Stack**은 **_LIFO(Last In First Out)_**으로 정리할 수 있다. 흔히 레스토랑의 접시를 쌓아올리는 것으로 예를 들지만 총으로 예시를 들어볼까 한다. 탄알집에 탄을 넣을 때(**PUSH**) 우리는 가장 먼저 넣은 탄알이 가장 안쪽으로 들어가게 된다. 그리고 가장 늦게 넣은 탄알이 당연히 가장 위로 올라온다. 그리고 탄알집을 총에 장전하게 되어 발포(**POP**)하면 가장 늦게 넣은 탄알이 가장 먼저 발사될 것이다.

> _가장 늦게 넣은 Element가 가장 먼저 나온다. 그것이 **Stack**이다._

예시 그림에 **Heap**이 있으니 추가 설명하자면 **Heap**은 **Queue, Stack**과는 다른 방식으로 Elements를 저장한다. **Heap**에 대한 간단한 설명을 하자면 레스토랑에서 테이블 번호를 정해놓고 테이블에 손님이 앉는 구조를 상상하면 되는데, 이 때 테이블 번호는 **_Index_**가 되고 테이블에 앉는 손님은 각각의 Element가 된다고 볼 수 있다. **Queue, Stack**과 달리 저장할 때 테이블 번호(**_Index_**)가 필요하기 때문에 속도 면에서 **Queue, Stack**이 더 나은 퍼포먼스를 보여준다고 볼 수 있다. 반면 **Heap**은 느리지만 데이터를 능동적으로 관리할 수 있고, 메모리 사이즈에 구애 받지 않는 등의 장점이 있다. (자세한 내용은 <https://www.guru99.com/stack-vs-heap.html> 참고)

## 3. Queue와 Stack의 예시

상기한 **Queue**와 **Stack**의 예시는 이해를 돕기 위한 예시고, 실제 컴퓨터에서의 도움이 될 수 있는 예시를 들어보려고 한다.

먼저, **Queue**는 Input Stream, 즉 키보드나 네트워크처럼 선입력된 값들이 먼저 처리되는 것으로 설명할 수 있다. 또, 프린터에서 파일들을 출력하게 되면 인쇄를 지정한 파일의 시간적 순서대로 출력이 되는 것을 볼 수 있는데 이것 또한 Queue의 예시라고 볼 수 있다.

다음으로, **Stack**은 브라우저의 뒤로가기, 앞으로가기 기능을 생각하면 좋다. 브라우징한 페이지들은 탄알집의 탄알처럼 쌓이게 되고 가장 나중의 탄알(페이지)이 뒤로가기 페이지에 할당되게 되는 것이다.

## 4. 참고 사이트

A. <https://www.guru99.com/stack-vs-heap.html> <br/>
B. <https://cse.buffalo.edu/~shapiro/Courses/CSE116/notes10.html> <br/>
C. <https://developer.mozilla.org/ko/docs/Web/JavaScript/EventLoop>

{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}