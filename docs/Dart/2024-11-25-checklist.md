---
layout: post
title: (요약)효과적인 Dart 코딩을 위한 체크리스트
description: Dart 및 기타 언어에서 사용할 수 있는 좋은 코딩을 위한 체크리스트를 소개합니다.
date: 2024-11-25 00:00:00 +0900
parent: Dart
categories: dart,checklist
nav_order: 3
comments: false
---

_2024-11-25 작성_

# (요약)효과적인 Dart 코딩을 위한 체크리스트

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

_[원문링크](https://dart.dev/effective-dart)_

> 번역자 주: 본문은 다트 언어를 사용해 효과적으로 코딩하는 방법에 대한 것을 다룹니다. 그러나 모든 프로그래밍 언어에 보편적으로 적용할 수 있는 부분도 있기에 참고하면 좋겠습니다.


**AVOID**: 피해야 할 것
**CONSIDER**: 고려해봐야 할 것
**DO**: 해야 할 것
**DON'T**: 하지 말아야 할 것
**PREFER**: 하면 좋은 것

## 스타일

### 식별자

#### DO `UpperCamelCase`를 사용해 이름을 지으세요(클래스, 타입 정의, 타입 파라미터 등).

```dart
class SliderMenu { ... }

typedef Predicate<T> = bool Function(T value);
```

#### DO Extension도 `UpperCamelCase`를 사용하세요.

```dart
extension MyFancyList<T> on List<T> { ... }

extension SmartIterable<T> on Iterable<T> { ... }
```

#### DO 패키지, 디렉토리, 소스 파일은 `lowercase_with_underscores` 형식으로 이름 지으세요.

```
my_package
└─ lib
   └─ file_system.dart
   └─ slider_menu.dart
```

#### DO import 접두어는 `lowercase_with_underscores` 형식으로 이름 지으세요.

```dart
// as 이하 문구
import 'dart:math' as math;
import 'package:angular_components/angular_components.dart' as angular_components;
import 'package:js/js.dart' as js;
```

#### DO 다른 식별자는 `lowerCamelCase`를 사용하세요(클래스 멤버, 상위에 정의된 것, 변수, 파라미터 그리고 named 파라미터 등).

```dart
var count = 3;

HttpRequest httpRequest;

void align(bool clearItems) {
  // ...
}
```

#### PREFER 변하지 않는 변수는 `lowerCamelCase`를 사용하세요.

```dart
const pi = 3.14;
const defaultTimeout = 1000;
final urlScheme = RegExp('^([a-z]+):');

class Dice {
  static final numberGenerator = Random();
}
```

#### DO 두 글자보다 긴 축약어와 두문자어(예시: UN)는 대문자를 사용하세요.

```dart
// Longer than two letters, so always like a word:
Http // "hypertext transfer protocol"
Nasa // "national aeronautics and space administration"
Uri // "uniform resource identifier"
Esq // "esquire"
Ave // "avenue"

// Two letters, capitalized in English, so capitalized in an identifier:
ID // "identifier"
TV // "television"
UI // "user interface"

// Two letters, not capitalized in English, so like a word in an identifier:
Mr // "mister"
St // "street"
Rd // "road"
```

#### PREFER `_`, `__` 등 사용하지 않는 콜백 파라미터는 개수에 따라 underscore로 표기하세요.

```dart
futureOfVoid.then((_) {
  print('Operation complete.');
});
```

#### DON'T private이 아닌 식별자에 대해 underscore을 사용하지 마세요(다트는 `_`를 private으로 인식).

#### DON'T 접두문자를 사용하지 마세요. 다트는 접두사에 힌트를 주지 않아도 타입, 스코프 등을 추적합니다.

```dart
/// Good 🙆
defaultTimeout

/// Bad 🙅
kDefaultTimeout
```

#### DON'T 라이브러리에 직접적인 이름을 짓지 마세요.

```dart
/// Good 🙆
@TestOn('browser')
library;

/// Bad 🙅
library my_library;
```

### 순서

#### DO `dart:`와 관련된 import문은 다른 import문 앞에 두세요.

```dart
import 'dart:async';
import 'dart:html';

import 'package:bar/bar.dart';
import 'package:foo/foo.dart';
```

#### DO `package:`와 관련된 import문은 상대 경로 import문보다 앞에 두세요.

```dart
import 'package:bar/bar.dart';
import 'package:foo/foo.dart';

import 'util.dart';
```

#### DO export문은 import문과 별개의 구역에 정의하세요.

```dart
import 'src/error.dart';
import 'src/foo_bar.dart';

export 'src/error.dart';
```

#### DO 각 구역은 알파벳순으로 정렬하세요.

```dart
import 'package:bar/bar.dart';
import 'package:foo/foo.dart';

import 'foo.dart';
import 'foo/foo.dart';
```

### 포맷

#### DO `dart format` 명령어를 통해 코드를 포맷 하세요.

#### CONSIDER 짜여진 코드가 formatter에 친화적인 코드가 되도록 변경하세요.

포맷을 변경하는 것은 코드를 다시 구성하거나 코드를 간단하게 만드는 것이므로 `dart format`을 파트너로 생각하고, 직접 다시 수정했다면 어떻게 했을지 고민해보는 것이 좋습니다.

#### AVOID 코드 라인이 80자 이상을 넘지 않도록 하세요(URI와 file path 그리고 긴 string 값 제외).

#### DO 모든 흐름 제어문에 중괄호를 사용하세요. 이를 통해 흐름제어문의 모호성으로 인해 생기는 문제를 해결합니다.

```dart
if (isWeekDay) {
  print('Bike to work!');
} else {
  print('Go dancing or read a book!');
}
```

## 설명 문서 작성

여기서는 개발된 코드에 대해 효과적으로 의도를 전달하는 방법에 대해 체크해봅니다.

### 코멘트

#### DO 문장과 같은 형태로 코멘트의 포맷을 구성하세요.

첫째 단어의 경우 대소문자가 중요한 경우를 제외하고는 대문자로 시작합니다.

```dart
// Not if anything comes before it.
if (_chunks.isNotEmpty) return false;
```

#### DON'T 블록 코멘트를 작성하지 마세요.

```dart
void greet(String name) {
  // Assume we have a valid name. 🙆
  /* Assume we have a valid name. */ 🙅
  print('Hi, $name!');
}
```

### Doc 코멘트

Doc 코멘트는 `dart doc` 명령어를 통해 쉽게 만들 수 있는 doc 페이지를 위한 코멘트를 말합니다. 이 코멘트들은 `///`을 가져야 합니다.

#### DO 문서에 포함시킬 것은 `///`를 사용해 코멘트를 남기세요. 참고로 `/** ... */`도 지원합니다.

```dart
/// The number of characters in this chunk when unsplit. 🙆
// The number of characters in this chunk when unsplit. 🙅
int get length => ...
```

#### PREFER Public API에 대해 doc 코멘트를 남기세요.

모든 개별 라이브러리, 변수 등에 코멘트를 남길 필요는 없지만 가능하면 작성해주는 것이 좋습니다.

#### CONSIDER 라이브러리 단위에서 doc 코멘트를 남기는 것을 고려해보세요.

다음과 같은 것들을 포함시킬 수 있습니다.

- 라이브러리가 무엇에 관한 것인지에 대한 한줄 요약
- 라이브러리에 사용되는 전문용어에 대한 설명
- API를 사용하는데 참고할만한 샘플 코드
- 가장 자주 사용되거나 가장 중요하다고 생각되는 클래스 그리고 함수에 대한 참고 링크
- 해당 라이브러리에 관계된 주요 도메인에 대해 참조할 수 있는 외부 링크

```dart
/// A really great test library.
@TestOn('browser')
library;
```

#### CONSIDER Private API에 대해 코멘트를 남기는 것을 고려해보세요.

Doc 코멘트는 외부 소비자들만을 위해 남기지는 않습니다. 라이브러리 내부에서 사용되는 부분들을 이해하는데 사용할 수도 있습니다.

#### DO Doc 코멘트는 한줄 요약으로 시작하세요.

```dart
/// Deletes the file at [path] from the file system.
void delete(String path) {
  ...
}
```

#### DO Doc 코멘트에서 첫째 문장은 개별 문단으로 구분하세요.

```dart
/// Deletes the file at [path].
///
/// Throws an [IOError] if the file could not be found. Throws a
/// [PermissionError] if the file is present but could not be deleted.
void delete(String path) {
  ...
}
```

#### AVOID 문맥 상 불필요한 내용은 삭제하세요.

독자가 모르는 내용을 우선하여 작성합니다.

```dart
class RadioButtonWidget extends Widget {
  /// Sets the tooltip to [lines], which should have been word wrapped using
  /// the current font. 🙆
  void tooltip(List<String> lines) {
    ...
  }
}

class RadioButtonWidget extends Widget {
  /// Sets the tooltip for this radio button widget to the list of strings in
  /// [lines]. 🙅
  void tooltip(List<String> lines) {
    ...
  }
}
```

#### PREFER 함수 혹은 method 코멘트는 제3자 동사로 시작하세요.

Doc 코멘트는 코드가 하는 것에 집중합니다.

```dart
/// Returns `true` if every element satisfies the [predicate].
bool all(bool predicate(T element)) => ...

/// Starts the stopwatch if not already running.
void start() {
  ...
}
```

#### PREFER Boolean이 아닌 변수 혹은 프로퍼티와 관련된 코멘트는 명사구로 시작하세요.

해당 변수 혹은 프로퍼티가 무엇을 말하는지 강조해서 서술해야 합니다.

```dart
/// The current day of the week, where `0` is Sunday.
int weekday;

/// The number of checked buttons on the page.
int get checkedCount => ...
```

#### PREFER Boolean 변수 혹은 프로퍼티 코멘트는 "Whether(-이거나)"와 어울리는 명사 혹은 동명사를 사용해 서술하세요.

```dart
/// Whether the modal is currently displayed to the user.
bool isVisible;

/// Whether the modal should confirm the user's intent on navigation.
bool get shouldConfirm => ...

/// Whether resizing the current browser window will also resize the modal.
bool get canResize => ...
```

#### DON'T 한 프로퍼티의 getter와 setter 각각에 주석을 작성하지 마세요.

`dart doc`은 getter와 setter을 하나의 필드로서 다루므로 한개의 코멘트만 남기도록 합니다.

```dart
// Good 🙆
/// The pH level of the water in the pool.
///
/// Ranges from 0-14, representing acidic to basic, with 7 being neutral.
int get phLevel => ...
set phLevel(int level) => ...

// Bad 🙅
/// The depth of the water in the pool, in meters.
int get waterDepth => ...

/// Updates the water depth to a total of [meters] in height.
set waterDepth(int meters) => ...
```

#### PREFER 라이브러리 혹은 타입 코멘트는 명사구로 시작하세요.
```dart
/// A chunk of non-breaking output text terminated by a hard or soft newline.
///
/// ...
class Chunk { ... }
```

#### CONSIDER Doc 코멘트에 코드 샘플을 포함하세요.

```dart
/// Returns the lesser of two numbers.
///
/// ```dart
/// min(5, 3) == 3
/// ```
num min(num a, num b) => ...
```

#### DO 스코프 안에 있는 식별자를 참고할 때는 doc 코멘트에 대괄호를 사용하세요.

method 혹은 생성자를 참고할 때는 `()`를 사용해 좀 더 깔끔하게 문서를 작성할 수 있습니다.

```dart
/// Throws a [StateError] if ...
/// similar to [anotherMethod()], but ...

/// Similar to [Duration.inDays], but handles fractional days.

// 이름이 없는 생성자는 .new를 사용합니다.
/// To create a point, call [Point.new] or use [Point.polar] to ...
```

#### DO Parameters, return values와 exceptions를 설명할 때는 산문체를 사용합니다.

```dart
/// Defines a flag.
///
/// Throws an [ArgumentError] if there is already an option named [name] or
/// there is already an option using abbreviation [abbr]. Returns the new flag.
Flag addFlag(String name, String abbr) => ...
```

#### DO Metadata annotation 전에 doc 코멘트를 넣으세요.

```dart
// Good 🙆
/// A button that can be flipped on and off.
@Component(selector: 'toggle')
class ToggleComponent {}

// Bad 🙅
@Component(selector: 'toggle')
/// A button that can be flipped on and off.
class ToggleComponent {}
```

### 마크다운

`dart doc` 명령어는 대부분의 마크다운 포맷을 허용하고 있습니다. 아래는 사용예시입니다.

```dart
/// This is a paragraph of regular text.
///
/// This sentence has *two* _emphasized_ words (italics) and **two**
/// __strong__ ones (bold).
///
/// A blank line creates a separate paragraph. It has some `inline code`
/// delimited using backticks.
///
/// * Unordered lists.
/// * Look like ASCII bullet lists.
/// * You can also use `-` or `+`.
///
/// 1. Numbered lists.
/// 2. Are, well, numbered.
/// 1. But the values don't matter.
///
///     * You can nest lists too.
///     * They must be indented at least 4 spaces.
///     * (Well, 5 including the space after `///`.)
///
/// Code blocks are fenced in triple backticks:
///
/// ```dart
/// this.code
///     .will
///     .retain(its, formatting);
/// ```
///
/// The code language (for syntax highlighting) defaults to Dart. You can
/// specify it by putting the name of the language after the opening backticks:
///
/// ```html
/// <h1>HTML is magical!</h1>
/// ```
///
/// Links can be:
///
/// * https://www.just-a-bare-url.com
/// * [with the URL inline](https://google.com)
/// * [or separated out][ref link]
///
/// [ref link]: https://google.com
///
/// # A Header
///
/// ## A subheader
///
/// ### A subsubheader
///
/// #### If you need this many levels of headers, you're doing it wrong
```

#### AVOID 마크다운을 과하게 사용하지 마세요.

사용해도 될지 의심이 된다면 가능한 적게 사용하는 것이 좋습니다.

#### AVOID HTML을 사용해 포맷을 만들지 마세요.

표를 만들거나 하는 등의 특별한 경우 도움이 될지도 모르겠으나 그렇게 복잡하게 표현해야 하는 마크다운은 사용하지 않는 편이 나으니까요.

#### PREFER Backtick(`\``)을 사용해 코드블록을 만드세요.

```dart
/// You can use [CodeBlockExample] like this:
///
/// ```dart
/// var example = CodeBlockExample();
/// print(example.isItGreat); // "Yes."
/// ```
```

### 쓰기

_[기술적 쓰기 방법](https://en.wikiversity.org/wiki/Technical_writing/Style)_

#### PREFER 간결성을 유지하세요.

명백하고 정확하되 간결하게 작성하는 것이 중요합니다.

#### AVOID 정확한 의미를 가진 축약어와 두음문자가 아니라면 사용을 지양하세요.

"i.e.", "e.g." 그리고 "et al"과 같은 축약어도 어떤 사람들은 이해하지 못할지도 모르니 널리 알려진 것이 아니라면 사용을 하지 않는 것이 좋습니다.

#### PREFER 멤버의 객체를 참조할 때는 "the"보다는 "this"를 사용하세요.

```dart
class Box {
  /// The value this wraps.
  Object? _value;

  /// True if this box contains a value.
  bool get hasValue => _value != null;
}
```

## 사용

### 라이브러리

#### DO `part of` 지시자에 string 값을 사용하세요.

라이브러리의 한 부분을 떼어내기 위해 `part`를 사용한다면, 라이브러리의 떼어진 부분을 가리키는 다른 파일을 필요로 합니다. 이 때, string 값은 URI string을 사용하는 것을 보통 선호합니다.

```dart
// my_library.dart
library my_library;

part 'some/other/file.dart';
```

`my_library`의 part file은 URI string을 사용하는 것이 좋습니다.

```dart
// Good 🙆
part of '../../my_library.dart';

// Bad 🙅
part of my_library;
```

#### DON'T 다른 패키지의 `src` 디렉토리 안에 있는 라이브러리를 import 하지 마세요.

`src` 이하 코드는 광범위한 변경사항이 자유롭게 발생하는 공간이므로 주 패키지 자체는 변경사항이 적을지라도 패키지 안의 라이브러리들은 광범위한 변경사항이 발생해 기존에 작동하던 코드를 망가뜨릴 수 있습니다.

#### DON'T import path가 `lib`의 내부 혹은 외부 요소에 닿도록 허용하지 마세요.

```dart
// Bad 🙅
// 같은 경로지만 서로 다른 것으로 인식함.
import 'package:my_package/api.dart';
import '../lib/api.dart';
```

위와 같은 경우를 피하기 위해 다음의 방법을 제시합니다.

- import path에 `/lib/`을 사용하지 마세요.
- `lib` 디렉토리를 벗어나는 `../`과 같은 표현을 사용하지 마세요.

package의 lib 디렉토리에 접근해야 하는 경우 다음과 같이 하세요.

```dart
// Good 🙆
import 'package:my_package/api.dart';
```

#### PREFER 직전의 룰이 작동하지 않는다면 relative import path를 사용하세요.

import 할 때 `lib`을 통해 타겟 위치에 도달할 수 없다면 relative import를 사용하는 것이 좋습니다. 예를 들어 다음의 디렉토리 구조가 있다고 가정하겠습니다.

```
my_package
└─ lib
   ├─ src
   │  └─ stuff.dart
   │  └─ utils.dart
   └─ api.dart
   test
   │─ api_test.dart
   └─ test_utils.dart
