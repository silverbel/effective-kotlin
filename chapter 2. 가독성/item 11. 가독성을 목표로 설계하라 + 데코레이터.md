# 아이템 11. 가독성을 목표로 설계하라

> “개발자가 코드를 작성하는데는 1분 걸리지만, 이를 읽는데는 10분이 걸린다”  
>                                - Robert C. Martin (클린코드)

- 코드를 작성하다 오류가 발생하면, 오랜 시간 코드를 읽는다.
- 항상 가독성을 생각하며 코드를 작성해야한다.

## 인식 부하 감소
```kotlin
// 구현 A
if (person != null && person.isAdult) {
  view.showPerson(person)
} else {
  view.showError()
}
// 구현 B
person?.takeIf { it.isAdult }
  ?.let(view::showPerson)
  ?: view.showError()
```
- 코틀린 초보자는 A가 더 이해하기 쉽다.
- 숙련된 코틀린 개발자는 B도 쉽게 읽을 수 있다.
- 숙련된 개발자만을 위한 코드는 좋은 코드가 아니다.
- A와 B는 비교조차 할 수 업을 정도로 A가 훨씬 가독성이 좋은 코드이다.

**구현 A가 확장하기도 더 쉽다.**

```kotlin
// 구현 A
if (person != null && person.isAdult) {
  view.showPerson(person)
  view.hideProgressWithSuccess()
} else {
  view.showError()
  view.hideProgress()
}
// 구현 B
person?.takeIf { it.isAdult }
  ?.let {
    view.showPerson(it)
    view.hideProgressWithSuccess()
  } ?: run {
    view.showError()
    view.hideProgress()
  }
```
- 구현 A는 확장을 위해 if/else 블록에 코드를 추가하면 된다.
- 구현 B는 확장을 위해 함수를 추가로 사용해야한다.

**디버깅 또한 A가 더 쉽다.**
- 일반적인 디버깅 도구조차 이런 구조를 더 잘 분석해 주기 때문

**구현 A와 구현 B의 결과가 다르다**
- 구현 A는 showPerson 혹 showError를 호출한다.
- 구현 B에서 let은 람다식의 결과를 리턴하기 때문에, showPerson이 null을 리턴할 경우 showError도 호출된다.
- 익숙하지 않은 구조(구현 B)의 결과를 예측하기 어렵다.

## 극단적이 되지 않기
앞에서 let으로 인해 예상하지 못한 결과가 나올 수 있다고 했다고 let을 절대로 쓰면 안된다고 생각하면 안된다.
-nullable 안전 호출

```kotlin
class Person(val name: String)
var person: Person? = null

// person != null 일 때만 실행하고 싶은 코드가 있는 경우
fun printName(){
  person?.let{
    println(it.name)
  }
}
```

- 연산을 아규먼트 처리 후로 이동시킬 때
```kotlin
students
  .filter { it.result >= 50 }
  .joinToString(separator = "\n") {
    "${it.name} ${it.surname}, ${it.result}"
  }
  .let(::print) // print 를 뒤로 이동시킨 경우
```

- 데코레이터를 사용해서 객체를 wrap 할 때 ex) `class(myClass(decoClass()))`
```kotlin
var obj = FileInputStream("/file.gz")
    	.let(::BufferedInputStream)
    	.let(::ZipInputStream)
    	.let(::ObjectInputStream)
    	.readObject() as SomeObject
```

## 컨벤션
사람에 따라 가독성에 대한 관점이 다르다.

- 함수 이름을 어떻게 지어야 하는지
- 어떤 것이 명시적이고, 어떤 것이 암묵적이여야 하는지
- 어떤 관용구를 사용하는지

프로그래밍은 표현력의 예술이다. 이를 위해 이해하고 기억해야 하는 몇가지 규칙이 있다.
```kotlin
val abc = "A" { "B" } and "C"
print(abc) // ABC
```
이렇게 사용하기 위해서는, 아래와 같은 코드가 필요하다.

```kotlin
operator fun String.invoke(f: ()->String): String = this + f()

fun main() {
    val myString = "Hello, "
    
    // 람다를 사용하여 'myString'을 함수처럼 호출
    val result = myString {
        "World!"
    }
    
    println(result) // 출력: "Hello, World!"
}
```

```kotlin
// 중위 함수(Infix Function)란?
// 중위 함수는 함수 호출 시 점(dot)과 괄호를 사용하지 않고, 키워드 형태로 사용할 수 있는 함수입니다. 중위 함수는 일반적으로 두 피연산자를 연결하거나 비교하는 작업을 간결하게 표현하는 데 사용됩니다.

infix fun String.and(s: String) = this + s

val part1 = "Hello"
val part2 = "World"

// 전통적인 연결 방식
val result1 = part1 + " " + part2

// 중위 함수를 활용한 연결 방식
val result2 = part1 and " " and part2

println(result1) // 출력: "Hello World"
println(result2) // 출력: "Hello World"
```
이는 아래의 컨벤션 규칙을 위반한다.
- '연산자는 의미에 맞게 사용해야 한다. invoke를 이런 형태로 사용하면 안된다.
- '람다를 마지막 아규먼트로 사용한다' 라는 컨벤션을 적용하면 코드가 복잡해진다.
- '함수 이름을 보고 동작을 유추할 수 있어야한다.' and라는 함수 이름이 실제 함수 내부에서 이루어지는 처리와 맞지 않습니다.
- 문자열을 결합하는 기능은 이미 언어에 내장되어 있다. 이미 있는 것을 다시 만들 필요는 없다.

---
# ❓데코레이터 패턴이란
> 기존 객체의 구조를 변경하지 않고 기능을 확장할 수 있는 유연한 방법을 제공하는 객체지향 패턴

### 장점
- 확장성: 객체의 기본 기능을 변경하지 않고 추가적인 기능을 제공할 수 있다.
- 유연성: 여러 데코레이터를 결합하여 다양한 방식으로 객체를 확장할 수 있다.
- 단일 책임 원칙 준수: 각 데코레이터는 특정 기능만 추가하므로, 코드의 모듈성을 높이고 유지보수가 용이하다.

### 예시
- 이렇게 하면 기존 객체를 수정하지 않고도 다양한 방식으로 기능을 조합할 수 있다.
```kotlin
// 메시지 서비스 인터페이스
interface MessageService {
    fun sendMessage(message: String)
}

// 기본 메시지 서비스를 구현하는 클래스
class BasicMessageService : MessageService {
    override fun sendMessage(message: String) {
        println("Sending message: $message")
    }
}

// 로깅 기능을 추가하는 데코레이터
class LoggingDecorator(private val service: MessageService) : MessageService {
    override fun sendMessage(message: String) {
        println("Logging message: $message")
        service.sendMessage(message) // 기본 서비스에 위임
    }
}

// 메시지를 암호화하는 데코레이터
class EncryptionDecorator(private val service: MessageService) : MessageService {
    override fun sendMessage(message: String) {
        val encryptedMessage = "Encrypted($message)" // 간단한 암호화 예시
        service.sendMessage(encryptedMessage) // 기본 서비스에 위임
    }
}

// 데코레이터 패턴 적용 예시
fun main() {
    val basicService = BasicMessageService()

    // 로깅 데코레이터로 랩핑
    val loggingService = LoggingDecorator(basicService)
    loggingService.sendMessage("Hello, World!") // 출력: Logging message, Sending message

    // 암호화 데코레이터로 랩핑
    val encryptedLoggingService = EncryptionDecorator(loggingService)
    encryptedLoggingService.sendMessage("Secret Message") // 출력: Logging message, Sending message with encryption
}
```
