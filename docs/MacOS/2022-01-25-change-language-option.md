---
layout: post
title: Mac OS 한/영키 바꾸기
date: 2022-01-25 17:25:00 +0900
parent: MacOS
categories: MacOS, setting, 한영키
nav_order: 1
comments: false
---

*2022-01-25 17:25 작성*

# Mac OS 한/영키 바꾸기

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

## <kbd>caps lock</kbd>한/영키 불편함

맥북으로 한/영키를 전환하는데 있어서 기본적으로 <kbd>caps lock</kbd>를 사용한다. 한 번 누르면 바로 변환되면 좋겠지만 항상 바로 변환되지 않아 다시 한 번 더 눌러야만 한다. 한 두 번이야 그냥 넘어가지만 이게 계속 되다보면 은근히 스트레스가 생긴다.

## karabiner를 통한 한/영키 전환의 껄끄러움

인터넷에 떠도는 방법 중 karabiner를 통해 cmd 키를 한/영키로 바꾸는 조작법이 널리 퍼져 있는데 키보드 권한을 받아내는 부분에서 뭔가 껄끄러움을 없앨 수가 없었다. 그래서 나는 이 방법을 시도했다가 앱을 삭제했다.

## 내부 설정을 통해 자신의 키 설정

그래서 나는 내가 키를 지정해 사용하기로 마음 먹었다. 

1. `시스템 환경설정 - 키보드 - 입력 소스`에 들어간다.
2. '두벌식'과 'ABC'가 있을 텐데 이 두 개를 각각 클릭해 'Caps Lock 키로 ABC 입력 소스 전환'을 체크 해제한 후, '문서의 입력 소스로 자동으로 전환'을 체크한다.
3. 이제 `입력소스` 탭이 아닌 `단축키` 탭으로 넘어가서 좌측 탭에서 '입력소스'를 클릭한다.
4. 우측 탭의 입력 메뉴에서 다음 소스 선택을 선택하고 <kbd>Tab</kbd>을 누르거나 키 설정 부분을 클릭한 뒤, 자신이 원하는 키를 누른다. (본인의 경우, cmd + space 보다 cmd + a가 동선이 편해 그렇게 설정함)
5. 중복 되는 키가 있을 경우 주의 표시가 뜨고, 중복되지 않도록 않게 바꿔준다.

p.s. cmd + a는 전체 선택이므로 중복이 있어 이전 입력 소스 전환 키인 ctrl + space 키로 바꾸어 쓰기로 결정, 현재 그럭저럭 적응해 가는 중이다.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