```

```dart
// Good 🙆
// 위치: lib/api.dart
import 'src/stuff.dart';
import 'src/utils.dart';

// Good 🙆
// 위치: lib/src/utils.dart
import '../api.dart';
import 'stuff.dart';

// Good 🙆
import 'package:my_package/api.dart'; // Don't reach into 'lib'.
import 'test_utils.dart'; // Relative within 'test' is fine.
```

### NULL

#### DON'T 변수에 명시적으로 `null`로 초기화 하지 마세요.

만약 변수가 `null`이 될 수 없는 타입을 가지고 있다면, 다트는 이 변수가 초기화 되기 전에 컴파일 에러를 발생시킵니다. 만약 변수가 `null`이 될 수 있는 타입이라면, 이 변수가 `null`로 초기화 될 수 있음을 암시하게 됩니다. 다트에는 "초기화되지 않은 메모리"라는 개념이 없기 때문에 명시적으로 변수에 "안전하게" 초기화시키기 위해 `null`을 값으로 줄 필요는 없습니다.

```dart
// Good 🙆
Item? bestDeal(List<Item> cart) {
  Item? bestItem;

  for (final item in cart) {
    if (bestItem == null || item.price < bestItem.price) {
      bestItem = item;
    }
  }

  return bestItem;
}

// Bad 🙅
Item? bestDeal(List<Item> cart) {
  Item? bestItem = null; // 명시적 null 초기화

  for (final item in cart) {
    if (bestItem == null || item.price < bestItem.price) {
      bestItem = item;
    }
  }

  return bestItem;
}
```

#### DON'T `null`을 기본값으로 사용하지 마세요.

`null` 값이 될 수도 있는 파라미터는 `null`을 기본값으로 지정하기 때문에 명시적으로 값을 할당할 필요는 없습니다.

```dart
// Good 🙆
void error([String? message]) {
  stderr.write(message ?? '\n');
}

// Bad 🙅
void error([String? message = null]) {
  stderr.write(message ?? '\n');
}
```

#### DON'T 동일 여부를 판별하는 연산자에 `true` 또는 `false`를 사용하지 마세요.

```dart
// Good 🙆
if (nonNullableBool) { ... }

if (!nonNullableBool) { ... }

// Bad 🙅
if (nonNullableBool == true) { ... }

if (nonNullableBool == false) { ... }
```

_nullable_ 에 대해 boolean 표현 방식으로 평가하고 싶다면 `??` 혹은 `!=` `null` 체크를 사용해야 합니다.

```dart
// Good 🙆
// If you want null to result in false:
if (nullableBool ?? false) { ... }

// If you want null to result in false
// and you want the variable to type promote:
if (nullableBool != null && nullableBool) { ... }

// Bad 🙅
// Static error if null:
if (nullableBool) { ... }

// If you want null to be false:
if (nullableBool == true) { ... }
```

`nullableBool == true`은 실행 가능한 표현이긴 하지만 사용하지 말아야 할 이유가 몇 가지 있습니다.

- 코드가 `null`과 어떤 관계를 가지고 있음을 나타내지 않습니다.
- `null`과 관계되었다는 증거가 보이지 않기 때문에 쉽게 실수가 생길 수 있습니다.
- boolean 로직은 개발자로 하여금 혼란을 유발합니다. `nullableBool`이 `null`이라면 `nullableBool == true`는 `false`로 평가됩니다.

#### AVOID 변수가 초기화 됐는지 여부를 확인하려면 `late` 선언을 피하세요.

다트는 `late` 변수가 초기화 됐는지 혹은 할당 됐는지 알려줄 수 있는 방법이 없기 때문에 initializer가 있다면 실행하거나 오류를 발생시킵니다.

#### CONSIDER `nullable` 타입을 사용하는 경우에는 타입 승격(Type promotion) 혹은 null-check 패턴을 고려하세요.

`nullable` 변수가 `null`과 같지 않다면 변수는 `null`이 아닌 타입으로 승격합니다. 하지만 타입 승격은 지역 변수, 파라미터, priviate final 필드에서만 지원됩니다. 변경 될 수 있도록 열려 있는 값은 타입 승격을 할 수 없습니다.

구성요소(Member)를 `private` 그리고 `final`로 선언하는 것은 이러한 제약을 우회하게끔 하지만 항상 이 방법을 선택할 필요는 없습니다.

타입 승격의 제약을 해결하기 위한 한 방법으로 null-check 패턴을 권합니다. 이 패턴은 구성요소의 값이 null이 아님을 확정하는 동시에 null 타입만 제외된 새로운 변수에 바인드 됩니다.

```dart
class UploadException {
  final Response? response;

  UploadException([this.response]);

  @override
  String toString() {
    if (this.response case var response?) {
      return 'Could not complete upload to ${response.url} '
          '(error code ${response.errorCode}): ${response.reason}.';
    }
    return 'Could not upload (no response).';
  }
}
```

또 다른 방법은 필드값을 지역 변수에 할당하는 겁니다. 이 지역 변수에 null-check를 하게 되면 타입 승격이 발생하고 안전하게 null이 아닌 값으로 다룰 수 있게 됩니다.

```dart
class UploadException {
  final Response? response;

  UploadException([this.response]);

  @override
  String toString() {
    final response = this.response; // 지역 변수에 할당
    if (response != null) {
      return 'Could not complete upload to ${response.url} '
          '(error code ${response.errorCode}): ${response.reason}.';
    }
    return 'Could not upload (no response).';
  }
}
```

지역 변수(method 내부에서 선언된 변수)를 사용할 때는 조심하세요. 만약 필드(클래스에 속한 변수)에 다시 값을 할당해야 한다면, 지역 변수에 할당하는 실수를 저지르지 않도록 하세요(지역 변수가 `final`이라면 이러한 실수를 예방할 수 있습니다). 그리고 만약 지역 변수가 스코프 안에 있을 때 필드를 변화시킨다면, 지역 변수는 오래된 값이 된다는 것에 주의하세요.

때때로 단순히 필드에 `!`를 사용하는 것이 제일 좋은 방법일 수도 있습니다. 하지만 몇몇 경우에서는 지역 변수를 사용하거나 null-check 패턴을 사용하는 것이 좀 더 깔끔하고 안전할 수 있다는 점을 기억하세요.

```dart
// Bad 🙅
class UploadException {
  final Response? response;

  UploadException([this.response]);

  @override
  String toString() {
    if (response != null) {
      return 'Could not complete upload to ${response!.url} '
          '(error code ${response!.errorCode}): ${response!.reason}.';
    }

    return 'Could not upload (no response).';
  }
}
```

### Strings

#### DO 인접한 별개의 string을 연속으로 사용하세요.

```dart
// Good 🙆
raiseAlarm('ERROR: Parts of the spaceship are on fire. Other '
    'parts are overrun by martians. Unclear which are which.');

// Error 🙅
// "+"를 사용할 필요는 없습니다.
raiseAlarm('ERROR: Parts of the spaceship are on fire. Other ' +
    'parts are overrun by martians. Unclear which are which.');
```

#### PREFER `+` 연산자를 사용하지 말고, string 내부에 특정한 패턴으로 값을 넣으세요.

```dart
// Good 🙆
'Hello, $name! You are ${year - birth} years old.';

// Bad 🙅
'Hello, ' + name + '! You are ' + (year - birth).toString() + ' y...';
```

#### AVOID string에 값을 넣을 때, 불필요한 중괄호 사용을 피하세요.

```dart
// Good 🙆
var greeting = 'Hi, $name! I love your ${decade}s costume.';

// Bad 🙅
var greeting = 'Hi, ${name}! I love your ${decade}s costume.';
```

### Collections

다트는 list, map, queue, set 4가지 컬렉션을 지원합니다.

#### DO 가능한 한 컬렉션 리터럴을 사용하세요.

다트는 세가지 핵심적인 컬렉션 타입을 가지고 있습니다(List, Map 그리고 Set). Map과 Set 클래스는 대부분의 클래스가 그러한 것처럼 이름 없는 생성자를 가지고 있습니다. 하지만 이러한 컬렉션들은 빈번하게 사용되기 때문에, 다트에서는 이러한 컬렉션들을 생산할 수 있는 좀 더 훌륭한 빌트인 문법을 제공합니다.

```dart
// Good 🙆
var points = <Point>[];
var addresses = <String, Address>{};
var counts = <int>{};

// Bad 🙅
var addresses = Map<String, Address>();
var counts = Set<int>();
```

이 가이드라인은 컬렉션 클래스에 명명된 생성자가 존재할 경우에는 적용되지 않습니다(`List.from()`, `Map.fromIterable()` 등).

특히 컬렉션 리터럴은 spread 연산자와 `if` 그리고 `for`을 제공합니다.

```dart
// Good 🙆
var arguments = [
  ...options,
  command,
  ...?modeFlags,
  for (var path in filePaths)
    if (path.endsWith('.dart')) path.replaceAll('.dart', '.js')
];

// Bad 🙅
var arguments = <String>[];
arguments.addAll(options);
arguments.add(command);
if (modeFlags != null) arguments.addAll(modeFlags);
arguments.addAll(filePaths
    .where((path) => path.endsWith('.dart'))
    .map((path) => path.replaceAll('.dart', '.js')));
```

#### DON'T 컬렉션이 빈 상태인지 확인하기 위해 `.length`를 사용하지 마세요.

`.length`를 호출해 컬렉션이 비어 있는 상태인지 확인한다면, 끔찍스러울 정도로 느릴 수 있습니다.

대신에, 좀 더 빠르고 읽기 쉬운 형태의 getter를 사용해볼 수 있습니다. `.isEmpty`와 `.isNotEmpty`를 사용해보세요.

```dart
// Good 🙆
if (lunchBox.isEmpty) return 'so hungry...';
if (words.isNotEmpty) return words.join(' ');

// Bad 🙅
if (lunchBox.length == 0) return 'so hungry...';
if (!words.isEmpty) return words.join(' ');
```

#### AVOID `Iterable.forEach()`를 함수 리터럴과 사용하는 것을 피하세요.

`for-in` loop에서 개발자가 대개 기대하고 있는 것을 얻을 수 없기 때문에, 자바스크립트 진영에서는 `forEach()` 함수를 폭넓게 사용하고 있습니다. 그러나 다트에서는 `for-in` loop을 통해 구현할 수 있습니다.

```dart
// Good 🙆
for (final person in people) {
  ...
}

// Bad 🙅
people.forEach((person) {
  ...
});
```

> 이 가이드라인에서는 "함수 _리터럴_"에 한해서라고 명시합니다. 만약 _이미 존재하는_ 함수를 `forEach()`에 적용시키는 것이라면 괜찮습니다.

```dart
// Good 🙆
people.forEach(print);
```

> `Map.forEach()`를 사용하는 것은 언제든지 가능합니다. Map은 반복문을 실행할 수 없기 때문에 이 가이드라인의 내용이 적용되지 않습니다.

#### DON'T 얻은 결과의 타입을 의도적으로 변경하려 하지 않는 이상 `List.from()`을 사용하지 마세요.

주어진 반복문으로 같은 요소를 포함하고 있는 새로운 List를 생산하는 것에는 두 가지 명확한 방법이 존재합니다.

```dart
var copy1 = iterable.toList();
var copy2 = List.from(iterable);
```

명확하게 보이는 차이점은 첫번째가 더 짧다는 겁니다. 그리고 그보다 _중요한_ 차이는 첫번째의 경우 original object의 type 인자(argument)를 보존한다는 점입니다.

```dart
// Good🙆
// Creates a List<int>:
var iterable = [1, 2, 3];

// Prints "List<int>":
print(iterable.toList().runtimeType);

// Bad 🙅
// Creates a List<int>:
var iterable = [1, 2, 3];

// Prints "List<dynamic>":
print(List.from(iterable).runtimeType);
```

만약 타입을 변경하기를 원한다면, `List.from()`이 유용할 수 있습니다.

```dart
// Good 🙆
var numbers = [1, 2.3, 4]; // List<num>.
numbers.removeAt(1); // Now it only contains integers.
var ints = List<int>.from(numbers);
```

따라서 반복문을 단순히 복사하고 원 타입을 보존할 생각이거나 또는 타입을 상관하지 않을 생각이라면 `toList()`를 사용합니다.

#### DO 컬렉션을 타입 별로 필터링하기 위해서 `whereType()`을 사용하세요.

예를 들어, object에 복합 요소가 섞여 있는 배열이 있다고 가정해봅시다. 그리고 여기서 integer 값들만 빼내고 싶다면 `where()`를 사용해 다음과 같이 작성할 수 있을 겁니다.

```dart
// Bad 🙅
var objects = [1, 'a', 2, 'b', 3];
var ints = objects.where((e) => e is int);
```

위 코드는 부산스럽기도 할 뿐 아니라 더 안 좋은 점은 원치 않는 타입을 가진 순환 리스트를 반환한다는 점입니다. `Iterable<int>` 타입을 기대하고 필터링을 했겠지만 반환되는 결과값의 반환값은 `Iterable<Object>` 입니다.

가끔 `cast()`를 사용해 위 에러를 "올바르게" 고치려는 코드를 볼 수 있습니다.

```dart
// Bad 🙅
var objects = [1, 'a', 2, 'b', 3];
var ints = objects.where((e) => e is int).cast<int>();
```

위 방법 역시 부산스러워 보이는군요. 간접적인 두개 층의 wrapper를 생성할 뿐 아니라 불필요한 런타임 확인을 거치니 말입니다. 운이 좋게도, 코어 라이브러리에서는 `whereType()` method를 지원하고 있어 이러한 상황에서 사용할 수 있습니다.

```dart
// Good 🙆
var objects = [1, 'a', 2, 'b', 3];
var ints = objects.whereType<int>();
```

#### DON'T 앞선 구간에서 `cast()`와 같은 타입 전환을 할 수 있을 때는 굳이 `cast()`를 사용하지 마세요.

`Iterable` 또는 `Stream`을 다룰 때, 몇 번의 변화를 걸쳐 종래에는 특정 타입 인자를 생산하려고 할 때가 종종 있을 겁니다. 이 때, `cast()`를 사용하기 보다는 앞선 구간에서 타입 전환을 할 수 있는지 확인해보세요.

예를 들어 `toList()`를 사용했었다면 `List<T>.from()`으로 변경해 `T`에 원하는 타입을 집어넣도록 하세요.

```dart
// Good 🙆
var stuff = <dynamic>[1, 2];
var ints = List<int>.from(stuff);

// Bad 🙅
var stuff = <dynamic>[1, 2];
var ints = stuff.toList().cast<int>();
```

`map()`을 사용할 때도 명시적인 타입 인자를 넘기게 되면 원하는 형태의 타입을 가진 iterable을 만들 수 있습니다. 타입추론은 `map()`에 넘긴 함수에 기반해 적절한 타입을 선택하지만 때로는 명시적으로 주는 것이 좋습니다.

```dart
// Good 🙆
var stuff = <dynamic>[1, 2];
var reciprocals = stuff.map<double>((n) => 1 / n);

// Bad 🙅
var stuff = <dynamic>[1, 2];
var reciprocals = stuff.map((n) => 1 / n).cast<double>();
```

#### AVOID `cast()` 사용을 지양하세요. 대신 다음의 옵션을 고려해보는 것이 좋습니다.

- **올바른 타입을 생성하세요.** 컬렉션이 처음 생성된 곳을 변경해 제대로 된 타입을 가지게 하세요.

```dart
// Good 🙆
List<int> singletonList(int value) {
  var list = <int>[];
  list.add(value);
  return list;
}

// Bad 🙅
List<int> singletonList(int value) {
  var list = []; // List<dynamic>.
  list.add(value);
  return list.cast<int>();
}
```

- **접근 중인 요소를 변환(cast) 하세요.** 컬렉션을 반복(iterate)하고 있다면, 각각의 요소를 반복하는 동안 변환하세요.

```dart
// Good 🙆
void printEvens(List<Object> objects) {
  // We happen to know the list only contains ints.
  for (final n in objects) {
    if ((n as int).isEven) print(n);
  }
}

// Bad 🙅
void printEvens(List<Object> objects) {
  // We happen to know the list only contains ints.
  for (final n in objects.cast<int>()) {
    if (n.isEven) print(n);
  }
}
```

- **적극적으로 `List.from()`을 사용해서 변환하세요.** `cast()` method는 _모든 작동 구간_ 마다 요소의 타입을 확인하는 느린 컬렉션(lazy collection)을 반환합니다. 소수의 요소에 대해 소수의 작동만이 일어난다면 이러한 느린 특성(laziness)은 좋은 선택일 수 있습니다. 그러나 많은 경우, 느린 검증 과정과 이득을 넘어서는 기타 비용(overhead) 등이 생기기 때문에 추천하지 않습니다.

```dart
// Good 🙆
int median(List<Object> objects) {
  // We happen to know the list only contains ints.
  var ints = List<int>.from(objects);
  ints.sort();
  return ints[ints.length ~/ 2];
}

// Bad 🙅
int median(List<Object> objects) {
  // We happen to know the list only contains ints.
  var ints = objects.cast<int>();
  ints.sort();
  return ints[ints.length ~/ 2];
}
```

### Functions

다트에서는 함수도 오브젝트입니다.

#### DO 함수를 이름에 바인드 할 때는 함수 선언을 사용하세요.

```dart
// Good 🙆
void main() {
  void localFunction() {
    ...
  }
}

