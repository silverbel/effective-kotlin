# 아이템 14. 변수 타입이 명확하지 않은 경우 확실하게 지정하라

코틀린은 컴파일 시점에 타입이 결정된다.
코틀린은 수준 높은 타입 추론 시스템을 갖추고 있다.

```kotlin
  val num = 10 // Int
  val name = "Marcin" // String
  val ids = listOf(12, 112, 554, 997) // List<Int>
```

유형이 명확할 때 코드가 짧아져 가독성이 크게 향상된다.

but, 유형이 명확하지 않을 때 좋지 않다.

```kotlin
  val data = getSomeData() // 타입을 숨김
```
이런 코드는 가독성이 떨어진다.
깃허브 같은 환경에서는 코드 정의로 쉽게 이동하기 어렵다.

따라서 유형이 명확하지 않다면 아래와 같이 타입을 지정해준다.
```kotlin
  val data: UserData = getSomeData() 
```
이는 [아이템3](https://github.com/silverbel/effective-kotlin/blob/main/chapter%201.%20%EC%95%88%EC%A0%95%EC%84%B1/item%203.%20%EC%B5%9C%EB%8C%80%ED%95%9C_%ED%94%8C%EB%9E%AB%ED%8F%BC_%ED%83%80%EC%9E%85%EC%9D%84_%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80_%EB%A7%90%EB%9D%BC.md), [아이템4](https://github.com/silverbel/effective-kotlin/blob/main/chapter%201.%20%EC%95%88%EC%A0%95%EC%84%B1/item%204.%20inferred%20%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C%20%EB%A6%AC%ED%84%B4%ED%95%98%EC%A7%80%20%EB%A7%90%EB%9D%BC.md) 에서 다루었듯 가독성 향상 이외에 안전을 위한 행위다.

### ❗ 타입을 무조건 지정하는 것이 아닌 상황에 맞게 사용하면 된다.
