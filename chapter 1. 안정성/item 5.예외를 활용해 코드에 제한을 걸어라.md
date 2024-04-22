# 정리
- 제한을 훨씬 더 쉽게 확인할 수 있따.
- 애플리케이션을 더 안정적으로 지킬 수 있다.
- 코드를 잘못 쓰는 상황을 막을 수 있다.
- 스마트 캐스팅을 활용할 수 있다.
----

# 아이템5. 예외를 활용해 코드에 제한을 걸어라

코틀린에서 코드의 동작에 제한을 걸 때 사용할 수 있는 방법
- require 블록: 아규먼트 제한
- check 블록: 상태와 관련된 동작 제한
- assert 블록: 어떤 것이 true인지 확인 (테스트 모드에서만 작동)
- return 또는 throw와 함께 활용하는 Elivs 연산자

```kotlin
// Stack<T>의 일부
fun pop(num: Int = 1) : List<T> {
  require(num <= size) {
    "Cannot remove more elements than current size"
  }
  check(isOpen) { "Cannot pop from closed stack" }
  val ret = collection.take(num)
  collection = collection.drop(num)
  assert(ret.size == num)
  return ret
}
```
제한을 거는 것의 장점
- 제한을 걸면 문서를 읽지 않은 개발자도 문제를 확인할 수 있다.
- 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 예외를 throw한다. 예상하지 못한 동작을 하는 것은 예외를 throw하는 것보다 굉장히 위험하며, 상태를 관리하는 것이 힘들다.
- 코드가 어느 정도 자체적으로 검사된다. 관련된 단위 테스트를 줄일 수 있다.
- 스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스트(타입 변환)를 적게 할 수 있다.

## 아규먼트
함수를 정의할 때 타입 시스템을 활용해 아규먼트에 제한을 거는 코드를 많이 사용한다.
- 숫자를 아규먼트로 받아서 팩토리얼을 계산한다면 숫자는 양의 정수여야 한다.
- 좌표들을 아규먼트로 받아서 클러스트를 찾을 때는 비어있지 않은 좌표 목록이 필요하다.
- 사용자로부터 이메일 주소를 입력받을 때는 값이 입력되어 있는지, 그리고 이메일 형식이 올바른지 확인해야 한다.
일반적으로 이러한 제한을 걸 때는 require함수를 사용한다. require 함수는 제한을 확인하고, 제한을 만족하지 못할 경우 예외를 throw한다.
```kotlin
fun factorial(n: Int): Long{
  require(n >= 0)
  return if (n <= 1) 1 else factorial(n - 1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
  require(points.isNotEmpty())
  //...
}

fun sendEmail(user: User, message: String) {
  requireNotNull(user.email)
  require(isValidEmail(user.email))
}
```
이와 같은 형태의 입력 유효성 검사 코드는 함수의 가장 앞부분에 배치되므로, 읽는 사람도 쉽게 확인할 수 있다.(물론 문서에 관련된 내용을 반드시 명시해야함)
require 함수는 조건을 만족하지 못할 때 무조건적으로 IllegalArgumentException을 발생시키므로 제한을 무시할 수 없다.
일반적으로 이러한 처리는 함수의 가장 앞부분에 하게 되므로, 코드를 읽을 때 쉽게 확인할 수 있다.

람다를 활용해 지연 메시지를 정의할 수도 있다.

❓ 람다가 어디... 있지...?
```kotlin
fun factorial(n: Int): Long{
  require(n >= 0) { "Cannot calculate factorial of $n " + "because it is smaller than 0" }
  return if (n <= 1) 1 else factorial(n - 1) * n
}
```
## 상태
어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야할 때가 있다.
- 어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고 싶은 함수
- 사용자가 로그인했을 때만 처리를 하게 하고 싶은 함수
- 객체를 사용할 수 있는 시점에 사용하고 싶은 함수
```kotlin
fun speak(text: String) {
  check(isInitialized)
  //...
}

fun next(): T {
  check(isOpen)
  //...
}
```
check 함수는 require과 비슷하지만, 지정된 예측을 만족하지 못할 때, IllegalStateException을 던진다.
예외 메시지는 require과 마찬가지로 지연 메시지를 사용해 변경할 수 있다.
함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 require 블록 뒤에 배치한다.(check를 나중에)

## Assert 계열 함수 사용
- 테스트할 때 사용
```kotlin
fun pop(num: Int = 1): List<T> {
  //...
  assert(ret.size == num)
  return ret
}
```
테스트를 할 때만 활성화된다.
Assert 사용의 장점
- 코드를 자체 점검, 더 효율적인 테스트
- 특정 상황이 아닌 모든 상황에 대한 테스트
- 실행 시점에 정확하게 어떻게 되는지 확인 가능
- 실제 코드가 더 빠른 시점에 실패하게 만든다. 따라서 예상하지 못한 동작이 언제 어디서 실행되었는지 쉽게 찾을 수 있음

## nullability와 스마트 캐스팅
코틀린에서 require과 check 블록으로 어떤 조건을 확인해 true가 나왔다면, 해당 조건은 이후로도 true일 것이라고 가정한다.
이를 활용해 타입 비교를 했다면, 스마트 캐스트가 작동한다.
```kotlin
fun changeDress(person: Person){
  require(person.outfit is Dress)
  val dress: Dress = person.outfit
  //...
}
```
위 코드는 person.outfit이 dress여야 작동한다. 만약 outfit 이 final 이라면, Dress로 스마트캐스트 된다.
이런 특징은 어떤 대상이 null인지 확인할 때 유용하다.
```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
  require(person.email != null)
  val email: String = person.email
  //...
}
```
이런 경우 requireNotNull, checkNotNull 을 사용해도 된다. 둘 다 스마트 캐스트를 지원하므로, 변수를 언팩하는 용도로 활용할 수 있다.

```kotlin
class Person(val email: String?)
fun validateEmail(email: String) { /*...*/ }

fun sendEmail(person: Person, message: String) {
  val email = requireNotNull(person.email)
  validateEmail(email)
  //...
}

fun sendEmail(person: Person, message: String) {
  requireNotNull(person.email)
  validateEmail(email)
  //...
}
```
elvis연산자 활용하는 경우 오류를 발생시키지 않고 단순하게 함수를 중지시킬 수 있다.
```kotlin
fun sendEmail(person: Person, text: String) {
  val email: String = person.email ?: return
  //...
}
```
프로퍼티에 문제가 있어서 null 일 때 여러 처리를 해야 할 때도, return/throw와 run 함수를 조합해서 활용하면 된다. (미선님 코멘트 : try-catch-finally에서 finally 같은 느낌입니다)
```kotlin
fun sendEmail(person: Person, text: String) {
  val email: String = person.email ?: run {
    log("Email not sent, no email address")
    return
  }
  //...
}
```
return과 throw를 활용한 Elvis 연산자는 nullable을 확인할 때 굉장히 많이 사용되는 관용적인 방법이다.
따라서 적극적으로 활용하는 것이 좋다.
이런 코드는 함수의 앞부분에 넣어 잘 보이게 만드는 것이 좋다.