// Bad 🙅
void main() {
  var localFunction = () {
    ...
  };
}
```

#### DON'T tear-off가 작동할 때는 람다를 생성하지 마세요.

괄호 없이 함수, method 혹은 이름이 있는 생성자를 참조하면 다트는 _tear off_ 라는 것을 생성합니다. 이 클로저는 함수로서 같은 파라미터를 가지고 있고, 호출했을 때 기저의 함수를 작동시킵니다. 만약 클로저가 받아들이는 것과 같은 파라미터를 가지고 있으면서 이름이 있는 함수를 작동시키려면 람다로 감싸지 말고, tear-off를 사용합니다.

```dart
// Good 🙆
var charCodes = [68, 97, 114, 116];
var buffer = StringBuffer();

// Function:
charCodes.forEach(print);

// Method:
charCodes.forEach(buffer.write);

// Named constructor:
var strings = charCodes.map(String.fromCharCode);

// Unnamed constructor:
var buffers = charCodes.map(StringBuffer.new);

// Bad 🙅
var charCodes = [68, 97, 114, 116];
var buffer = StringBuffer();

// Function:
charCodes.forEach((code) {
  print(code);
});

// Method:
charCodes.forEach((code) {
  buffer.write(code);
});

// Named constructor:
var strings = charCodes.map((code) => String.fromCharCode(code));

// Unnamed constructor:
var buffers = charCodes.map((code) => StringBuffer(code));
```

### Variables

#### DO 지역 변수로서 `var`과 `final`에 대한 일관성 있는 규칙을 따르세요.

대부분의 지역 변수는 타입 어노테이션을 가지고 있어서는 안 되며, `var` 혹은 `final`을 사용해 선언 되어야 합니다. 어떤 케이스에서 어떤 것을 사용할지에 대해 널리 쓰이는 두 가지 규칙이 있습니다.

- 재할당 되지 않는 지역 변수는 `final`을 사용합니다. 재할당 되는 변수에는 `var`을 사용합니다.
- 모든 지역 변수에 `var`을 사용합니다. 심지어 재할당 되지 않더라도 `var`을 사용합니다. 지역 변수에 대해 `final`을 절대 사용하지 않습니다.(필드와 top-level 변수에 대해 `final`을 사용합니다.)

두 가지 규칙 모두 받아들일만 합니다. 그러나 하나를 선택했다면 일관성 있게 그 규칙을 유지하도록 합니다. 그렇게 했을 때 독자는 `var`을 봤을 때, 함수 내에서 나중에 변수에 값이 할당되는지 여부를 알 수 있게 됩니다.

#### AVOID 계산할 수 있는 것은 저장하는 것을 피하세요.

종종 클래스를 디자인 할 때 같은 기저의 상태에서 여러개의 뷰를 노출시키기를 원하는 상황이 있을 겁니다. 이 때, 관계된 뷰를 생산자에서 모두 계산해 저장하는 코드를 본 적이 종종 있을 겁니다.

```dart
// Bad 🙅
class Circle {
  double radius;
  double area;
  double circumference;

  Circle(double radius)
      : radius = radius,
        area = pi * radius * radius,
        circumference = pi * 2.0 * radius;
}
```

이 코드는 두 가지 측면에서 잘못됐습니다. 첫째, 이 코드는 메모리를 낭비하기 쉽습니다. 엄밀히 말하면, `area`와 `circumference`는 _캐쉬_ 입니다. 이것들은 저장된 계산으로 다른 데이터를 재계산할 때도 이미 가지고 있는 것으로 계산 할 수 있습니다. 이는 CPU 사용량을 감소시키는 대신 메모리를 증가시킨다는 점에서 일종의 거래를 하는 겁니다. 그런데 이렇게 trade-off를 해서 이득이 있을 정도로 퍼포먼스에 문제가 있다는 것을 우리가 알고 있는 상태인가요?

더 나쁜 점은 코드가 _잘못됐다_ 는 겁니다. 캐쉬가 오래된 상태인 시점과 재계산을 해야 하는 시점에 대한 캐쉬의 무효화 문제가 있습니다. 여기서는 `radius`가 변할 수 있음에도 무효화 하지 않습니다. 따라서 다른 값을 할당하더라도 `area`와 `circumference`는 이전의 값을 유지하고 있을 것이므로 정확하지 않은 값을 반환할 겁니다.

캐시 무효화를 정확하게 다루기 위해 우리는 다음이 필요합니다.

```dart
// Bad 🙅
class Circle {
  double _radius;
  double get radius => _radius;
  set radius(double value) {
    _radius = value;
    _recalculate();
  }

  double _area = 0.0;
  double get area => _area;

  double _circumference = 0.0;
  double get circumference => _circumference;

  Circle(this._radius) {
    _recalculate();
  }

  void _recalculate() {
    _area = pi * _radius * _radius;
    _circumference = pi * 2.0 * _radius;
  }
}
```

작성하고, 유지하고, 디버그하고 읽기에는 너무 많은 코드들이 짜여 있는 상태입니다. 다음과 같이 실행해 봅시다.

```dart
// Good 🙆
class Circle {
  double radius;

  Circle(this.radius);

  double get area => pi * radius * radius;
  double get circumference => pi * 2.0 * radius;
}
```

이 코드가 좀 더 짧고 메모리를 덜 사용하고 에러를 덜 발생시킵니다. 이 코드는 circle을 표현하는데 있어 최소한의 데이터만 저장합니다. 여기서는 진실한 단 하나의 공급자만 있기 때문에 동기화 될 필드는 존재하지 않습니다.

몇몇 경우에서 느린 계산 결과에 대해서는 캐시를 남기는 것이 필요할지도 모릅니다. 그러나 이는 퍼포먼스 문제가 있다는 것을 명확히 인지하고 조심스럽게 실행해야 합니다. 물론, 이러한 최적화 코드에는 설명을 남겨야 합니다.

### Members

다트에는 함수 (method) 혹은 데이터 (객체 변수)가 될 수 있는 멤버라고 불리는 오브젝트를 가지고 있습니다.

#### DON'T Setter와 Getter에 불필요한 필드를 감싸지 마세요.

Java와 C#에서는 getter와 setter가 단순히 해당 필드로 포워딩 하는 역할만 할 뿐일지라도, 모든 필드를 getter와 setter에 숨기는 것이 흔합니다. 그래서 해당 멤버들에 접근해 작업한다면 호출 지점을 건드리지 않고도 많은 것을 할 수 있습니다. 그 이유 중 하나는 자바에서 필드에 접근하는 방식이었던 getter method를 호출하는 것은 다트에서는 다른 형태이기 때문입니다. 그리고 C#에서 raw 필드에 접근할 때와 달리 프로퍼티에 접근하는 것은 바이너리 호환적이지 않기 때문입니다.

다트에는 이러한 한계가 없습니다. 필드와 getter/setter는 완전히 구분히 안 됩니다. 클래스에서 필드를 노출시킬 수 있을 뿐 아니라 해당 필드를 사용하는 코드에 어떤 영향도 주지 않고 getter와 setter로 감쌀 수도 있습니다.

```dart
// Good 🙆
class Box {
  Object? contents;
}

// Bad 🙅
class Box {
  Object? _contents;
  Object? get contents => _contents;
  set contents(Object? value) {
    _contents = value;
  }
}
```

#### PREFER read-only 프로퍼티를 만들기 위해서는 `final` 필드를 사용하세요.

외부 코드가 볼 수 있으나 값은 할당할 수 없게 하려는 필드가 있다면 `final`로 표시합니다.

```dart
// Good 🙆
class Box {
  final contents = [];
}

// Bad 🙅
class Box {
  Object? _contents;
  Object? get contents => _contents;
}
```

#### CONSIDER 간단한 멤버에 대해서는 `=>` 사용을 고려해보세요.

다트는 함수 표현에 대해 `=>`을 사용하는 것에 더하여 이를 통해 멤버를 정의할 수 있게 합니다. 이 스타일은 계산과 값 반환 따위의 간단한 것에 적합합니다.

```dart
double get area => (right - left) * (bottom - top);

String capitalize(String name) =>
    '${name[0].toUpperCase()}${name.substring(1)}';
```

사람들은 `=>`를 사용해 코드를 _작성하는_ 것을 좋아하는 것처럼 보입니다만, 이 표현은 남용하기 매우 쉽고 이로 인해 코드가 _읽기_ 어려울 수 있습니다. 만약 선언부가 여러 라인으로 구성되거나 깊은 내재 표현(nested expression)을 포함하고 있다면 (가령 연속적인 표현 혹은 조건 연산자) 스스로나 당신의 코드를 읽는 모든 사람을 위해 블록 body와 어느 정도의 설명문을 곁들이는 것이 좋습니다.

```dart
// Good 🙆
Treasure? openChest(Chest chest, Point where) {
  if (_opened.containsKey(chest)) return null;

  var treasure = Treasure(where);
  treasure.addAll(chest.contents);
  _opened[chest] = treasure;
  return treasure;
}

// Bad 🙅
Treasure? openChest(Chest chest, Point where) => _opened.containsKey(chest)
    ? null
    : _opened[chest] = (Treasure(where)..addAll(chest.contents));
```

값을 반환하지 않는 멤버에 대해서도 `=>`를 사용할 수 있습니다. 이 관용구는 setter의 수가 적고 `=>`를 사용하는 상응하는 getter가 있을 때 사용할 수 있습니다.

```dart
// Good 🙆
num get x => center.x;
set x(num value) => center = Point(value, center.y);
```

#### DON'T 이름이 있는 생성자로 방향을 전환하거나 섀도잉(shadowing)을 피하는 것을 제외하고는 `this.`를 사용하지 마세요.

자바스크립트는 최근 오브젝트에서 실행되는 method 멤버를 참고하기 위해 명시적인 `this.`를 요구합니다. 하지만 다트는 이런 한계점은 없습니다.

`this.`를 사용해야 할 때가 두 번 있습니다. 

첫째, 접근하고자 하는 멤버를 같은 이름을 가진 지역 변수로 섀도우 할 때 입니다:

```dart
// Bad 🙅
class Box {
  Object? value;

  void clear() {
    this.update(null); // 불필요
  }

  void update(Object? value) {
    this.value = value;
  }
}

// Good 🙆
class Box {
  Object? value;

  void clear() {
    update(null);
  }

  void update(Object? value) {
    this.value = value;
  }
}
```

둘째로 `this.`를 사용할 때는 명명된 생성자로 경로를 변경할 때 입니다:

```dart
// Bad 🙅
class ShadeOfGray {
  final int brightness;

  ShadeOfGray(int val) : brightness = val;

  ShadeOfGray.black() : this(0);

  // This won't parse or compile!
  // ShadeOfGray.alsoBlack() : black();
}

// Good 🙆
class ShadeOfGray {
  final int brightness;

  ShadeOfGray(int val) : brightness = val;

  ShadeOfGray.black() : this(0);

  // But now it will!
  ShadeOfGray.alsoBlack() : this.black();
}
```

생성자 파라미터는 constructor initializer lists에서 절대 필드를 섀도우 하지 않습니다.

```dart
class Box extends BaseBox {
  Object? value;

  Box(Object? value)
      : value = value,
        super(value);
}
```

#### DO 가능하다면 선언할 때 필드를 초기화 하세요.

만약, 필드가 어떤 생성자 파라미터에도 의존하고 있지 않다면 가능한 한 그리고 최대한 선언할 때 초기화 해야 합니다. 그렇게 해야 한 클래스가 복수의 생성자를 가지고 있을 때, 더 적은 코드로 짤 수 있고, 중첩을 피할 수 있습니다.

```dart
// Bad 🙅
class ProfileMark {
  final String name;
  final DateTime start;

  ProfileMark(this.name) : start = DateTime.now();
  ProfileMark.unnamed()
      : name = '',
        start = DateTime.now();
}

// Good 🙆
class ProfileMark {
  final String name;
  final DateTime start = DateTime.now();

  ProfileMark(this.name);
  ProfileMark.unnamed() : name = '';
}
```

`this`를 참조해야 하는 어떤 필드들은 선언할 때 초기화 할 수 없을 수도 있습니다. 예를 들면, 다른 필드를 사용한다던지 다른 method를 호출하는 경우가 있을 겁니다. 그러나, 만약 `late`으로 표시된 필드가 있다면 initializer는 `this`에 _접근할 수_ 있습니다.

물론 한 필드가 생성자 파라미터에 의존하고 있다거나 다른 생성자에 의해 다르게 초기화 되었다면, 이 가이드라인은 적용되지 않습니다.

### Constructors

#### DO 가능하면 매개변수 초기화 방식으로(initializing formal) 초기화 하세요:

```dart
// Bad 🙅
class Point {
  double x, y; // 1번
  Point(double x, double y) // 2번
      : x = x, // 3번
        y = y; // 4번
}

// Good 🙆
class Point {
  double x, y; // 1번
  Point(this.x, this.y); // 2번
}
```

생성자 파라미터 이전의 `this.` 문법을 "매개변수 초기화(initializing formal)"라고 부릅니다. 항상 이 방법의 장점을 취할 수는 없을 겁니다. 때때로 초기화하려는 필드의 이름과 다른 생성자 파라미터를 원할 때가 있을 테니까요. 하지만 매개변수 초기화를 _사용할 수 있다면 사용해야 합니다._

#### DON'T Constructor initializer list로 가능하다면 `late`을 사용하지 마세요.

다트는 당신에게 non-nullable 필드가 읽기 전에 초기화 되도록 요구합니다. 필드는 생성자 몸체(constructor body) 안에서 읽을 수 있기 때문에, 만약 생성자 몸체가 돌기 전에 non-nullable 필드를 초기화하지 않는다면 에러가 발생하게 됩니다.

이 에러를 없애기 위해 필드에 `late` 표시를 하는 방법이 있을 수 있습니다. 이렇게 표시하면 초기화 되기 전에 필드에 접근할 경우, compile-time 에러에서 _runtime error_ 로 전환됩니다. 어떤 경우에서는 이 방법이 필요할 수도 있겠습니다만, 올바른 수정 방법은 종종 constructor initializer list가 될 수 있습니다:

```dart
// Good 🙆
class Point {
  double x, y;
  Point.polar(double theta, double radius)
      : x = cos(theta) * radius, // constructor initializer list
        y = sin(theta) * radius;
}

// Bad 🙅
class Point {
  late double x, y;
  Point.polar(double theta, double radius) { // Body
    x = cos(theta) * radius;
    y = sin(theta) * radius;
  }
}
```

Initializer list는 생산자 파라미터에 접근할 수 있게 해주고, 필드를 읽기 전에 해당 필드를 초기화 할 수 있게 합니다. 그래서 만약 initializer list를 사용할 수 있다면 가능한 한 사용하는 것이, `late` 표시를 해 정적 안정성(static safety)과 퍼포먼스 성능을 잃는 것보다 나은 선택입니다.

#### DO 비어 있는 생산자 몸체(constructor bodies)에 대해서는 `{}` 대신 `;`을 사용하세요.

다트에서 빈 몸체를 가지고 있는 생산자는 semicolon 하나로 끝낼 수 있습니다. (사실, const 생산자에 대해서는 필요하기는 합니다.)

```dart
// Good 🙆
class Point {
  double x, y;
  Point(this.x, this.y);
}

// Bad 🙅
class Point {
  double x, y;
  Point(this.x, this.y) {}
}
```

#### DON'T `new`를 사용하지 마세요.

생산자를 호출할 때 `new` 키워드를 붙이는 것은 선택입니다. 팩토리 생산자(factory constructor)는 `new`를 적용하는 것이 꼭 새로운 오브젝트를 반환한다고 보기 어렵기 때문입니다.

다트는 `new`를 여전히 사용할 수 있도록 지원하고 있으나 더이상 사용되지 않거나 피해야 할 것으로 생각하세요.

```dart
// Good 🙆
Widget build(BuildContext context) {
  return Row(
    children: [
      RaisedButton(
        child: Text('Increment'),
      ),
      Text('Click!'),
    ],
  );
}

// Bad 🙅
Widget build(BuildContext context) {
  return new Row(
    children: [
      new RaisedButton(
        child: new Text('Increment'),
      ),
      new Text('Click!'),
    ],
  );
}
```

#### DON'T `const`를 불필요하게 많이 쓰지 마세요.

문맥 상 표현이 _constant여야만_ 하고 `const` 키워드의 의미가 내포됐으며 쓸 필요가 없고 사용해서는 안 되는 상황에서는 불필요하게 많이 사용하지 않도록 합니다. 그런 맥락적 표현은 다음과 같은 것을 포함합니다:

- const collection literal
- const constructor 호출
- 메타데이터 어노테이션
- const 변수 선언에 대한 initializer
- switch case 표현 - case의 몸체가 아닌 `case`와 `:` 사이

(다트의 미래 버전은 non-const 기본값(default values)을 지원할 수 있으므로 이 리스트에는 기본값은 포함하지 않습니다.)

기본적으로, 다트에서는 `const` 대신에 `new`를 작성하라는 에러가 나는 곳 어디든 `const`를 제외시킬 수 있도록 허용합니다.

```dart
// Good 🙆
const primaryColors = [
  Color('red', [255, 0, 0]),
  Color('green', [0, 255, 0]),
  Color('blue', [0, 0, 255]),
];

