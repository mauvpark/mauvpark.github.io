---
layout: post
title: "Git Rebase"
date: 2022-01-10 18:41:00 +0900
parent: Git
categories: git, rebase
nav_order: 3
comments: false
---

*2022-01-10 18:41 작성*

# Git Rebase

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

Git에서 한 branch를 다른 branch와 병합하는 방법에는 두 가지가 있다.

1. Git Merge

2. Git Rebase

두 명령어를 사용해서 얻을 수 있는 최종 결과물은 같으나 세부적인 부분에 있어서 약간의 차이가 있다.

## Git Merge를 이용한 병합

Git에서 두 branch를 병합하는 가장 쉬운 방법은 `git merge` 명령을 사용하는 것이다. 이 방법은 두 branch의 마지막 커밋 두 개와 공통 조상을 사용해 새로운 커밋을 만들어 낸다.

이 방법을 통해 두 branch를 병합하게 되면 서로 분기된 branch가 git history 상에 남게 된다.

이 분기된 branch가 git history에 남지 않고 마치 하나의 branch에서 작업된 것처럼 git history를 깔끔하게 만들 수 있는데 이 때 `git rebase`를 사용할 수 있다.

## Git Rebase를 이용한 병합

Git에서 두 브랜치의 최종 결과물만을 가지고 병합하는 `git merge`와 달리 `git rebase`는 변경사항들을 차례로 수행하며 병합한다는 점에서 git history를 깔끔하게 한다. rebase를 활용했을 때 얻게 되는 장점은 다음과 같다.

1. master branch에서 수행된 변경사항을 먼저 적용한 뒤에 본인이 작성한 커밋 내용을 적용하게 된다. 이는 즉, 협업으로 커밋된 master branch를 우선시 한다는 의미이므로 협업에 있어서 긍정적인 점을 보여준다.

2. rebase로 작성된 git history는 마치 하나의 branch에서 작성된 깔끔한 history를 구성한다.

## 주의사항

Git Rebase를 사용하는데 있어 한 가지 유념해야 할 것이 하나 있다. **이미 공개 저장소에 Push 한 커밋을 Rebase 하지 마라**는 것인데, 관련 Docs만 보고서는 한 번에 이해가 되지 않아 정리 용도로 남겨 둔다.

1. A라는 개발자가 작업하고 있는 master branch에서 B라는 개발자가 `git clone`을 해서 작업을 진행하고 commit과 merge를 진행한 뒤에 서버에 push했다.

2. 그런데 B라는 개발자는 merge를 원하지 않아 다시 rebase 하기로 결정한다. 그래서 서버에 새로운 히스토리를 덮어 씌우기 위해 `git push --force` 명령을 사용한 후에 Fetch, rebase를 수행하고 push를 진행했다.

3. 이렇게 되면 기존의 master branch는 rebase를 수행하기 전 push 되었던(더이상 필요하지 않은) branch 뿐만 아니라 rebase 된 브랜치의 영향을 받게 된다. 즉, git history가 꼬이게 되어 더이상 참고하지 않아도 되는 branch의 코드 영향을 받아 엉망이 될 수 있게 되는 것이다.

## 당부사항

위와 같은 이유로 만약 git history가 꼬이게 될 수 있기에 다음의 사항을 유념한다.

1. Rebase와 Merge는 병합에 대한 방법의 차이다. 각각의 개발 상황에 따라 서로 다른 명령을 사용할 수 있고 어느 하나가 부족하거나 나쁘다는 의미가 아니다. 따라서 위의 [주의사항](#주의사항)과 같은 위험을 감수하면서까지 진행할 필요는 없다.

2. branch를 변경해 기존의 branch에서 업데이트 된 사항을 받아올 때 `git pull`을 사용하게 된다. 이 때, `git pull --rebase`를 사용해 origin branch의 내용을 먼저 적용하고 본인의 커밋 내용을 적용할 수 있다. 항시적으로 `--rebase`를 허용하기 위해서는 `git config --global pull.rebase true`를 사용한다.(branch 관리자가 아닐 경우, 주로 사용할 기능)

3. `git rebase <basebranch> <topicbranch>`를 통해 직접 조작할 경우(예를 들어, branch 관리자일 경우), 해당 명령어를 수행한 뒤에 꼭 Fast-forward를 통해 수정사항이 master branch에 적용되도록 한다. (Github - Rebase and merge)

```
<!-- Fast-forward 예시 -->
$ git checkout master
$ git merge server

<!-- Fast-forward가 끝난 뒤 이하의 branch가 필요없다면 삭제한다. -->
$ git branch -d client
$ git branch -d server
```

## 참고

[git-scm](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)

## 라이센스

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/3.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/">Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License</a>.