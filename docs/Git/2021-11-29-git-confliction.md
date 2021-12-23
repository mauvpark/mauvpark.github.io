---
layout: post
title: "Git 병합 충돌(Git merge confliction)"
date: 2021-11-29 20:42:00 +0900
parent: Git
categories: git, merge, confliction
nav_order: 2
comments: false
---

*2021-11-29 20:42 작성*

# Git Conflict Resolution
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

*본 테스트는 다음 [리포지토리](https://github.com/mauvpark/git-conflict-resolution)에서 확인가능합니다.*

## 설명
Branches 단위에서 master로 merge 될 때, conflict가 발생됐음을 가정하고 수정하는 테스트입니다.

## 구조

```js
// main branch: ingredients.txt
1 tbsp cilantro
2 avocados
1 lime
1 tsp salt

// like-cilantro branch: ingredients.txt
3 tbsp cilantro
2 avocados
1 lime
1 tsp salt

// dislike-cilantro branch: ingredients.txt
0.5 tbsp cilantro
2 avocados
1 lime
1 tsp salt
```

## 충돌 (Conflict)
main(=master) branch에서 새로 만들어진 두 개의 새로운 branch에서 `tbsp cilantro`의 값을 바꾸고 `git merge 브랜치명`을 실행하게 되면 두 번째 브랜치 병합에서 **충돌**이 발생한다.

### Conflict(충돌)은 왜 발생하는가?
만약, 충돌이 없다고 가정해보자.

```js
const IMPORTANT = 10;

function calculate () {
    // 0-IMPORTANT 사이의 난수 생성
    return Math.random() * (IMPORTANT - 0) + 0;
}

calculate();
```

Conflict 기능은 존재하지 않으며 '망한게임즈'라는 회사에서는 팀원 간 업무 커뮤니케이션이 안 된다고 가정해보자. 특정 기능을 구현하는 동료(A)가 최초의 `IMPORTANT` 값을 다른 브랜치에서 업무상의 이유로 변경하게 됐고, 그것을 모르는 다른 동료(B)가 이를 모르고 자신의 업무를 위해 동일값을 다른 브랜치에서 변경해 작업했다. 그 날 저녁, 서로의 작업이 완료되고 부장(C)은 각자 작업한 브랜치를 모두 병합했다. 다음 날 기능이 잘 작동하는지 알기 위한 회의에서 기능을 실현해보니 B의 기능은 정상적으로 실행이 되고 A의 기능은 완전히 이상한 형태로 작동해 부장과 동료 A는 사장에게 꾸지람을 들었다.

이 상황에서 서로 다른 `IMPORTANT`값이 병합됨에 따라 업무가 이상하게 처리됐다는 것을 알 수 있다. **Conflict**가 존재했다면 이 상황은 미연에 방지가 될 수 있었을 것이다.

### 충돌 시 나타나는 내용
> `git merge 브랜치명`: *충돌 (추가/추가): ingredients.txt에서 합병 충돌이 발생했습니다.<br/>자동 병합: ingredients.txt<br/>자동 병합이 실패했습니다. 충돌을 바로잡고 결과물을 커밋하십시오.*

## 해결
```console
<<<<<<< HEAD
* 2 tbsp cilantro
=======
* 0.5 tbsp cilantro
>>>>>>> dislike-cilantro
```

위의 중복되는 2개의 값에서 적합한 것을 고르고 `git add 충돌된파일명(확장자포함)`을 실행한 후에 `git status`에서 문제없음을 확인한다. 최종적으로 `git commit`을 실행한다.

## main branch가 아닌 새로운 브랜치 간 병합
main branch에서는 최초 병합에서 성공적으로 병합이 됐던 것과 달리 새로 만들어진 브랜치 간 병합에서는 최초 병합임에도 서로 Conflict가 발생한다.

```js
<<<<<<< HEAD
const IMPORTANT = 4;
const Hello = "World!";
=======
const IMPORTANT = 6;
>>>>>>> random-6

function calculate() {
	// 0-IMPORTANT 사이의 난수 생성
	return Math.random() * (IMPORTANT - 0) + 0;
}

calculate();
```

## 관련 명령어
- `git checkout 브랜치명`: 해당 브랜치로 이동
- `git status`: 현재의 깃 상태와 해결방법을 보여줌

```console
$ git status

On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   ingredients.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

- `git merge 목표브랜치명`: 현재 브랜치에서 목표브랜치의 파일을 가져와 병합함

```console
// conflict 발생 시 다음과 같은 경고가 나옴.
$ git merge dislike-cilantro

Auto-merging ingredients.txt
CONFLICT (content): Merge conflict in ingredients.txt
Automatic merge failed; fix conflicts and then commit the result.
```

- `cat 파일명(확장자포함)`: `git merge`에서 발생한 conflict 내역을 보여줌
- `git diff 브랜치1 브랜치2`: `브랜치1(a)`과 `브랜치2(b)`를 비교함
- Conflict 발생 후 `git diff`: Conflict 발생 부분만 보여줌

```console
diff --cc ingredients.txt
index 83f2f94,2f60e23..0000000
--- a/ingredients.txt
+++ b/ingredients.txt
@@@ -1,4 -1,4 +1,8 @@@
++<<<<<<< HEAD
 +* 2 tbsp cilantro
++=======
+ * 0.5 tbsp cilantro
++>>>>>>> dislike-cilantro
  * 2 avocados
  * 1 lime
  * 1 tsp salt
```