// Bad 🙅
const primaryColors = const [
  const Color('red', const [255, 0, 0]),
  const Color('green', const [0, 255, 0]),
  const Color('blue', const [0, 0, 255]),
];
```

### Error handling

다트는 당신의 프로그램에서 에러가 발생하면 exceptions를 사용합니다.

#### AVOID `on` 절 없이 catch 절 사용을 피하세요.

`on` 식별자가 없는 catch 절은 try 블록 안에서 잡힌 _모든 것_ 을 잡아냅니다. [Pokémon exception handling](https://blog.codinghorror.com/new-programming-jargon/)을 원하던 것이 아니라면요.

대부분의 경우에서 `on` 절을 사용해 당신이 주시하고 있고 적절하게 다루고자 하는 런타임 실패 종류의 한계를 설정해야 합니다.

매우 드물게, 어떤 런타임 에러라도 잡고 싶을 수 있습니다. 이는 대개 프레임워크 혹은 저수준 코드 안에서 추상적인 애플리케이션 코드를 문제를 만드는 원인들로부터 보호하기 위해 사용됩니다. 심지어 여기서도 모든 타입의 에러를 catch 하는 것보다 Exception 클래스를 catch 하는 것이 더 좋은 선택입니다. Exception은 모든 _런타임 에러_ 에 대한 기본 클래스를 말하고 코드 안에서의 _프로그래매틱_ 버그를 가리키는 에러사항은 배제합니다.

#### DON'T `on` 절 없이 catch를 사용해 에러를 버리는 것을 지양하세요.

만약 정말 일부 코드군에서 발생하는 _모든_ 에러를 catch 할 필요가 있다고 느낀다면 catch 하는 것에 대한 _어떤 처리를 하세요._ 로그를 남긴다던지, 유저에게 보여준다던지 혹은 다시 에러를 발생시킨다던지 말입니다. 다만 조용하게 버리지만 마세요.

#### DO 프로그래매틱 에러에 대해서만 `Error`를 구현(implement)한 오브젝트를 에러로 던지도록 하세요.

에러 클래스는 _프로그래매틱_ 에러에 대한 기본 클래스입니다. 해당 타입의 오브젝트 혹은 ArgumentError와 같은 subinterface 중 하나가 에러로 발생한다면 이는 코드 상에 _버그_ 가 있다는 것을 의미합니다. 당신의 API를 호출한 곳에 잘못 사용하고 있다고 에러를 던지고 싶을 때 사용할 수 있습니다.

역으로, 만약 Exception이 런타임 실패의 어떤 종류라면 이는 코드 상의 버그를 가리키지 않습니다. 따라서 Error을 throw 하는 것은 잘못된 신호를 날리는 겁니다. 대신, Core Exception 혹은 다른 타입을 throw 하는 것이 좋습니다.

#### DON'T 명시적으로 Error를 catch하거나 Error 클래스를 구현하는 타입을 catch 하지 마세요.

이 내용은 위의 내용에 이어집니다. Error는 코드 상의 버그를 가리키기 때문에 모든 callstack을 풀고, 프로그램을 멈추며 스택 추적을 프린트해 버그의 출처를 찾아 수정할 수 있게 합니다.

이런 종류의 에러를 catch 하는 것은 진행 중이었던 것을 중지시키고 버그 표시를 합니다. 사후에 이러한 exception을 다루는 error-handling code를 더하는 대신, 문제의 출처가 되는 코드로 돌아가 고치도록 하세요.

#### DO 잡은 exception을 rethrow 할 때는 `rethrow`를 사용하세요.

만약 exception을 rethrow 하기로 결정했다면, `throw`를 사용해 같은 exception 오브젝트를 던지는 것 대신에 `rethrow`를 사용하도록 하세요. `rethrow`는 해당 예외의 본래의 stack trace를 보존하니까요. 반면에 `throw`를 사용하게 된다면 마지막에 던져진 시점의 stack trace로 초기화 됩니다.

```dart
// Bad 🙅
try {
  somethingRisky();
} catch (e) {
  if (!canHandle(e)) throw e;
  handle(e);
}

// Good 🙆
try {
  somethingRisky();
} catch (e) {
  if (!canHandle(e)) rethrow;
  handle(e);
}
```

### Asyncrony

#### PREFER 기본 상태의 futures를 사용하는 것보다 async/await을 사용하세요.

비동기 코드는 futures와 같은 훌륭한 관념을 사용한다고 하더라도 읽기 힘들고 디버깅도 힘들기로 악명이 높습니다. `async/await` 문법은 가독성을 높여주고, 비동기 코드에서 다트의 모든 컨트롤 흐름의 구조를 사용할 수 있게 해줍니다.

```dart
// Good 🙆
Future<int> countActivePlayers(String teamName) async {
  try {
    var team = await downloadTeam(teamName);
    if (team == null) return 0;

    var players = await team.roster;
    return players.where((player) => player.isActive).length;
  } catch (e) {
    log.error(e);
    return 0;
  }
}

// Bad 🙅
Future<int> countActivePlayers(String teamName) {
  return downloadTeam(teamName).then((team) {
    if (team == null) return Future.value(0);

    return team.roster.then((players) {
      return players.where((player) => player.isActive).length;
    });
  }).catchError((e) {
    log.error(e);
    return 0;
  });
}
```

#### DON'T 효용성 있는 효과가 없다면 `async`를 사용하지 마세요.

비동기와 관계된 어떤 함수나 `async`를 사용하는 습관이 쉽게 생길 수 있습니다. 하지만 몇몇 경우에는 관계없을 수 있습니다. 만약 함수의 행위를 변화시키지 않고도 `async`을 뺄 수 있다면 그렇게 하세요.

```dart
// Good 🙆
Future<int> fastestBranch(Future<int> left, Future<int> right) {
  return Future.any([left, right]);
}

// Bad 🙅
Future<int> fastestBranch(Future<int> left, Future<int> right) async {
  return Future.any([left, right]);
}
```

`async`가 유용한 경우는 다음과 같습니다:

- `await`을 사용할 때 사용합니다.(이 때는 명백히 사용할 때 입니다.)
- 비동기적으로 에러를 반환할 때 사용합니다. `async`를 사용하고 `throw`를 사용하는 것이 `return Future.error(...)`보다 짧습니다.
- 값을 반환하고 이 때, future로 감싸져 있다고 암시하길 원할 때 사용합니다. `async`가 `Future.value(...)`보다 짧습니다.

```dart
Future<void> usesAwait(Future<String> later) async {
  print(await later);
}

Future<void> asyncError() async {
  throw 'Error!';
}

Future<String> asyncValue() async => 'value';
```

#### CONSIDER Stream 변환에 대해서는 고차 함수를 사용합니다.

#### AVOID Completer을 직접적으로 사용하는 것을 피하세요.

비동기 프로그래밍을 새로 접하는 많은 사람들이 future를 생산하는 코드를 작성하기를 원합니다. Future의 constructors는 그러한 프로그래머들에게 맞지 않는 것처럼 보여 결국에는 Completer 클래스를 찾아 사용하게 됩니다.

```dart
// Bad 🙅
Future<bool> fileContainsBear(String path) {
  var completer = Completer<bool>();

  File(path).readAsString().then((contents) {
    completer.complete(contents.contains('bear'));
  });

  return completer.future;
}
```

Completer는 두 종류의 저수준 코드에 필요합니다: 새 비동기 기본 요소(primitive)와 futures를 사용하지 않는 비동기 코드에 접속하는 것이죠. 대부분의 다른 코드는 async/await 또는 `Future.then()`을 사용해야 합니다. 그래야 더 깔끔하기도 하고 에러 핸들링이 쉽기 때문입니다.

```dart
// Good 🙆
Future<bool> fileContainsBear(String path) {
  return File(path).readAsString().then((contents) {
    return contents.contains('bear');
  });
}
```

```dart
// Good 🙆
Future<bool> fileContainsBear(String path) async {
  var contents = await File(path).readAsString();
  return contents.contains('bear');
}
```

#### DO Type argument가 `Object`일 수 있는 `FutureOr<T>`의 타입을 명확히 지정할 때, `Future<T>`로 테스트 하세요.

`FutureOr<T>`를 활용해 어떤 유용한 것을 하기 전에 보통 `is`를 통해 `Future<T>`인지 기본 `T`인지 확인할 필요가 있습니다. 만약 type argument가 `FutureOr<int>`와 같은 특정 타입이라면 `is int`를 쓰던 혹은 `is Future<int>`를 쓰던 문제가 되지 않습니다. 두 타입은 서로 연결성이 없기 때문에 어느 쪽이라도 작동합니다.

그러나 만약 value type이 `Object`이거나 type parameter가 `Object`로 객체화 될 수 있는 가능성이 있다면 두개의 분기가 겹칠 수 있습니다. `Future<Object>`는 `Object`를 구현하고 있어서 `is Object` 혹은 `Object`로 객체화 될 수 있는 type parameter로서의 `T`를 가지고 있는 `is T`는 오브젝트가 future라고 하더라도 true를 반환합니다. 대신에, 명시적으로 Future 케이스를 테스트합니다:

```dart
// Good 🙆
Future<T> logValue<T>(FutureOr<T> value) async {
  if (value is Future<T>) {
    var result = await value;
    print(result);
    return result;
  } else {
    print(value);
    return value;
  }
}
```

```dart
// Bad 🙅
Future<T> logValue<T>(FutureOr<T> value) async {
  if (value is T) {
    print(value);
    return value;
  } else {
    var result = await value;
    print(result);
    return result;
  }
}
```

Bad 예시에서, 만약 `Future<Object>`를 넘긴다면 부정확하게 기본 오브젝트로 받아들여 동기 값으로 취급합니다.

## 디자인

### Names

이름 짓기(Naming)는 가독성 높고 유지 가능한 코드를 작성하는데 있어 중요한 부분입니다.

#### DO 일관적으로 용어를 사용하세요.

코드 전반에서 같은 것에 대해서는 같은 이름을 쓰세요. 만약 당신의 API 외부에 사용자들이 알 법한 선례가 존재한다면, 선례를 따르세요.

```dart
// Good 🙆
pageCount         // A field.
updatePageCount() // Consistent with pageCount.
toSomething()     // Consistent with Iterable's toList().
asSomething()     // Consistent with List's asMap().
Point             // A familiar concept.
```

```dart
// Bad 🙅
renumberPages()      // Confusingly different from pageCount.
convertToSomething() // Inconsistent with toX() precedent.
wrappedAsSomething() // Inconsistent with asX() precedent.
Cartesian            // Unfamiliar to most users.
```

이 방식의 목적은 사용자들이 이미 알고 있는 것에 대한 이점을 활용하는 것에 있습니다. 이는 이 문제가 해당하는 영역 자체(관련된 코드 범위)에 대한 지식과 코어 라이브러리의 컨벤션 그리고 당신의 API의 다른 부분들을 포함하게 됩니다. 이러한 방식의 도입하게 되면, 새로운 지식을 습득해야 하는 양을 줄여 사용자는 보다 빠르게 코드를 활용할 수 있게 됩니다.

#### AVOID 축약어를 피하세요.

축약어가 축약하지 않은 용어보다 더 일반적이지 않은 이상 축약하지 마세요. 대문자를 일부 활용해 올바르게 표시하도록 합니다.

```dart
// Good 🙆
pageCount
buildRectangles
IOStream
HttpRequest
```

```dart
// Bad 🙅
numPages    // "Num" is an abbreviation of "number (of)".
buildRects
InputOutputStream
HypertextTransferProtocolRequest
```

#### PREFER 가장 기술적(descriptive) 명사는 마지막에 넣는 것을 추천합니다.

마지막 단어는 표현하고자 하는 것을 가장 잘 의미하는 가장 기술적인 단어를 사용하도록 합니다. 이에 더해 접두사에 형용사 등을 통해 좀 더 자세히 묘사할 수 있습니다.

```dart
// Good 🙆
pageCount             // A count (of pages).
ConversionSink        // A sink for doing conversions.
ChunkedConversionSink // A ConversionSink that's chunked.
CssFontFaceRule       // A rule for font faces in CSS.
```

```dart
// Bad 🙅
numPages                  // Not a collection of pages.
CanvasRenderingContext2D  // Not a "2D".
RuleFontFaceCss           // Not a CSS.
```

#### CONSIDER 코드가 문장처럼 읽히게 만드세요.

이름 짓기에 대해 의심이 들 때, 당신의 API를 사용하는 코드를 작성해서 문장처럼 읽히는지 확인해보세요.

```dart
// Good 🙆
// "If errors is empty..."
if (errors.isEmpty) ...

// "Hey, subscription, cancel!"
subscription.cancel();

// "Get the monsters where the monster has claws."
monsters.where((monster) => monster.hasClaws);
```

```dart
// Bad 🙅
// Telling errors to empty itself, or asking if it is?
if (errors.empty) ...

// Toggle what? To what?
subscription.toggle();

// Filter the monsters with claws *out* or include *only* those?
monsters.filter((monster) => monster.hasClaws);
```

당신의 API를 사용해서 코드 상에서 어떻게 "읽히는지" 보는 것은 도움이 됩니다. 그러나 너무 가는 경우도 있죠😅. 기사를 작성하듯 하거나 이름이 _말 그대로_ 문법적으로 정확한 문장이 되도록 강제하는 것은 오히려 도움이 되지 않습니다.

```dart
// Bad 🙅
if (theCollectionOfErrors.isEmpty) ...

monsters.producesANewSequenceWhereEach((monster) => monster.hasClaws);
```

#### PREFER Non-boolean 프로퍼티 혹은 변수에 대해 명사구 사용을 추천합니다.

독자의 관점은 해당 프로퍼티가 무엇이냐에 초점이 맞춰져 있습니다. 만약 사용자가 _어떻게_ 프로퍼티가 결정됐는지에 대해 더 신경을 쓰고 있다면, 아마도 그것은 동사구 이름을 가진 method일 겁니다.

```dart
// Good 🙆
list.length
context.lineWidth
quest.rampagingSwampBeast
```

```dart
// Bad 🙅
list.deleteItems
```

#### PREFER Boolean 프로퍼티 혹은 변수에 대해 명령형이 아닌(non-imperative) 동사구 사용을 추천합니다.

Boolean 이름은 종종 통제 흐름(control flow)에서 사용되고는 합니다. 다음 중 어떤 것이 더 잘 읽히는지 비교해보세요:

```dart
if (window.closeable) ...  // 형용사.
if (window.canClose) ...   // 동사.
```

좋은 이름은 동사 몇 종류 중 하나로 시작하는 경향이 있습니다:

- "to be" 형태: `isEnabled`, `wasShown`, `willFire`. 이러한 것들은 가장 흔한 형태입니다.
- 조동사 형태: `hasElements`, `canClose`, `shouldConsume`, `mustSave`.
- 능동동사 형태: `ignoresInput`, `wroteFile`. 이것들은 그 의미가 모호하기 때문에 거의 사용하지 않습니다. `loggedResult`는 "결과가 기록 되었는지 혹은 그렇지 않은지" 또는 "기록된 결과"라는 다소 중의적 표현이 될 수 있으므로 좋지 않은 이름입니다. 마찬가지로, `closingConnection`은 "연결이 끊어지고 있는지 혹은 그렇지 않은지" 또는 "끊어지고 있는 연결" 두 가지를 의미할 수 있습니다. 능동동사는 이름이 _오직_ 서술부로 읽혀질 때만 허용됩니다.

이러한 모든 동사구가 method 이름과 구별되는 점은 _명령형_ 이 아니라는 점에 있습니다. Boolean 이름은 절대 오브젝트에 어떤 것을 하라고 하는 명령어처럼 들려서는 안 됩니다. 왜냐하면 프로퍼티에 접근하는 것이 오브젝트를 변경하지는 않기 때문입니다.(만약 프로퍼티가 오브젝트를 의미있는 방식으로 _수정한다면_ 그것은 method여야 합니다.)

```dart
// Good 🙆
isEmpty
hasElements
canClose
closesWindow
canShowPopup
hasShownPopup
```

```dart
empty         // Adjective or verb?
withElements  // Sounds like it might hold elements.
closeable     // Sounds like an interface.
              // "canClose" reads better as a sentence.
