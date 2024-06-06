# 아이템 18. 코딩 컨벤션을 지켜라

코틀린은 굉장히 잘 정리된 [코딩 컨벤션](https://kotlinlang.org/docs/coding-conventions.html)을 가지고 있다.
어떤 개발 언어든 코딩 컨벤션은 최대한 지켜주는 것이 좋다.
코딩 컨벤션을 지켜야하는 이유.
- 어떤 프로젝트를 접해도 쉽게 이해할 수 있따.
- 다른 개발자도 코드의 작동 방식을 쉽게 추측할 수 있다.
- 코드를 병합하고, 한 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉽다.

## 코딩 컨벤션을 지키는 데 도움이 되는 것..
- IntelliJ 포매터
  - Setting ➡ Editor ➡ Code Style ➡ Kotlin 클릭\
우측 상단 Set from... ➡ 원하는 스타일 가이드를 지정\
Style issues File ➡ [File is not formatted according to project settings] 체크

- 정적 분석 툴이나 IntelliJ 플러그인
  - ktlint - 공식 가이드에 기반한 코드 스타일과 컨벤션을 검사한다.
  - detekt - 다양한 옵션을 제공하며 컨벤션과 함께 코드 품질을 검사한다.
