# 아이템 7. 결과 부족이 발생할 경우 null과 Failure를 사용하라

## 결과가 부족한 상황
- 서버로부터 데이터를 읽어 들이려고 했는데, 네트워크 오류로 읽어 들이지 못한 경우
- 조건에 맞는 요소가 없는 경우
- 텍스트를 파싱해 객체를 만드려 했는데, 텍스트의 형식이 맞지 않는 경우

이런 상황을 처리하는 메커니즘 2가지
- null 또는 '실패를 나타내는 sealed 클래스'를 리턴한다.
- 예외를 throw 한다.

예외는 정보를 전달하는 방법으로 사용하면 안된다.
예외는 예외적인 상황이 발생했을 때 사용하는 것이 좋다.
- 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못한다.
- 코틀린의 모든 예외는 unchecked 예외이다.(처리하지 않아도 실행에 문제가 없는 예외)
- 예외는 예외적인 상황을 처리하기 위해 만들어졌으므로 명시적 테스트만큼 빠르게 동작하지 않는다.
- try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한된다.

null과 Failure는 예상되는 오류를 표현할 때 굉장히 좋다.
명시적이고, 효율적이며, 간단한 방법으로 처리할 수 있다.
null을 처리하는 경우 safe call이나 Elvis 연산자 사용

```kotlin
// safe call
var str1: String? = null
var len = str1?.length
println(len) // null

// elvis
var str1: String? = null
var len = str1?.length ?: -1
println(len) // -1
```

Result와 같은 공용체(union type)를 리턴하기로 했다면, when 표현식을 사용해서 이를 처리할 수 있다.
- Result 타입 : 성공적인 결과를 캡슐화하거나 임의의 throwable 예외가 있는 실패를 캡슐화하는 식별된 공용체
```kotlin
val person = userText.readObjectOrNull<Person>()
val age = when(person) {
  is Success -> person.age
  is Failure -> -1
}
```
- 이런 오류 처리 방식은 try-catch 블록보다 효율적이고 사용하기 쉽고 더 명확
- 일반적으로 두 가지 형태의 함수를 사용함
- 하나는 예상할 수 있을때와 다른 하나는 예상할 수 없을 때 사용함
  - get: 특정 위치에 있는 요소를 추출할 때 사용
  - getOrNull: out of range 오류가 발생할 수 있는 경우에 사용하고 발생한 경우에 null을 리턴
- 이외에도 getOrDefault를 사용할 수 있음
- null이 발생할 가능성이 있다면 getOrNull등을 사용해서 무엇이 리턴되는지 예측할 수 있게 하는 것이 좋다

---
## ❓ Sealed Class 란
> Sealed Class 는 Enum Class 의 확장판과도 같다.
>
> 제한적인 계층관계를 효과적으로 표현할 수 있고, 이에 따라 when 문 사용 시 효과적으로 사용할 수 있다.

### enum의 제약사항
- 각 enum 상수들은 단 하나의 인스턴스만 가질 수 있음 (싱글톤)
- enum 상수 정의 이후에 속성값을 변경할 수 없음
- enum 클래스에 대해 서브 클래스를 생성할 수 없음

### sealed Class의 장점
- sealed 클래스의 서브 클래스들은 반드시 같은 파일 내에 선언되어야 함
- 단, sealed 클래스의 서브 클래스를 상속한 클래스들은 같은 파일 내에 없어도 됨
- sealed 클래스는 기본적으로 abstract 클래스임
- sealed 클래스는 private 생성자만 갖게 됨

```kotlin
// sealed 클래스 선언 예제
sealed class Color {
    data class Red(val r: Int, val g: Int, val b: Int) : Color()
    data class Orange(val r: Int, val g: Int, val b: Int) : Color()
    data class Yellow(val r: Int, val g: Int, val b: Int) : Color()
    data class Green(val r: Int, val g: Int, val b: Int) : Color()
    data class Blue(val r: Int, val g: Int, val b: Int) : Color()
    data class Indigo(val r: Int, val g: Int, val b: Int) : Color()
    data class Violet(val r: Int, val g: Int, val b: Int) : Color()
}
```
```kotlin
// sealed 클래스 사용 예제
fun main() {
    val color: Color = Color.Red(255, 0, 20)
    when (color) {
        is Color.Red -> println("빨강")
        is Color.Orange -> println("주황")
        is Color.Yellow -> println("노랑")
        is Color.Green -> println("초록")
        is Color.Blue -> println("파랑")
				is Color.Indigo -> println("인디고")
				is Color.Violet -> println("바이올렛")
    }
}
```