closingWindow // Returns a bool or a window?
showPopup     // Sounds like it shows the popup.
```

#### CONSIDER 명명된 boolean 파라미터에 대해서는 동사를 제외하는 것을 고려하세요.

이 규칙은 직전의 규칙을 개선하는데 사용할 수 있습니다. Boolean인 명명된 파라미터에 대해 이름에 동사가 없더라도 종종 그 자체로 확실합니다. 그리고 그런 코드는 호출 지점에서 가독성을 더욱 높일 수 있죠.

```dart
// Good 🙆
Isolate.spawn(entryPoint, message, paused: false);
var copy = List.from(elements, growable: true);
var regExp = RegExp(pattern, caseSensitive: false);
```

#### PREFER Boolean 프로퍼티 혹은 변수에 대해 "긍정적인" 이름을 만들 것을 추천합니다.

대부분의 boolean 이름은 개념적으로 "긍정적인" 그리고 "부정적인" 형태를 띄고 있는데, 긍정적인 것은 근본적인 개념을 뜻하는 것처럼 느껴지고 부정적인 것은 반대를 뜻하는 것처럼 느껴집니다-"open" 그리고 "closed", "enabled" 그리고 "disabled" 등. 종종 이후의 이름이 말 그대로 긍정적인 것을 부정하는 접두사를 가지고 있습니다: "visible" 그리고 "in-visible", "connected" 그리고 "dis-connected", "zero" 그리고 "non-zero".

이름에 따라 `true`를 나타내는 두 가지 경우에서 하나를 선택할 때, 긍정적이고 좀 더 근본적인 것을 선택하세요. 만약 당신의 프로퍼티가 그 자체로서 부정하는 것처럼 읽혀진다면, 독자로 하여금 정신적으로 두 번의 부정을 연산하게 해 코드가 어떤 의미를 가지는지 이해하기 어렵게 만듭니다.

```dart
// Good 🙆
if (socket.isConnected && database.hasData) {
  socket.write(database.read());
}
```

```dart
// Bad 🙅
if (!socket.isDisconnected && !database.isEmpty) { // 이중 부정
  socket.write(database.read());
}
```

어떤 프로퍼티들에 대해서는 명확한 긍정적인 형태가 없을 수 있습니다. 디스크에 저장 되어 있는 문서는 "saved"인 것일까요? 아니면 "un-changed"인 것일까요? 만약 문서가 아직 기록되지 않았다면 "un-saved"인 것일까요? 아니면 "changed"인 것일까요? 이처럼 모호한 경우에서는 덜 부정적인 선택이나 좀 더 짧은 이름에 기댑니다.

**예외**: 몇몇 프로퍼티에 대해서는 부정적인 형태가 압도적으로 사용될 필요가 있는 것들이 있습니다. 긍정 케이스를 선택하는 것은 모든 곳에서 프로퍼티에 `!`를 붙여 긍정적인 경우를 부정하도록 강제하는 경우가 있을 수 있습니다. 이런 경우에는 해당 프로퍼티에 부정 케이스를 사용하는 것이 더 나을 수 있습니다.

#### PREFER 함수 또는 주 목적이 부작용인 method에 대해서 명령형 동사구 사용을 추천합니다.

호출 가능한 요소들은 호출자에게 결과값을 반환하고 다른 작업을 수행하거나 부작용을 수행합니다. 다트와 같은 명령 언어에서 요소들은 종종 주로 부작용을 위해 호출 되고는 합니다: 이 부작용에는 오브젝트 안의 상태를 변화시키거나 결과를 만들어내거나 바깥 세계에 전달하는 작용이 있습니다.

이러한 종류의 요소들은 명령형 동사구로 명명해 해당 요소가 성취하려는 작업을 명확하게 전달해야 합니다.

```dart
// Good 🙆
list.add('element');
queue.removeFirst();
window.refresh();
```

#### PREFER 만약 값을 반환하는 것이 첫번째 목적이라면 명사구나 비명령형 동사구를 사용하는 것을 추천합니다.

부작용이 조금 있으나 유용한 결과를 호출자에게 반환하는 것이 주요 목적인 요소들이 있을 수 있습니다. 만약 그러한 요소들이 파라미터 없이 역할을 수행한다면 일반적으로 getter라고 할 겁니다. 그러나 때로는 논리적 "프로퍼티"에 몇몇 파라미터가 필요하기도 합니다. 예를 들어, `elementAt()`은 컬렉션에서 데이터의 한 부분을 반환합니다. 하지만 _어떤 데이터의_ 일부를 반환해야 할지에 대한 파라미터를 필요로 합니다.

이는 이 요소가 _문법적으로_ method라는 것을 의미하나 _개념적으로_ 프로퍼티라는 것을 의미합니다. 그렇기에 이 때는 이 요소가 _반환하는 것을_ 서술할 구로 명명하는 것이 타당합니다.

```dart
// Good 🙆
var element = list.elementAt(3);
var first = list.firstWhere(test);
var char = string.codeUnitAt(4);
```

때때로 method는 부작용이 없어도 `list.take()` 또는 `string.split()`이라고 이름 짓는 것이 더 단순할 때도 있기에 좀 더 세심하게 적용하는 것이 좋습니다.

#### CONSIDER 만약 함수 또는 method가 성취하는 작업에 대해 이목을 끌고 싶다면 명령형 동사구를 사용하는 것을 고려해보세요.

어느 한 요소가 어떤 부작용 없이 결과를 생성할 때, 대개는 getter가 되거나 반환하는 결과 값을 서술하는 명사구 이름을 가진 method가 되어야 합니다. 하지만 때때로 결과를 생산하는데 있어 요구되는 작업이 중요한 경우가 있습니다. 런타임 실패, 혹은 네트워킹 또는 I/O와 같은 무거운 리소스를 사용하는 것이 될 수 있습니다. 이런 경우에서는 호출자로 하여금 그 요소가 하는 일을 생각하게끔 해주고 싶은 곳에서 요소에 동사구에 작업을 서술하는 이름을 지어주면 됩니다.

```dart
// good 🙆
var table = database.downloadData();
var packageVersions = packageGraph.solveConstraints();
```

그럼에도 불구하고 이 가이드라인은 이전의 두 가지 보다 엄격하지 않다는 사실을 인지하는 것이 좋습니다. 연산자가 성취하는 작업은 종종 호출자와 연관되지 않은 실행 디테일을 가졌고 성능과 강력함의 경계는 시간이 지나며 변합니다. 대개의 경우, 요소들에 대한 이름을 지을 때, _어떻게_ 하는지 보다는 _무엇을_ 해주는지에 초점을 맞춰 명명하도록 합니다.

#### AVOID method 이름을 지을 때, `get`으로 시작하는 것을 피하세요.

대부분의 경우, method는 이름에 `get`이 없는 상태로 getter가 되어야 합니다. 예를 들어, `getBreakfastOrder()`라고 method를 명명하는 대신 `breakfastOrder`라고 명명한 getter를 정의하는 거죠.

인자들(arguments)이 필요해 요소를 method로 만들어야 하거나 getter로 만들기에는 적절하지 않다고 판단했다고 하더라도 여전히 `get`이라는 이름을 붙이는 것은 피해야 합니다. 이전 가이드라인을 참고해 이름을 지을 수 있습니다:

- 만약 호출자가 method가 반환하는 값에 많은 신경을 쓰고 있다면, 간단하게 `get`을 없애고 `breakfastOrder()`와 같은 명사구 이름을 사용하면 됩니다.
- 호출자가 결과에 대해 신경을 쓰지만 `get`보다 좀 더 작업을 서술하는데 적합한 동사를 선택하고자 한다면 동사구 이름을 사용하세요. 예를 들어, `create`, `download`, `fetch`, `calculate`, `request`, `aggregate` 등이 있습니다.

#### PREFER 오브젝트의 상태를 복사해 새로운 오브젝트를 만든다면 `to ___()`라는 이름을 가진 method로 만들기를 추천합니다.

_전환_ method는 수신 받는 대부분의 모든 상태를 복사해 새로운 오브젝트를 만들어 반환하지만 약간 다른 형태 또는 표현을 가집니다. 코어 라이브러리들은 컨벤션으로 결과의 종류에 따라 `to`로 시작되는 method들을 가지고 있습니다.

만약 당신이 컨버전 method를 정의한다면, 다음의 컨벤션을 따르는 것이 도움이 될 겁니다.

```dart
// Good 🙆
list.toSet();
stackTrace.toString();
dateTime.toLocal();
```

#### PREFER 원(original) 오브젝트에 의해 지지 되는 다른 표현을 가진 것을 반환한다면 `as ___()` method라고 명명하는 것을 추천합니다.

전환 method는 "스냅샷"입니다. 결과 오브젝트는 원 오브젝트 상태의 복사본을 가지고 있습니다. _views_ 를 반환하는 method와 같이 다른 종류의 전환 method도 있지만 이 method도 원본 오브젝트를 참조하고 있다는 것은 같습니다. 그렇기에 원본 오브젝트에 대한 변화는 view에 반영됩니다.

코어 라이브러리 컨벤션은 `as ___()` 입니다.

```dart
// Good 🙆
var map = table.asMap();
var list = bytes.asFloat32List();
var future = subscription.asFuture();
```

#### AVOID 함수 또는 method 이름에 파라미터를 서술하는 것을 피하세요.

사용자는 호출 지점에서 필요 인자를 볼 것이기 때문에 이름에 참고할 내용을 써 놓는다고 하더라도 가독성에 도움을 주지는 않습니다.

```dart
// Good 🙆
list.add(element);
map.remove(key);v
```

```dart
// Bad 🙅
list.addElement(element)
map.removeKey(key)
```

그러나 다른 타입을 가지는 이름이 비슷한 method를 구분짓기 위해 파라미터를 언급하는 것은 유용할 수 있습니다:

```dart
// Good 🙆
map.containsKey(key);
map.containsValue(value);
```

#### DO 타입 파라미터를 명명할 때, 연상되는 컨벤션이 존재한다면 따르세요.

한 문자 이름이 정확하지는 않지만 거의 모든 제네릭 타입은 한 문자 이름을 사용합니다. 운이 좋게도 대부분 지속적이고, 연상이 되는 방식으로 사용하고 있습니다. 컨벤션은 다음과 같습니다:

- `E`는 컬렉션에서 **element** 타입을 말합니다:

```dart
// Good 🙆
class IterableBase<E> {}
class List<E> {}
class HashSet<E> {}
class RedBlackTree<E> {}
```

- `K`와 `V`는 연상 컬렉션에서 각각 **key**와 **value** 타입을 의미합니다:

```dart
// Good 🙆
class Map<K, V> {}
class Multimap<K, V> {}
class MapEntry<K, V> {}
```

- `R`은 함수 또는 클래스의 method **reutrn** 타입으로 사용됩니다. 사용할 때가 흔하지는 않지만 타입 정의에서 가끔 사용하기도 하며 visitor 패턴을 구현하는 클래스에서도 사용하기도 합니다.(visitor 패턴: 특정 데이터 구조에 새로운 연산을 추가하기 위한 디자인 패턴으로 기존에 정의된 클래스 구조를 변경하지 않고 새로운 기능을 추가할 수 있는 유연한 방법을 제공합니다.)

```dart
// Good 🙆
abstract class ExpressionVisitor<R> {
  R visitBinary(BinaryExpression node);
  R visitLiteral(LiteralExpression node);
  R visitUnary(UnaryExpression node);
}
```

- 위의 타입에 해당하지 않는 경우, 제네릭에 `T`, `S` 그리고 `U`를 사용할 수 있습니다. 이 타입들은 싱글 타입 파라미터와 의미가 명백한 타입들에 둘러쌓여 있는 경우에 사용합니다.

```dart
// Good 🙆
class Future<T> {
  Future<S> then<S>(FutureOr<S> onValue(T value)) => ...
}
```

만약 위의 경우들 중 적당한 것을 못 찾았다면, 연상 가능한 단문자 이름 또는 서술적 이름을 써도 괜찮습니다:

```dart
// Good 🙆
class Graph<N, E> {
  final List<N> nodes = [];
  final List<E> edges = [];
}

class Graph<Node, Edge> {
  final List<Node> nodes = [];
  final List<Edge> edges = [];
}
```

### Libraries

앞에 언더스코어 문자(`_`)가 있는 요소는 라이브러리에서 private으로 사용되고 있음을 나타냅니다. 이것은 단지 컨벤션을 나타내는 것이 아니라 다트 언어 자체에 그렇게 설계되어 있기도 합니다.

#### PREFER 선언은 private으로 만들 것을 추천합니다.

라이브러리 안에서 public 선언은 다른 라이브러리들이 이 요소에 접근이 가능하거나 접근해야 한다는 신호를 줍니다. 이것은 라이브러리의 해당 부분이 그 public 선언을 지원해야 하고 호출이 되었을 때 적절히 동작해야 한다는 약속을 의미하기도 합니다.

만약 그게 의도한 바가 아니라면 `_` 하나로 행복을 찾으면 됩니다. 적은 수의 public interface는 유지보수 하기 쉬울 뿐 아니라 사용자가 배우기도 쉽습니다. 보너스로 다트의 분석기가 사용하지 않는 private 선언들에 대해 알려주므로 데드(dead) 코드를 쉽게 정리할 수 있습니다. 그런데 public이라면 외부에서 사용되고 있는지 여부를 확신할 수 없으므로 이러한 보너스를 누릴 수 없게 됩니다.

#### CONSIDER 같은 라이브러리 안에 복수의 클래스를 선언하는 것을 고려해보세요.

자바와 같은 몇몇 언어들에서는 파일들의 구조가 클래스의 구조에 매여 있습니다-각각의 파일은 오직 하나의 최상위 클래스를 정의합니다. 다트는 이런 한계가 없습니다. 라이브러리들은 클래스들과는 다른 별개의 뚜렷한 독립체입니다. 다수의 클래스들, 최상위 변수들 그리고 함수들이 논리적으로 화합되어 있다면 하나의 라이브러리 안에 포함되는 것이 가능하죠.

하나의 라이브러리 안에 복수의 클래스들을 놓는 것은 몇가지 유용한 패턴을 사용할 수 있게 합니다. 다트에서 프라이버시는 클래스 수준이 아닌 라이브러리 수준에서 동작하기 때문에 C++에서와 같이 "친구" 클래스들을 정의할 수 있습니다. 다트에서 같은 라이브러리 안에서 선언된 모든 클래스는 서로의 private 요소들에 접근할 수 있습니다.

물론, 이 가이드라인은 여러 클래스를 하나의 큰 라이브러리 안에 몽땅 _넣어야만_ 한다는 말은 아닙니다. 다만 하나의 라이브러리에 하나의 클래스만 할당해야 한다는 것은 아니라는 말이죠.

### Classes와 mixins

다트는 모든 오브젝트가 클래스의 객체인 "순수" 객체 지향 언어입니다. 하지만 다트는 모든 코드를 클래스 안에 정의할 것을 요구하지는 않습니다-당신은 최상위 변수들, 상수들 그리고 함수들을 통해 절차 지향적 혹은 함수 지향적 언어로 사용할 수 있죠.

#### AVOID 하나의 간단한 함수만 실행하는 경우, abstract class 사용을 피하세요.

만약 한개의 abstract 요소를 가진 클래스를 정의하기 위해 `call` 혹은 `invoke`와 같은 의미없는 이름을 사용할 때는 함수를 사용하는 것이 좋을 수 있습니다.

```dart
// Good 🙆
typedef Predicate<E> = bool Function(E element);
```

```dart
// Bad 🙅
abstract class Predicate<E> {
  bool test(E element);
}
```

#### AVOID 정적 요소만을 포함하는 클래스를 정의하는 것을 피하세요.

자바와 C#에서는 모든 정의가 _꼭_ 클래스 안에 있어야 하므로 "클래스"가 정적 요소를 집어 넣을 유일한 장소처럼 보이고는 합니다.어떤 다른 클래스들은 네임스페이스처럼 쓰여 서로 관련된 것들끼리 묶는 역할을 하거나 이름이 충돌하지 않도록 하는 역할을 합니다.

다트는 최상위 함수, 변수 그리고 상수들을 가지고 있어 어떤 것을 정의하기 위해 꼭 클래스를 사용할 필요는 없습니다. 만약 네임스페이스를 원한다면 라이브러리를 사용하는 것이 더 적절하죠. 라이브러리는 import prefix와 show/hide combinator를 지원하고 있어 사용자로 하여금 이름 충돌이 일어나지 않게 관리하는 방법을 제공합니다.

만약 함수 또는 변수가 논리적으로 클래스에 묶여 있지 않다면 최상위로 올리세요. 만약 이름 충돌이 염려된다면, 좀 더 정확한 이름을 주거나 접두사로 import 할 수 있는 별개의 라이브러리로 옮기도록 하세요.

```dart
// Good 🙆
DateTime mostRecent(List<DateTime> dates) {
  return dates.reduce((a, b) => a.isAfter(b) ? a : b);
}

const _favoriteMammal = 'weasel';
```

```dart
// Bad 🙅
class DateUtils {
  static DateTime mostRecent(List<DateTime> dates) {
    return dates.reduce((a, b) => a.isAfter(b) ? a : b);
  }
}

