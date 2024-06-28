# 아이템 27. 변화로부터 코드를 보호하려면 추상화를 사용하라

> 물 위를 걷는 것과 명세서로 소프트웨어를 개발하는 것은 쉽다. 둘 다 동결되어 있다면.. - Edward V. Berard

추상화를 통해 변화로부터 코드를 보호하는 행위가 어떤 자유를 가져오는지 살펴보자.

## 상수
리터럴은 아무것도 설명하지 않는다. 따라서 코드에 반복적으로 등장할 때 문제가 된다.
리터럴을 상수 프로퍼티로 변경하면 해당 값에 이름을 붙일 수 있으며, 값을 변경할 때 훨씬 쉽게 변경가능하다.

```kotlin
fun isPasswordValid(text: String): Boolean{
  if(text.length < 7) return false
  //...
}

const val MIN_PASSWORD_LENGTH = 7
fun isPasswordValid(text: String): Boolean{
  if(text.length < MIN_PASSWORD_LENGTH) return false
  //...
}
```
함수 내부 로직을 이해못해도 상수의 값만 변경하면 비밀번호 최소 길이를 변경할 수 있다.

상수를 추출하면
- 이름을 붙일 수 있음.
- 나중에 해당 값을 쉽게 변경할 수 있음.

## 함수
toast를 발생하는 코드같이 많이 사용되는 알고리즘은 간단한 확장 함수로 만들어 사용할 수 있다.

```kotlin
fun Context.showMessage(
    message: String,
    duration: MessageLength = MessageLEngth.LONG
) {
    val toastDuration = when(duration) {
        SHORT -> Length.LENGTH_SHORT
        LONG -> Length.LENGTH_LONG
    }
    Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

함수는 추상화를 표현하는 수단이며, 함수 시그니처는 이 함수가 어떤 추상화를 표현하는지 알려준다.
따라서 의미있는 함수 이름은 굉장ㅇ히 중요하다.
함수는 매우 단순한 추상화지만, 제한이 많다. 

함수는
- 상태를 유지하지 않음
- 함수 시그니처를 변경하면 프로그램 전체에 큰 영향을 줄 수 있음.

## 클래스
함수보다 더 강력하게 구현을 추상화할 수 있는 방법

```kotlin
class MessageDisplay(val context: Context) {
    fun show(
        message: String,
        duration: MessageLength = MessageLEngth.LONG
    ) {
        val toastDuration = when(duration) {
            SHORT -> Length.LENGTH_SHORT
            LONG -> Length.LENGTH_LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}
enum class MessageLength { SHORT, LONG }

// 사용
val messageDisplay = MessageDisplay(context)
messageDisplay.show("Message")
```
why 함수보다 더 강력한가?
- 상태를 가질 수 있고, 많은 함수를 가질 수 있음
- 의존성 주입 프레임워크를 사용하면, 클래스 생성을 위임할 수 있음
- mock 객체를 활용하면 해당 클래스에 의존하는 다른 캘르스 기능 테스트 가능

## 인터페이스
코틀린은 거의 모든 것이 인터페이스로 표현됨
ex..
- listOf 함수는 List 인터페이스를 리턴 -> listOf는 팩토리 메서드라 할 수 있음
- 컬렉션 처리 함수는 Iterable / Collection의 확장 함수로서, List, Map 등을 리턴 모두 인터페이스임
결국 내부 클래스의 가시성을 제한해, 인터페이스를 통해 이를 노출하는 코드를 많이 사용함.
즉, 인터페이스 뒤에 객체를 숨겨 실질적인 구현을 추상화하고, 사용자가 추상화된 것에만 의존하게 만들 수 있는 것 -> coupling 줄어드는 것
```kotlin
interface MessageDisplay{
    fun show(
        message: String,
        duration: MessageLength = LONG
    )
}

class ToastDisplay(val context: Context): MessageDisplay {
    override fun show(
        message: String,
        duration: MessageLength = LONG
    ) {
        val toastDuration = when(duration){
            SHORT -> Length.SHORT
            LONG -> Length.LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}

enum class MessageLength { SHORT, LONG }
```

위와 같이 구성하면 많은 자유를 얻을 수 있다. 
여러 플랫폼에서 구현을 조금만 다르게 해 사용할 수 있다.
인터페이스 페이킹이 클래스 모킹보다 간단해, 별도의 모킹 라이브러리를 사용하지 않아도 테스트가 가능하다.
선언과 사용이 분리되어 ToastDisplay 등의 실제 클래스를 자유롭게 변경할 수 있다.

## 추상화의 문제
- 추상화는 거의 무한하게 할 수 있다.
- 너무 많은 것을 숨기면 결과를 이해하는 것 자체가 어려움
- 추상화를 이해하려면 예제를 잘 살펴봐야함

문제를 대비해..
- 추상화가 너무 많지도, 적지도 않게 잘 균형을 잘유지
- 팀의 크기, 팀의 경험, 프로젝트의 크기, 특징 세트(feature set), 도메인 지식에 따라 추상화 정도를 조절

## 정리
- 추상화는 단순히 중복성을 제거해 코드를 구성하기 위한 것이 아님
- 코드를 변경할 때 도움이 되는 것이 추상화
- 사용하기 위해 장점과 단점을 모두 이해하고, 균형을 찾아 사용해야함 (그 사이 어딘가를... 맞춰야 한다!)
