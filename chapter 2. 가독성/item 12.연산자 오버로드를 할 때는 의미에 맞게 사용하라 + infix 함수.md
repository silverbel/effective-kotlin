# 아이템 12. 연산자 오버로드를 할 때는 의미에 맞게 사용하라
연산자 오버로딩은 강력한 기능이지만, 위험할 수 있다.

예를 들어 팩토리얼을 정의하는 확장함수를 만든다고 생각해보자.

```kotlin
fun Int.factorial(): Int = (1..this).product()

fun Iterable<Int>.product(): Int = fold(1) { ac,, i -> acc * i }
```
코틀린은 6! 같은 연산자를 지원하지 않지만, 다음과 같이 연산자 오버로딩을 활용하면 만들어 낼 수 있다.
```kotlin
operator fun Int.not() = factorial()

print(10 * 6!) // 7200
```
이렇게 할 수는 있지만, 이렇게 하면 안된다.
이 함수의 이름이 not이다.
not은 논리 연산에 사용해야지 팩토리얼 연산에 사용하면 안된다.

## 분명하지 않은 경우

함수를 곱한다는 것(* 연산자)를 사용하는 것은 아래와 같은 의미가 될 수 있다.
1. 함수를 여러번 반복하는 새로운 함수 ( () -> Unit )을 만들어낸다.
2. 같은 함수를 여러번 호출한다.

의미가 명확하지 않다면, infix를 활용한 확장 함수를 사용하는 것이 좋다.
일반적인 이항 연산자 형태처럼 사용할 수 있다.
```kotlin
infix fun Int.timesRepeated(operation: () -> Unit) = {
  reapeat(this) { operation() }
}

val tripledHello = 3 timesRepeated { print("Hello") }
tripledHello() // HelloHelloHello
```
또는 톱레벨 함수를 사용하는 것도 좋다.
```kotlin
repeat(3) { print("Hello") } // HelloHelloHello
```

## 규칙을 무시해도 되는 경우
위에서 설명한 규칙들을 무시해도 되는 경우는 도메인 특화 언어(DSL)을 설계할 때 이다.
- 해당 도메인을 위해 설계된 DSL 이기 때문이다.
❓ DSL을 많이 사용할까...? 안쓸거 같음...

``` kotlin
// HTML DSL. 코틀린 문법을 활용해서 만든 DSL이다.
body {
    div {
        +"Some Text" // <- String.unaryPlus 가 문자열 앞에 사용되었다.
    }
}

// Gradle DSL. 이것도 코틀린 문법을 이용해서 만든 DSL 이다. Gradle 5.0부터 지원한다.
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile
plugins {
    kotlin("jvm") version "1.6.10"
    application
}
version = "1.0-SNAPSHOT"
dependencies {
    testImplementation(kotlin("test"))
}
tasks.withType<KotlinCompile> {
    kotlinOptions.jvmTarget = "1.8"
}
application {
    mainClass.set("MainKt")
}
```
DSL 코드의 경우 연산자를 잘 못 이해할 일이 없으므로, 사용해도 상관이 없다.

## 정리
- 연산자 오버로딩은 그 이름의 의미에 맞게 사용하는게 좋다.
- 연산자의 의미가 명확하지 않다면, 사용하지 않는 것이 좋다.
- 연산자 같은 형태로 사용하고 싶다면, infix 확장 함수 또는 톱레벨 함수를 활용해라.

-----
## 중위 호출(infix call) 이란
> 함수가 두 개의 인자를 받을 때, 이 함수를 호출하는 보다 읽기 쉽고 자연스러운 문법을 제공한다. infix 함수는 다음 조건을 만족해야 한다.

- 멤버 함수이거나 확장 함수여야 한다.
- 오직 하나의 파라미터를 가져야 한다.
- 파라미터는 가변 인자(var-arg)를 사용할 수 없으며, 기본값을 가질 수 없다.

```kotlin
// 예시
infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)

val myPair = "key" to "value"
```
여기서 to 함수는 infix로 선언되었기 때문에, 위와 같이 . 없이 함수 이름과 인자 사이에 공백을 두고 호출할 수 있다.