class _Favorites {
  static const mammal = 'weasel';
}
```

Constants와 enum 같은 타입은 하나의 클래스 안에 그룹화하는 것이 자연스러울 수 있습니다.

```dart
// Good 🙆
class Color {
  static const red = '#f00';
  static const green = '#0f0';
  static const blue = '#00f';
  static const black = '#000';
  static const white = '#fff';
}
```

#### AVOID 서브 클래스로 의도되지 않은 클래스는 확장을 피하세요.

클래스에 서브 클래스를 허용할지 여부에 대한 것은 신중해야 합니다. 따라서 문서 코멘트로 소통하거나 클래스에 `IterableBase`와 같은 명확한 이름을 주어 확장 가능 여부에 대해 헷갈리지 않게 해야 합니다.

#### DO 당신의 클래스에 확장이 지원 된다면 문서를 작성하세요.

이 규칙은 위의 규칙에 필연적입니다. 만약 당신이 클래스에 서브 클래스를 허용하기를 원한다면, 설명을 적으세요. 클래스 이름의 접미사로 `Base`를 넣거나 클래스의 문서 코멘트에 언급하세요.

#### AVOID Interface로 의도된 클래스가 아닌 이상 클래스를 구현하는 것을 피하세요.

다트에서는 클래스에 대한 인터페이스를 따로 구현하지 않고도 인터페이스 역할을 할 수 있는 암시적인 인터페이스라고 하는 강력한 도구를 제공합니다.

하지만 이러한 장점에도 불구하고 의도하지 않은 클래스를 구현할 경우 문제가 발생할 수 있습니다. 예를 들어, 클래스에 새로운 요소를 추가하는 것은 안전하지만 그 클래스의 인터페이스를 구현하고 있는 클래스는 정적 에러가 발생하게 됩니다.

#### DO 만약 당신의 클래스가 인터페이스로 사용된다면 문서를 작성하세요.

#### PREFER `mixin class`를 순수 `mixin` 또는 순수 `class`로 정의하세요.

다트는 2.12 버전부터 2.19버전까지는 다른 클래스에 믹스 되는데 특정 제약을 충족하는 클래스라면 어느 것이라도 허용해왔습니다.(non-default constructor가 없고 superclass가 없는 등.) 이는 클래스 저자가 믹스 되는 것을 의도하지 않았을 수도 있기 때문에 혼란을 유발했습니다.

다트 3.0.0부터는 다른 클래스에 믹스되기 위해서, 뿐만 아니라 노멀 클래스로 여겨지기 위해서는 `mixin class`로 명시적 선언을 해야 하는 것으로 변경 되었습니다.

그러나 mixin이 되어야 하면서 클래스도 되어야 하는 타입은 희귀한 경우입니다. `mixin class` 선언은 pre-3.0.0 클래스들이 mixins로 좀 더 명확하게 사용될 수 있도록 돕기 위한 용도입니다. 따라서 새로운 코드는 순수 `mixin` 또는 순수 `class` 선언을 사용해서 의도와 행위를 더 명확하게 정의해야 하며 mixin class의 모호성을 피해야 합니다.

### Constructors

다트의 생산자는 클래스 그리고 선택적인 식별자와 같은 이름을 가진 함수를 선언할 때 생성됩니다. 선택적인 식별자의 경우를 _명명된 생산자(Named constructors)_ 라고 합니다.

#### CONSIDER 클래스가 지원한다면 생산자에 `const`를 적용하는 것을 고려해보세요.

만약 모든 필드가 final이고, 생산자가 아무것도 하지 않고 초기화를 시킬 뿐이라면 생산자를 `const`로 만들 수 있습니다. 이렇게 함으로써 사용자가 constants가 필요한 지점(다른 더 큰 constants, switch cases, default parameter values 등의 내부)에 해당 클래스 객체를 생성하게 합니다.

만약 명시적으로 `const`로 만들지 않으면, 위의 사용 예시에서 사용할 수가 없습니다.

그러나 `const` 생성자는 당신의 public API에서의 약속이라는 점에 주의하세요. 만약 나중에 생성자를 non-`const`로 변경하게 되면 constant 표현들에서 그것을 호출하고 있는 사용자들을 깨뜨리게 됩니다. 만약 이렇게 약속하고 싶지 않다면, `const`를 만들지 마세요. 실제 사용할 때는 `const` 생성자를 통해, 불변성을 가진 값과 같은 타입을 간단하게 지정하는데 유용하게 사용할 수 있습니다.

### Members

하나의 오브젝트에 속한 요소(member)는 method 또는 객체 변수 둘 다 될 수 있습니다.

#### PREFER 필드와 최상위 변수를 `final`로 만들 것을 추천합니다.

연관되어 있는 변화 가능한 상태의 수를 최소화 한 클래스와 라이브러리들은 유지하기가 더 쉬운 경향이 있습니다. 물론 종종 변화 가능한 데이터를 가지는 것이 유용할 때도 있습니다. 하지만 그러한 변화 가능한 데이터가 필요하지 않다면, 기본적으로 필드와 최상위 변수를 가능한 한 `final`로 만드는 것이 좋습니다.

`final` 객체 필드는 초기화 된 이후에는 변하지 않지만, 때때로 그 객체가 구성된 이후 초기화 할 수 없어 문제가 생기는 경우가 있습니다. 예를 들어, 객체의 다른 필드나 `this`를 참고해야 할 때가 있다고 해보죠. 이럴 경우에는 `late final` 필드를 만들 것을 고려해보세요. 또, 선언 지점에서 초기화 하는 방법도 시도해볼 수 있습니다.

#### DO 개념적으로 프로퍼티에 접근하는 연산자에는 getter를 사용하세요.

한 요소를 getter로 할지 method로 할지 결정하는 것에 미묘한 차이만이 존재한다고 할 때, 좋은 API를 만들기 위한 디자인을 고려한다면 긴 설명이 불가피합니다. 어떤 언어들은 getter를 기피하는 문화가 있기도 합니다. 그러한 언어들은 거의 정확히 필드와 같을 때에만 getter 연산자를 사용하죠. 이 경우에서 getter 보다 좀 더 복잡하거나 무거운 경우, 이름 뒤에 `()`를 사용해 "계산이 여기부터 시작됩니다!"는 신호를 주게 됩니다. 왜냐하면 `.` 다음의 기본형의 이름은 "필드"를 의미하기 때문이죠.

다트는 그런 식으로 작동하지 _않습니다. 모든_ 마침표가 있는 이름들은 계산을 할지도 모르는 호출 가능한 것들입니다. 필드는 특별히 다트에 의해 제공되는 실행 getter 입니다. 달리 말해, 다트에서 getter는 "특별히 느린 필드"가 아닙니다; 필드가 "특별히 빠른 getter입니다".

그렇기는 해도 method를 넘어서 getter를 선택하는 것은 호출자에게 중요한 신호를 보내는 겁니다. 대충, 신호는 getter 연산자가 "필드와 같다"는 겁니다. 적어도 규칙 상, 호출자가 아는 한 연산자는 필드를 사용해서 실행될 _수도_ 있습니다. 이는 다음의 의미를 내포합니다:

- **연산자는 어떤 인자도 받지 않고 결과만을 반환합니다.**
- **호출자의 대부분이 주로 결과에 대해서 신경 씁니다.** 만약 호출자로 하여금 생산된 결과보다 getter 연산자가 결과를 생산하는 방식에 신경 쓰기를 원한다면, 연산자에 동사 이름을 주어 작업을 서술하고 메서드로 만들도록 합니다.

이는 getter 연산자가 getter가 되기 위해 특별히 빨라야 한다던지를 의미하는 것은 _아닙니다._ `IterableBase.length`는 `O(n)`을 따르고, 그래도 괜찮습니다. Getter가 커다란 계산을 해도 나쁠 게 없습니다. 그러나 만약 _놀랄 정도로_ 많은 양의 작업을 수행해야 한다면, 이름에 동사를 사용해 서술함으로써 해당 getter가 무슨 작업을 수행하는지 알리는 방법을 통해 의도를 전달할 수도 있습니다.

```dart
// Bad 🙅
connection.nextIncomingMessage; // Does network I/O.
expression.normalForm; // Could be exponential to calculate.
```

- **연산자가 사용자가 볼 수 있는 부작용을 가지지 않습니다.** 실재(real) 필드에 접근하는 것이 프로그램의 오브젝트 혹은 다른 어떤 상태를 변경시키지 않습니다. 연산자는 output을 생산하지도 파일을 기록하지도 않죠. Getter 연산자는 이러한 것들을 해서는 안 됩니다.

"사용자가 볼 수 있는" 부분이 중요합니다. Getter가 숨겨진 상태를 변경하거나 영역을 벗어나는 것을 생성하는 등의 부작용은 일으켜도 괜찮습니다. Getter는 느리게 계산하고 결과를 저장하고 캐시에 기록하고 생성 기록을 남기는 등의 작업을 수행할 수도 있습니다. 호출자가 부작용에 대해 _신경쓰지_ 않는 한 말이죠.

```dart
// Bad 🙅
stdout.newline; // Produces output.
list.clear; // Modifies object.
```

- **연산자가 멱등성을 가집니다.** "멱등성"이라는 단어가 생소하겠으나, 여기서는 문맥상 연산자를 여러번 호출해도 각각의 결과는, 여러번의 호출 사이에 명시적인 변동이 없는 한 같다는 뜻을 가집니다. (확실히, `list.length`는 호출 사이에 배열 요소를 추가하게 되면 다른 결과를 생산한다는 것을 통해 알 수 있습니다.)

여기서 "같은 결과"는 하나의 getter가 연속적인 호출에서, 말 그대로 동일한 오브젝트를 생산해야 함을 의미하지는 않습니다. "같은 결과"를 강제하게 되면 많은 getter들이 불안정한 캐시를 가지게 되고 이는 getter를 사용하는 이유들을 모두 무효화시키는 처사와 다름 없습니다. 그렇기에 호출시, 매번 새로운 future와 list를 반환하는 것은 흔하고도 완전히 문제 없는 것입니다. 중요한 부분은 결과값은 _호출자가 신경쓰는 부분에 대한 측면에서는_ 같을 수 밖에 없습니다.

```dart
// Bad 🙅
DateTime.now; // New result each time.
```

- **결과로 나오는 오브젝트는 원 오브젝트 상태의 모든 것을 노출시키지는 않습니다.** 하나의 필드는 한 오브젝트의 일부분만을 노출시킵니다. 만약 당신의 연산자가 원 오브젝트의 전체 상태를 노출시키는 결과값을 반환한다면, `to___()` 또는 `as___()` 메서드를 사용하는 것이 낫습니다.

만약 위에서 서술한 것들이 당신의 연산자를 서술하고 있다면, getter로 선언하세요. 소수의 요소들만이 이 조건에 부합할 거라고 생각할 수도 있습니다만, 놀랍게도 실제로는 많은 것들이 여기에 부합합니다. 많은 연산자들이 단순히 어떤 상태 그리고 가능한 것에 대한 계산만을 수행하고 있기에 getter가 되어야 할 겁니다.

```dart
// Good 🙆
rectangle.area;
collection.isEmpty;
button.canShow;
dataSet.minimumValue;
```

#### DO 개념적으로 프로퍼티들을 변경하는 연산자들에 대해서는 setter를 사용하세요.

Setter와 메서드 사이에서 어떤 것을 사용할지 결정하는 것은 getter와 메서드 사이에서 어떤 것을 사용할지 결정하는 것과 유사합니다. 두가지 경우에서 모두 연산자는 "필드-같은" 성격을 가집니다.

Setter에서 "필드-같은"의 의미는 다음과 같습니다:

- **연산자가 하나의 인자를 가지고 결과값을 생산하지 않습니다.**
- **연산자가 오브젝트의 어떤 상태를 변경합니다.**
- **연산자가 멱등성을 가집니다.** 호출자가 신경을 쓰고 있는 한 같은 값을 가진 똑같은 setter를 두번 호출할 때, 두번째의 경우에서는 아무 행위도 해서는 안 됩니다. 아마 내부적으로는 캐시 검증 무효화 또는 로깅이 있을 수도 있겠죠. 그건 괜찮습니다. 하지만 호출자의 관점에서 두번째 호출은 아무것도 하지 않는 것으로 나타나게 됩니다.

```dart
// Good 🙆
rectangle.width = 3;
button.visible = false;
```

#### DON'T 상응하는 getter가 없다면 setter를 정의하지 마세요.

사용자들은 getter와 setter을 한 오브젝트의 보이는 프로퍼티로서 생각합니다. 기록 될 수 있으나 볼 수는 없는 "드롭박스" 프로퍼티는 혼란스럽고, 프로퍼티가 작동하는 방식에 대한 사용자들의 직관력을 해칩니다. 예를 들어, getter가 없는 setter는 `=`을 사용해 수정할 수는 있지만 `+=`는 사용할 수 없다는 것을 의미합니다.

이 가이드라인은 setter를 쓰기 위한 수단으로 getter를 사용하라는 것을 의미하지 않습니다. 오브젝트는 일반적으로 필요한 것보다 더 많은 상태를 노출시키지 않아야 합니다. 만약 수정될 수 있으나 같은 방식으로 노출되지는 않는 오브젝트 상태의 부분이 있다면, 메서드를 대신 사용하세요.

#### AVOID 런타임 타입 테스트를 오버로딩 흉내를 내는 용도로 사용하는 것을 피하세요.

API에서는 파라미터들의 다른 타입들에 대한 비슷한 연산자들을 지원하는 것이 흔합니다. 이 유사성을 강조하기 위해, 어떤 언어들은 같은 이름을 가지지만 서로 다른 파라미터 리스트를 가지는 복수의 메서드들을 정의하는 _오버로딩(overloading)_ 을 지원하기도 합니다. 컴파일 시간에, 컴파일러는 어떤 메서드를 호출할지 결정하기 위해 실제 인자 타입을 찾습니다.

다트는 오버로딩을 가지고 있지 않습니다. 대신 하나의 메서드로 정의하고 몸체 안에 인자들의 런타임 타입을 바라보는 `is` 타입 테스트를 사용해 오버로딩과 같은 API를 정의하고, 적절한 작동을 성취할 수 있습니다. 그러나 이런 식으로 가짜 오버로딩을 흉내내는 것은 _컴파일 시간_ 메서드 선택(Method Selection:실제로 실행될 메서드를 결정하는 과정)을 _런타임_ 에 일어나는 선택(choice)으로 바꾸게 됩니다.

만약 호출자가 대개 어떤 타입을 가지고 있는지 알고 있고 어떤 특정한 연산자를 원하고 있는지 알고 있다면, 서로 다른 이름을 가진 별개의 메서드들을 정의해 호출자로 하여금 올바른 연산자를 선택할 수 있게 하는 것이 더 좋습니다.

그러나, 만약 사용자가 알려지지 않은 타입 오브젝트를 갖고 있거나 내부적으로 올바른 연산자를 골라내기 위한 `is`를 사용하는 API를 _원한다면_ 모든 타입을 지원하는 수퍼타입을 가진 파라미터가 있는 하나의 메서드가 합리적일 수 있습니다.

#### AVOID Initializer가 없는 public `late final` 필드를 피하세요.

다른 `final` 필드들과 달리 initializer가 없는 `late final` 필드는 _setter_ 를 _정의합니다._ 만약 그 필드가 public이라면, setter도 public이죠. 이런 형태는 거의 원하는 형태는 아닐 겁니다. 필드는 대개 `late`으로 표시 되어 종종 생성자 몸체 안 그리고 객체의 생명주기 안에서 어떤 시점에 내부적으로 _초기화_ 될 수 있습니다.

사용자가 setter를 호출하기를 원하지 않는 _한_, 다음의 해결책들 중 하나를 선택하는 것이 좋습니다:

- `late`을 사용하지 마세요.
- `final` 필드값을 계산하기 위해 factory 생성자를 사용하세요.
- `late`을 사용해도 `late` 필드 선언부에서 초기화하세요.
- `late`을 사용해도 `late` 필드를 private으로 만들고 그 필드를 가져오기 위한 public getter를 정의하세요.

#### AVOID Nullable인 `Future`,`Stream` 그리고 컬렉션 타입 반환을 피하세요.

API가 컨테이너 타입(Container type: 다양한 종류의 데이터를 묶어서 관리하는 데 사용되는 자료 구조. 리스트, 배열, 스택 등)을 반환하는 경우 데이터의 부재를 가리키는 방법에는 두가지가 있습니다: 비어있는 컨테이너 또는 `null`을 반환할 수 있죠. 사용자들은 일반적으로 "데이터 없음"을 가리키기 위해 비어 있는 컨테이너를 사용할 것을 가정하고 선호합니다. 그런 식으로, 사용자들은 실재 오브젝트를 가질 수 있고, `isEmpty`와 같은 메서드를 호출할 수 있습니다.

당신의 API가 제공할 데이터가 없다는 것을 가리키기 위해, 비어있는 컬렉션, nullable 타입의 non-nullable future 또는 어떤 값도 배출하지 않는 stream을 반환하는 것이 좋습니다.

**예외**: 만약 `null`을 반환하는 것이 _빈 컨테이너를 산출하는 것과 다른 어떤 것을 의미 한다면_, nullable 타입을 사용해도 괜찮습니다.

#### AVOID 단순히 유연한 인터페이스를 허용하기 위한 메서드로부터 `this` 반환을 피하세요.

Method cascades가 chaining method 호출에 대해서는 좀 더 나은 해결책입니다.

```dart
// Good 🙆
var buffer = StringBuffer()
  ..write('one')
  ..write('two')
  ..write('three');
```

```dart
// Bad 🙅
var buffer = StringBuffer()
    .write('one')
    .write('two')
    .write('three');
