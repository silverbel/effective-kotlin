# 아이템 31. 문서로 규약을 정의하라.md

아래 코드를 살펴보자.
```kt
fun Context.showMessage(
    message: String,
    length: MessageLength = MessageLength.LONG
) {
    val toastLength = when(length){
        SHORT -> Length.SHORT
        LONG -> Length.LONG
    }
    Toast.makeText(this, message, toastLength).show()
}

enum class MessageLength { SHORT, LONG }
```
위 함수는 메시지 출력 방법을 자유롭게 변경할 수 있게 추출한 코드다.
문서화가 되어 있지 않기 때문에 다른 개발자는 코드를 읽고, 토스트를 출력할 거라고 생각할 수 있다.
하지만 이 함수는 다른 타입으로도 메세지를 출력할 수 있게 하고자 붙인 이름이다.
따라서 이 함수가 무엇을 하는지 명확하게 설명하고 싶다면, KDoc 주석을 붙여 주는 것이 좋다.

## KDoc
- `/**`으로 시작해 `*/`으로 끝난다.
- 첫 번째 부분은 요소에 대한 요약 설명
- 두 번째 부분은 상세 설명
- 이어지는 줄은 모두 태그로 시작, 추가적인 설명을 위해 태그 사용

주요 태그
- @param 함수 파라미터 또는 클래스, 함수 타입 파라미터를 문서화
- @return 함수의 리턴 값을 문서화
- @throws 메서드 내부에서 발생할 수 있는 예외에 대한 문서화
- @see 특정한 클래스 또는 메서드에 대한 링크 추가. 문서 내용 중간에서 링크가 필요하다면 `[]`로 대체 가능

## 정리
- 요소, 특히 외부 API를 구현할 때는 규약을 장 정의
- 이런 규약은 이름, 문서, 주석, 타입을 통해 구현 가능
- 규약은 사용자가 쉽게 이해할 수 있고 예측 가능하게 해줌