```

### Types

당신의 프로그램 안에서 타입을 적을 때, 타입을 통해 코드 상 다른 부분들로 흐르는 값들의 종류에 제약을 줍니다. 타입은 두가지 경우에서 나타날 수 있습니다: 선언에 대한 `Type annotations`과 `generic invocations`에 대한 타입 인자들

Type annotations는 "정적 타입"을 생각할 때 일반적으로 떠오르는 것을 의미합니다. 변수, 파라미터, 필드 또는 반환 타입에 타입 주석을 달 수 있습니다. 다음의 예시에서 `bool`과 `String`은 type annotations 입니다. 코드의 정적 서술 구조에 달려 있어 런타임에서는 "실행되지" 않습니다.

```dart
bool isEmpty(String parameter) {
  bool result = parameter.isEmpty;
  return result;
}
```

Generic invocation은 collection literal, generic class 생성자에 대한 호출 또는 generic method의 실행을 뜻합니다. 다음의 예시에서 `num`과 `int`는 generic invocations에 대한 타입 인자들입니다. 이들은 타입임에도 불구하고 구체화 하게 하고 런타임 실행 단계로 넘기는 일급 엔티티(First-Class Entity)로서의 역할을 합니다.

```dart
var lists = <num>[1, 2];
lists.addAll(List<num>.filled(3, 4));
lists.cast<int>();
```

타입 인자들은 type annotations에서도 나타날 수 있다는 점에서 "generic invocation"은 중요합니다.

```dart
List<int> ints = [1, 2];
```

**타입 추론(Type inference)**

다트에서 Type annotations는 선택적입니다. 만약 하나를 빼게 되면, 다트는 가까운 컨텍스트에 기반해 타입을 추론합니다. 때때로 완벽한 타입을 추론하는데 충분한 정보가 없기도 합니다. 그럴 경우, 다트는 때때로 에러를 보고하는데, 하지만 대게 조용하게 놓친 부분들을 `dynamic`으로 채웁니다. 암시적 `dynamic`은 코드가 추론되고 안전한 것처럼 _보이게_ 하지만, 실제로는 타입 확인을 완전히 무효화하는 것입니다. 아래의 규칙들은 추론이 실패했을 때, 타입을 요구해 그런 무효화를 피하는 방법을 제시합니다.

다트가 타입 추론과 `dynamic` 타입 두개 모두를 가지고 있다는 사실은 코드에 "타입이 없다"는 것이 무슨 의미를 가지는지에 대한 혼란으로 이끕니다. 코드에 타입이 동적으로 지정 되었다는 뜻일까요? 아니면 타입을 _작성하지_ 않았다는 의미일까요? 그러한 혼란을 피하기 위해, "타입이 없다"라고 말하는 것을 피하고 대신에 다음의 전문용어를 사용하세요:

- 만약 코드가 _타입 주석 처리된 상태(type annotated)_ 라면, 타입은 명시적으로 코드에 쓰여진 겁니다.
- 만약 코드가 _타입 추론된 상태(inferred)_ 고, 다른 type annotation이 작성되지 않았다면, 다트는 성공적으로 스스로의 타입을 확인합니다. 가이드라인이 추론 된 것을 고려하지 않는다면 실패할 수도 있겠지만요.
- 만약 코드가 _동적인 타입 상태(dynamic)_ 라면, 코드의 정적 타입은 특별한 `dynamic` 타입입니다. 코드는 명시적으로 `dynamic`으로 주석 처리 되거나 추론된 타입이 사용될 수 있습니다.

달리 말하면, 어떤 코드가 주석처리가 되던지 혹은 추론되던지 여부는 `dynamic` 타입 혹은 다른 타입을 가지는 것과는 직교 관계(서로 독립적 관계)를 형성합니다.

추론은 명백하거나 재미없는 타입을 작성하고 읽는 수고를 줄여주는 강력한 도구입니다. 이 도구는 코드의 행위에 초점을 맞춰 독자의 주목을 유지시킵니다. 명시적 타입 또한 풍부하고, 유지 가능한 코드의 중요한 부분입니다. 명시적 타입은 한 API의 정적 형상을 정의하고 문서에 경계를 형성해 프로그램의 서로 다른 부분들에 도달하도록 허용된 각자의 값을 강제합니다.

물론, 추론은 마법이 아닙니다. 때때로 추론은 타입을 물려 받거나 선택하지만 당신이 원하는 타입이 아닐지도 모릅니다. 흔한 경우는 변수에 다른 타입의 값을 할당하려는 의도를 가진 채, 변수의 initializer으로부터 과도하게 정확한 타입을 추론하게 하는 겁니다. 그러한 경우에는 타입을 명시적으로 작성해야 합니다.

이 가이드라인들은 여기서 간결성과 통제, 유연성과 안전성 사이에서 우리가 찾아낸 최고의 균형을 커버하고 있습니다. 요약하자면 다음과 같습니다:

- 추론이 충분한 컨텍스트를 가지고 있지 않을 때 annotate을 사용하세요. `dynamic` 타입을 원한다고 해도 annotate을 사용하세요.
- 필요하다고 판단하지 않은 이상 locals와 generic invocations에 annotate을 사용하지 마세요.

```dart
// Bad 🙅
// 불필요한 type annotation
String userName = "Alice" // 컴파일러는 name이 String 타입임을 알 수 있습니다.
```

```dart
// Good 🙆
const defaultUserName = "Alice";
```

- Initializer가 타입을 확실하게 만들지 않는 한, 최상위 변수들과 필드들에 annotate 하는 편이 좋습니다.

#### DO Initializers 없이 변수에 type annotate 하세요.

최상위, 지역, 정적 필드 또는 객체 필드와 같은 변수의 타입은 종종 initializer에 의해 추론될 수 있습니다. 그러나 만약 initializer가 없다면 추론은 실패합니다.

```dart
// Good 🙆
List<AstNode> parameters;
if (node is Constructor) {
  parameters = node.signature;
} else if (node is Method) {
  parameters = node.parameters;
}
```

```dart
// Bad 🙅
var parameters;
if (node is Constructor) {
  parameters = node.signature;
} else if (node is Method) {
  parameters = node.parameters;
}
```

#### DO 만약 타입이 명확하지 않다면 필드와 최상위 변수에 type annotate 하세요.

Type annotation은 한 라이브러리가 어떻게 사용되어야 하는지에 대한 중요한 문서입니다. Type annotation은 한 프로그램의 구역 사이에서 경계를 형성하여 타입 에러 소스를 격리합니다. 다음을 생각해봅시다:

```dart
// Bad 🙅
install(id, destination) => ...
```

여기서 `id`는 무엇인지 불확실합니다. String일까요? `destination`은 뭘까요? String? 혹은 File 오브젝트? 이 메서드는 동기적일까요? 비동기적일까요? 다음이 좀 더 명확합니다:

```dart
// Good 🙆
Future<bool> install(PackageId id, String destination) => ...
```

그럼에도 불구하고 몇몇 경우에서 타입을 작성하는 것은 명백히 무의미하게 보이는 경우도 있죠:

```dart
// Good 🙆
const screenWidth = 640; // Inferred as int.
```

"명백히"라는 것은 정확히 정의될 수 없지만, 다음은 모두 좋은 후보군들입니다:

- Literals.
- Constructor invocations.
- 명시적으로 타입이 지정된 다른 상수들에 대한 참조.
- Strings와 numbers에 대한 간단한 표현들.
- 독자가 친근하게 느낄 것으로 기대되는 `int.parse()`,`Future.wait()` 등의 factory 메서드들.

뭐가 되었든 initializer 표현이 충분히 명확하다고 생각한다면, annotation을 제외할 수도 있습니다. 하지만 type annotating이 코드를 명확히 하는 데 도움이 된다면, 추가해도 좋습니다.

의심이 될 때는 type annotaion을 추가하세요. 타입이 명확할 때도, 여전히 명시적으로 annotate 하고 싶을 때가 있을 겁니다. 만약 다른 라이브러리들의 값 또는 선언에 의존해 타입을 추론한다면, 당신의 선언에 type annotate을 해, 다른 라이브러리의 변경사항이 당신의 인지없이 당신의 API의 타입 또한 조용히 변경하지 않도록 합니다.

이 규칙은 public과 private 선언 둘 모두에 적용됩니다. API에 대한 type annotations를 _코드 사용자_ 를 돕기 위한 것으로, private 요소들에 대한 타입은 _유지보수 하는 사람_ 을 돕기 위한 것으로 생각하세요.

#### DON'T 초기화된 지역 변수에 불필요하게 type annotate 하지 마세요.

지역 변수들 중, 특히 크기가 작은 함수들로 쓰여진 현대 코드는 아주 작은 범위를 가지고 있습니다. 타입을 빼는 것은 독자의 주의를, 더 중요하게 여겨지는 변수의 _이름_ 과 초기화된 값에 집중시킵니다.

```dart
// Good 🙆
List<List<Ingredient>> possibleDesserts(Set<Ingredient> pantry) {
  var desserts = <List<Ingredient>>[];
  for (final recipe in cookbook) {
    if (pantry.containsAll(recipe)) {
      desserts.add(recipe);
    }
  }

  return desserts;
}
```

```dart
// Bad 🙅
List<List<Ingredient>> possibleDesserts(Set<Ingredient> pantry) {
  List<List<Ingredient>> desserts = <List<Ingredient>>[];
  for (final List<Ingredient> recipe in cookbook) {
    if (pantry.containsAll(recipe)) {
      desserts.add(recipe);
    }
  }

  return desserts;
}
```

때때로 추론된 타입은 변수에 기대하는 타입과 다를 겁니다. 예를 들어, 추후에 다른 타입의 값을 할당하려는 의도를 가지고 있을 수 있습니다. 그러한 경우에는, 당신이 원하는 타입을 변수에 annotate 하세요.

```dart
// Good 🙆
Widget build(BuildContext context) {
  Widget result = Text('You won!'); // 의도를 가진 Widget annotation
  if (applyPadding) {
    result = Padding(padding: EdgeInsets.all(8.0), child: result);
  }
  return result;
}
```

#### DO 함수 선언들에 대해 반환 타입을 annotate 하세요.

한 함수의 파라미터 리스트는 바깥 세계로의 경계선을 결정합니다. 파라미터 타입을 annotating 하는 것은 그 경계를 잘 정의되게끔 만들어줍니다. 비록 default 파라미터값이 변수 initializer처럼 보이겠지만 다트는 default 값으로부터 선택적 파라미터 타입을 추론하지 않습니다.

```dart
// Good 🙆
void sayRepeatedly(String message, {int count = 2}) { // count에 type annotating
  for (var i = 0; i < count; i++) {
    print(message);
  }
}
```

```dart
// Bad 🙅
void sayRepeatedly(message, {count = 2}) {
  for (var i = 0; i < count; i++) {
    print(message);
  }
}
```

**Exception:** 함수 표현들과 양식을 초기화하는 것은 서로 다른 type annotation 컨벤션을 가지고 있습니다.

#### DON'T 함수 표현들에 대해 추론된 파라미터 타입들을 annotate 하지 마세요.

익명의 함수들은 거의 항상 즉시 어떤 타입의 콜백을 가지는 메서드로 넘겨집니다. 함수 표현이 타입 처리된 컨텍스트 안에서 생성되었을 때, 다트는 기대되는 타입에 기반해 함수 파라미터 타입을 추론하려고 노력합니다. 예를 들어, 함수 표현을 `Iterable.map()`에 넘길 때, 당신의 함수 파라미터 타입은 `map()`이 기대하는 콜백 타입에 기반해 추론됩니다:

```dart
// Good 🙆
var names = people.map((person) => person.name);
```

```dart
// Bad 🙅
var names = people.map((Person person) => person.name);
```

만약 다트가 한 함수 표현 안에서 파라미터로 원하는 타입을 추론할 수 있다면, annotate 하지 마세요. 극히 희소한 경우, 인근 컨텍스트가 하나 또는 그 이상의 함수 파라미터에 대해 타입을 제공하는데 충분히 정확하지 않을 때도 있습니다. 그러한 경우에 annotate할 필요가 있습니다. (만약 함수가 즉시 사용되는 게 아니라면, 명명된 선언(람다가 아닌 일반 함수)을 사용하는 것이 더 좋습니다.)

#### DON'T 양식 초기화(initializing formals)에 type annotate 하지 마세요.

만약 생산자 파라미터가 한 필드를 초기화하는데 `this.`를 사용한다면 또는 super 파라미터를 포워딩하기 위해 `super.`를 사용한다면, 그 파라미터의 타입은 초기화하려는 필드 또는 super-생산자 파라미터와 같은 타입을 갖도록 각각 추론되어야 합니다.

```dart
// Good 🙆
class Point {
  double x, y;
  Point(this.x, this.y);
}

class MyWidget extends StatelessWidget {
  MyWidget({super.key});
}
```

```dart
// Bad 🙅
class Point {
  double x, y;
  Point(double this.x, double this.y);
}

class MyWidget extends StatelessWidget {
  MyWidget({Key? super.key});
}
```

#### DO 추론되지 않은 generic invocations에 대해 타입 인자를 작성하세요.

다트는 generic invocations에서 타입 인자를 추론하는 것에 대해 꽤 똑똑한 성능을 가지고 있습니다. 다트는 표현이 발생하는 곳에서 기대되는 타입과 invocation에 전달되는 값들의 타입을 바라봅니다. 그러나 때때로 하나의 타입 인자를 완벽히 결정하는 데 충분하지 않을 수 있습니다. 그러한 경우에는 모든 타입 인자 리스트를 명시적으로 작성합니다.

```dart
// Good 🙆
var playerScores = <String, int>{};
final events = StreamController<Event>();
```

```dart
// Bad 🙅
var playerScores = {};
final events = StreamController();
```

때때로 invocation은 initializer로써 변수 선언에서 발생하기도 합니다. 만약 변수가 로컬이 _아니라면_, invocation 그 자체에 대한 타입 인자 리스트를 작성하는 것 대신에 선언에 type annotation을 넣을 수도 있습니다:

```dart
// Good 🙆
class Downloader {
  final Completer<String> response = Completer();
}
```

```dart
// Bad 🙅
class Downloader {
  final response = Completer();
}
```

#### DON'T 추론된 generic invocations에 대해 타입 인자를 작성하지 마세요.

```dart
// Good 🙆
class Downloader {
  final Completer<String> response = Completer();
}
```

```dart
// Bad 🙅
class Downloader {
  final Completer<String> response = Completer<String>();
}
```

필드 상 type annotation은 initializer에서 생성자 호출의 타입 인자를 추론하기 위한 인근 컨텍스트를 제공합니다.

```dart
// Good 🙆
var items = Future.value([1, 2, 3]);
```

```dart
// Bad 🙅
var items = Future<List<int>>.value(<int>[1, 2, 3]);
```

여기서, 컬렉션의 타입들과 객체는 그 컬렉션의 요소들과 인자들로부터 상향식 추론이 될 수 있습니다.

#### AVOID 불완전한 generic 타입 작성을 피하세요.

Type annotation 또는 타입 인자를 작성하는 것의 목표는 완벽한 타입을 분명하게 정의하기 위한 것에 있습니다. 그러나 만약 당신이 generic type의 이름을 작성하지만 타입 인자를 뺀다면, 당신은 완전히 구체화된 타입을 작성했다고 할 수 없습니다. 자바에서, 이를 "raw types"라고 부릅니다. 예를 들면 다음과 같습니다:

```dart
// Bad 🙅
List numbers = [1, 2, 3];
var completer = Completer<Map>();
```

여기서 `numbers`는 type annotation을 가지고 있지만 generic `List`에 대해 타입 인자를 제공하지는 않습니다. 마찬가지로, `Completer`의 `Map` 타입 인자는 완전히 명확하지는 않습니다. 이러한 경우에 다트는 인근 컨텍스트에 기반해 잔여 타입을 "채우려는" 노력을 하지는 않습니다. 대신에 `dynamic`(또는 만약 클래스가 가지고 있다면 가능성이 큰 것)으로 잃은 타입 인자들을 조용히 채웁니다. 이것을 원하는 사람은 극히 적겠죠.

대신, 만약 어떤 invocation 안에 타입 인자로써 또는 type annotation 안에 generic type을 작성한다면, 완전한 타입을 작성할 것을 확실히 하세요:

```dart
// Good 🙆
List<num> numbers = [1, 2, 3];
var completer = Completer<Map<String, int>>();
```

#### DO 추론이 실패하도록 두는 것 대신에 `dynamic`과 함께 annotate 하세요.

추론이 타입을 채워 넣지 않을 때, 대개 `dynamic`이 기본입니다. 만약 `dynamic`이 원하는 타입이라면, 기술적으로 가장 간단한 방식으로 얻을 수 있습니다. 그러나 가장 _깔끔한_ 방식은 아닙니다. 가볍게 코드를 읽는 독자들은 당신이 의도적으로 `dynamic`을 사용했는지, 다른 타입을 채워 넣는 추론을 기대한 것인지 또는 단순히 annotation 작성을 잊은 것인지 알 수 있는 방법이 없기 때문에 annotation이 사라졌다고 생각하게 됩니다.

`dynamic` 타입이 당신이 원하는 타입일 때, 명시적으로 의도를 드러내어 이 코드는 정적 안정성이 덜하다는 강조 표시를 하세요.

```dart
// Good 🙆
dynamic mergeJson(dynamic original, dynamic changes) => ...
```

```dart
// Bad 🙅
mergeJson(original, changes) => ...
```

다트가 _성공적으로_ `dynamic`을 추론할 때, 타입을 빼는 것은 괜찮습니다.

```dart
// Good 🙆
Map<String, dynamic> readJson() => ...

void printUsers() {
  var json = readJson();
  var users = json['users'];
  print(users);
}
```

여기서, 다트는 `json`에 대해 `Map<String, dynamic>`으로 `users`에 대해 `dynamic`을 추론합니다. `users`에 type annotation을 적용하지 않고 두어도 괜찮습니다. 그 차이는 조금 미묘합니다. 어떤 다른 곳에서 `dynamic`으로 type annotation 된 곳에서 `dynamic`을 전파해 추론을 허용하는 것은 괜찮습니다. 그러나 당신의 코드가 구체화하지 않은 곳에서 `dynamic` type annotation을 주입하는 것은 지양하는 것이 좋습니다.

> **⚠️Note**
> 다트의 강력한 타입 시스템과 타입 추론 능력으로 사용자들은 다트가 추론된 정적 타입 언어(자동으로 타입이 지정 되는 언어)로 작동하길 기대합니다. 하지만 그렇게 됐을 경우, 모든 정적 타입의 성능과 안정성을 잃어버리게 될 수 있다는 점을 유의하세요.

**Exception:** 사용되지 않는 파라미터들에 대한 type annotations는 (`_`) 제외 될 수 있습니다.

#### PREFER 함수 type annotations에서 입력값과 출력값이 대한 타입을 명시적으로 표기하기(signature)를 추천합니다.

어떤 반환 타입 또는 파라미터 시그니처가 없는 `Function` 식별자는 특별한 `Function` 타입을 참조합니다. 이 타입은 `dynamic`을 사용하는 것보다 미미하게 더 유용합니다. 만약 이 타입을 annotation 하는 데 사용할 거라면, 함수의 파라미터와 반환값을 포함하는 완전한 함수 타입을 사용할 것을 추천합니다.

```dart
// Good 🙆
bool isValid(String value, bool Function(String) test) => ... // bool Function(String): 반환값-bool, 파라미터-string을 가진 함수
```

```dart
// Bad 🙅
bool isValid(String value, Function test) => ...
```

**Exception:** 때때로, 당신은 서로 다른 복수의 함수 타입들 표현할 때 대표로 사용할 수 있는 하나의 타입을 원할 수도 있습니다. 예를 들어, 파라미터를 하나 가진 한 함수 또는 두개 가진 한 함수를 가져온다고 가정해 보죠. 우리는 union 타입들을 가지고 있지 않아 타입을 정확히 할 수 있는 방법이 없으므로 일반적으로 `dynamic.Function`을 사용하는 편이 좀 더 유용한 방법일 겁니다:

```dart
// Good 🙆
void handleError(void Function() operation, Function errorHandler) {
  try {
    operation();
  } catch (err, stack) {
    if (errorHandler is Function(Object)) {
      errorHandler(err);
    } else if (errorHandler is Function(Object, StackTrace)) {
      errorHandler(err, stack);
    } else {
      throw ArgumentError('errorHandler has wrong signature.');
    }
  }
}
```

#### DON'T setter에 대해 반환 타입을 명시하지 마세요.

다트에서 setter는 항상 `void`를 반환합니다. `void`를 작성하는 것은 무의미합니다.

```dart
// Bad 🙅
void set foo(Foo value) { ... }
```

```dart
// Good 🙆
set foo(Foo value) { ... }
```

#### DON'T 레거시 `typedef` 문법을 사용하지 마세요.

다트는 하나의 함수 타입에 대해 명명된 typedef를 정의할 수 있는 두가지 표기법을 가지고 있습니다. 본래의 표기법은 다음과 같습니다:

```dart
// Bad 🙅
typedef int Comparison<T>(T a, T b);
```

이 문법은 몇 가지 문제점을 가집니다:

- _generic_ 함수 타입에 이름을 할당할 수 있는 방법이 없습니다. 위 문제에서 typedef는 그 자체로 generic 입니다. 만약 당신의 코드에서 타입 인자 없이 `Comparison`를 참조하면, `int Function<T>(T, T)`가 아니라 당신은 암시적으로 `int Function(dynamic, dynamic)` 함수 타입을 가지게 됩니다. 실제 환경에서는 문제가 발생하지는 않는 것처럼 보이지만, 특정 특이 케이스에서는 문제가 됩니다.
- 하나의 파라미터를 가졌을 때, 단 하나의 식별자는 _타입_ 이 아닌 파라미터의 _이름_ 으로 해석됩니다. 다음이 주어졌다고 봅시다:

```dart
// Bad 🙅
typedef bool TestNumber(num);
```

대부분의 사용자들은 이것을 `num`을 받아 `bool`을 반환하는 함수 타입일 것이라고 생각합니다. 실제로는 이 함수 타입은 `dynamic`인 _어떤_ 오브젝트를 받아 `bool`을 반환합니다. 파라미터의 _이름_ (typedef의 문서를 제외하고 어떤 용처에도 사용되지 않는)은 "num"입니다. 이것은 다트에서 오랫동안 유지됐던 에러 소스입니다.

새로운 문법은 다음과 같습니다:

```dart
// Good 🙆
typedef Comparison<T> = int Function(T, T);
```

만약 파라미터 이름을 포함하고 싶다면, 그것도 가능합니다:

```dart
// Good 🙆
typedef Comparison<T> = int Function(T a, T b);
```

새로운 문법은 오래된 문법의 어떤 형태도 표현할 수 있고 그 이상도 가능합니다. 그리고 하나의 식별자가 파라미터의 타입 대신 이름으로 여겨지는 것에 대한 에러 경향성도 제거가 가능하죠. typedef에서 `=` 이후에 같은 함수 타입 문법은 type annotation이 등장할 수 있는 어느 곳에서나 허용됩니다. 이를 통해 우리는 하나의 일관된 방식으로 프로그램 상 어느 곳에서나 함수 타입을 작성할 수 있게 됩니다.

기존의 코드를 망가뜨리지 않게 오래된 typedef 문법을 여전히 지원하고는 있으나 더이상 사용되지 않는 것으로 처리 됐음(deprecated)을 유의하세요.

#### PREFER typedefs가 아닌 한 줄 함수 타입 사용을 추천합니다.

다트에서 필드, 변수 또는 generic 타입 인자에 대해 함수 타입을 사용하기를 원한다면, 함수 타입의 typedef를 정의할 수 있습니다. 그러나 다트는 한줄 함수 타입 문법을 지원하고 있어서 type annotation이 허용되는 곳이라면 어느 곳에서든 사용이 가능합니다:

```dart
// Good 🙆
class FilteredObservable {
  final bool Function(Event) _predicate;
  final List<void Function(Event)> _observers;

  FilteredObservable(this._predicate, this._observers);

  void Function(Event)? notify(Event event) {
    if (!_predicate(event)) return null;

    void Function(Event)? last;
    for (final observer in _observers) {
      observer(event);
      last = observer;
    }

    return last;
  }
}
```

특별히 길거나 빈번하게 사용되는 함수 타입은 typedef를 선언해서 사용하는 것도 괜찮은 방법입니다. 그러나 대부분의 경우, 사용자들은 타입이 사용되는 곳에서 무슨 함수 타입이 적절한지 알고 싶어하므로, 여기서 함수 타입 문법이 명료함을 줄 수 있습니다.

#### PREFER 파라미터에 대해 함수 타입 문법(function type syntax) 사용을 사용할 것을 추천합니다.

다트는 타입이 함수인 파라미터를 정의할 때 사용할 수 있는 특별한 문법을 가지고 있습니다. C와 같이 함수의 반환 타입과 파라미터 시그니처로 파라미터의 이름을 감쌀 수 있습니다:

```dart
Iterable<T> where(bool predicate(T element)) => ...
```

다트가 함수 타입 문법을 더하기 전에, typedef를 사용하지 않고서는 하나의 함수에 하나의 파라미터를 주는 것이 유일한 방법이었습니다. 이제는 다트에서 함수 타입들에 대해 일반적인 표기법을 가지고 있어 함수-타입을 가진 파라미터들에서 사용할 수 있게 됐습니다:

```dart
// Good 🙆
Iterable<T> where(bool Function(T) predicate) => ...
```

새로운 문법은 조금 장황해보이지만, 새로운 문법을 사용해야만 하는 곳들에서 일관성을 유지시킵니다.

#### AVOID 정적 확인(static checking)을 무효화하는 것을 원하지 않는 이상, `dynamic` 사용을 피하세요.

`log()` 메서드는 어떤 오브젝트라도 받아 `toString()`을 호출할 수 있습니다. 다트에서 `Object?` 그리고 `dynamic` 타입은 어느 값이라도 허용합니다. 그러나 이 타입들이 전달하는 것들은 다릅니다. 만약 당신이 간단하게 모든 오브젝트들을 허용할 수 있다고 명시하려면, `Object?`를 사용하면 됩니다. 만약 `null`을 제외한 모든 오브젝트들을 허용하기를 원한다면, `Object`를 사용하면 됩니다.

`dynamic` 타입은 모든 오브젝트들을 받아들일 뿐만 아니라 모든 _연산자_ 들을 허용합니다. `dynamic` 타입의 값에 접근하는 요소는 어떤 것이라도 컴파일 시간에는 허용되지만, 런타임에서 예외를 발생시켜 작동에 있어서는 실패할 겁니다. 이런 위험성을 감수하면서, 유연한 dynamic 처리를 원한다면 `dynamic`은 사용하기 적절한 타입입니다.

그렇지 않다면, `Object?` 또는 `Object`를 사용할 것을 추천합니다. 당신이 접근하기 전에 접근하고자 하는 요소를 해당 값의 런타임 타입이 지원하도록 `is` 확인과 타입 승급(type promotion)에 의존하세요.

```dart
// Good 🙆
/// Returns a Boolean representation for [arg], which must
/// be a String or bool.
bool convertToBool(Object arg) {
  if (arg is bool) return arg;
  if (arg is String) return arg.toLowerCase() == 'true';
  throw ArgumentError('Cannot convert $arg to a bool.');
}
```

이 규칙에서 주로 나타나는 예외는 기존의 `dynamic`을 사용하는 API들과 작동할 때 입니다. 특히 generic 타입 안에서 발생하죠. 예를 들어, `Map<String, dynamic>` 타입을 가진 JSON 오브젝트들과 같은 타입을 허용해야 하는 당신의 코드가 있다고 해봅시다. 이러한 API들 중 하나로부터 값을 사용할 때는 요소들에 접근하기 전에 좀 더 정확한 타입으로 캐스트 하는 것이 좋은 아이디어입니다.

#### DO 값을 생산하지 않는 비동기 요소들의 반환 타입은 `Future<void>`을 사용하세요.

값을 반환하지 않는 동기적인 함수가 있을 때, `void`를 반환 타입으로 사용할 겁니다. await을 해야 하는 비동기 함수의 경우에는 `Future<void>`를 사용합니다.

#### AVOID 반환 타입으로 `FutureOr<T>` 사용을 피하세요.

만약 `FutureOr<int>`를 반환하면 사용자들은 `int`를 받을지 `Future<int>`를 받을지 확인부터 해야 합니다. 이런 경우, `Future<int>`만 반환하게 하는 게 오히려 더 깔끔합니다. 사용자들이 이해하기 쉽도록 하려면 항상 비동기를 반환하던지 동기를 반환하던지 선택하는 게 좋습니다.

```dart
// Good 🙆
Future<int> triple(FutureOr<int> value) async => (await value) * 3;
```

```dart
// Bad 🙅
FutureOr<int> triple(FutureOr<int> value) {
  if (value is int) return value * 3;
  return value.then((v) => v * 3);
}
```

좀 더 정확하게 이 가이드라인을 공식화하자면, `FutureOr<T>` 를 _반변(반대로 변함) 위치에서만_ 사용하도록 합니다. 파라미터들은 반변이고 반환 타입들은 공변(함께 변함)입니다. 중첩 함수들에서 이는 뒤집어집니다-만약 타입이 그 자체로 함수인 파라미터를 가지고 있으면, 콜백의 반환 타입은 반변 위치가 되고 콜백의 파라미터는 공변이 됩니다. 즉, _콜백_ 은 `FutureOr<T>` 타입으로 반환해도 괜찮다는 의미입니다:

```dart
// Good 🙆
Stream<S> asyncMap<T, S>(
    Iterable<T> iterable, FutureOr<S> Function(T) callback) async* {
  for (final element in iterable) {
    yield await callback(element);
  }
}
```

### Parameters

다트에서 선택적 파라미터들은 위치로 구분할 수도 또는 이름으로 구분할 수도 있으나 두가지를 동시에 하는 것은 안 됩니다.

#### AVOID 위치에 따른(positional) boolean 파라미터

다른 타입들과 달리 booleans는 대개 리터럴 폼에서 사용됩니다. numbers와 같은 값들은 대개 명명된 상수들로 감싸져 있지만, 우리는 일반적으로 `true`와 `false`와 같은 값은 직접적으로 바로 넘기고는 합니다. 이는 호출 지점들을 읽기 어렵게 만들고 boolean이 뜻하는 바를 모호하게 만듭니다:

```dart
// Bad 🙅
new Task(true);
new Task(false);
new ListBox(false, true, true);
new Button(false);
```

대신 명명된 인자, 명명된 생성자 또는 명명된 상수들을 사용해 호출하는 것이 무엇을 하고 있는지 명확하게 할 것을 추천합니다.

```dart
Task.oneShot();
Task.repeating();
ListBox(scroll: true, showScrollbars: true);
Button(ButtonState.enabled);
```

이 가이드라인은 setters에 대해서는 적용되지 않는다는 점에 주의하세요. Setter는 이름으로 값이 무엇을 의미하는지 명확하게 할 수 있습니다.

```dart
// Good 🙆
listBox.canScroll = true;
button.isEnabled = false;
```

#### AVOID 위치에 따른 파라미터에서 사용자가 빼기를 원할 수 있는 파라미터를 더 앞에 위치시키는 것을 피하세요.

선택적 위치에 따른 파라미터(Optional positional parameters)들은 논리적 진행을 거쳐야 합니다. 그리고 파라미터의 순서 상 좀 더 사용 빈도수가 높은 파라미터는 선순위를 가져야 합니다. 사용자들이 앞선 위치에 따른 파라미터를 제외 시키기 위해 "구멍(예:파라미터1,(구멍),파라미터2)"을 놓고 그 다음에 파라미터를 넣는 행위를 하게 해서는 안 됩니다. 이런 단점들을 개선하기 위해 명명된 인자(named argument)를 사용하는 것이 더 낫습니다.

```dart
// Good 🙆
String.fromCharCodes(Iterable<int> charCodes, [int start = 0, int? end]);

DateTime(int year,
    [int month = 1,
    int day = 1,
    int hour = 0,
    int minute = 0,
    int second = 0,
    int millisecond = 0,
    int microsecond = 0]);

Duration(
    {int days = 0,
    int hours = 0,
    int minutes = 0,
    int seconds = 0,
    int milliseconds = 0,
    int microseconds = 0});
```

#### AVOID "없는 인자"를 표시하는 특별한 값을 필수적으로 넣도록 하는 파라미터를 피하세요.

```dart
// Good 🙆
var rest = string.substring(start);
```

```dart
// Bad 🙅
var rest = string.substring(start, null);
```

#### DO 파라미터에서 범위를 받는 것에 시작을 포함하고 끝을 배제하세요.

```dart
// Good 🙆
[0, 1, 2, 3].sublist(1, 3) // [1, 2]
'abcd'.substring(1, 3) // 'bc'
```

### Equality

한 클래스에서 사용자화 된 동등성(custom equality)을 실행하는 것은 까다로울 수 있습니다.

#### DO `==`를 덮어쓴다면 `hashCode`를 덮어쓰세요.

기본 해시 코드 실행은 하나의 _독자성_ 해시를 제공합니다-만약 두개의 오브젝트가 완전히 같은 오브젝트라면 일반적으로 같은 해시 코드를 가집니다. 마찬가지로 `==`의 기본 작동은 독자성을 가집니다.

만약 `==`을 덮어쓰게 된다면, 당신의 클래스에 의해 다른 오브젝트들이 "같은" 것으로 고려된다는 점을 암시합니다. **같은 두개의 오브젝트들은 같은 해시 코드를 가져야만 합니다.** 그렇지 않으면, maps와 다른 hash-based collections는 두개의 오브젝트가 서로 동등하다는 인지에 실패하게 됩니다.

#### DO 당신의 `==` 연산자가 수학적 동등성 규칙을 지키도록 하세요.

동등성 관계는 다음과 같아야 합니다:

- 재귀적인(Reflexive): `a == a`는 항상 `true`를 반환해야 합니다.
- 대칭적인(Symmetric): `a == b`는 `b == a`와 같은 것을 반환해야 합니다.
- 전이적인(Transitive): `a == b`와 `b == c`가 `true`를 반환하면 `a == c`도 그래야 합니다.

`==`를 사용하는 코드와 사용자들은 이 규칙들이 지켜진다고 가정합니다. 만약 당신의 클래스가 이러한 규칙들을 지키지 않는다면, `==`은 당신이 표현하려고 하는 연산자에 대한 올바른 이름이 아닙니다.

#### AVOID 변할 수 있는 클래스들에 대해서는 사용자화된 동등성(custom equality) 정의를 피하세요.

`==`를 정의할 때, `hashCode`도 같이 정의해야 합니다. 두가지 모두 클래스 오브젝트의 필드들을 고려해야 합니다. 필드들이 _변한다는 것은_ 클래스 오브젝트의 해시 코드도 변할 수 있음을 암시합니다.

대부분의 해시에 기초하는 컬렉션들은 한 오브젝트의 해시 코드가 보통 같을 거라고 예측하지만, 영원할 거라고 가정하지는 않습니다. 따라서 어떤 경우, 필드값이 변경 돼 컬렉션의 해시 코드가 변경 되었을 때는 컬렉션은 예측과 다르게 작동할 수 있습니다.

#### DON'T `==` 연산자로의 파라미터가 nullable이 되지 않도록 하세요.

```dart
// Good 🙆
class Person {
  final String name;

  // ···

  bool operator ==(Object other) => other is Person && name == other.name;
}
```

```dart
// Bad 🙅
class Person {
  final String name;

  // ···

  bool operator ==(Object? other) =>
      other != null && other is Person && name == other.name;
}
```

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
