# 정리
- 변수의 스코프는 좁게 만들어 활용하는 것이 좋음
- var 보단 val을 사용하는 것이 좋음
- 람다에서 변수를 캡처한다
---

# 아이템 2. 변수의 스코프를 최소화하라

- 프로퍼티보다는 지역 변수를 사용하는 것이 좋다.
- 최대한 좁은 스코프를 갖게 변수를 사용한다.

```kotlin
// 나쁜 예
var user: User
for(i in users.indices){
  user = users[i]
  print("User at $i is $user")
}

// 좀 더 좋은 예
for(i in users.indices){
  val user = users[i]
  print("User at $i is $user")
}

// 제일 좋은 예
for((i, user) in useurs.withIndex()){
  print("User at $i is $user")
}
```
스코프를 좁게 만드는 중요한 이유
- 프로그램을 추적하고 관리하기 쉬움
- mutable 프로퍼티는 좁은 스코프에 걸쳐 있을수록, 그 변경을 추적하기 쉬움
- 변수의 스코프 범위가 넓을 수록, 다른 개발자에 의해 잘못 사용될 수 있음
- 변수는 읽기 전용 또는 읽고 쓰기 전용 여부와 상관 없이, 변수를 정의할 때 초기화되는 것이 좋음
- if, when, try-catch, Elvis 표현식 등을 활용하면, 최대한 변수를 정의할 때 초기화할 수 있음

```kotlin
// 나쁜 예
val user: User
if(hasValue){
  user = getValue()
} else{
  user = User()
}

// 조금 더 좋은 예
val user: User = if(hasValue) {
  getValue()
} else{
  User()
}
```
여러 프로퍼티를 한번에 설정해야 하는 경우 구조분해 선언을 활용하는 것이 좋다.

```kotlin
// 나쁜 예
fun updateWeather(degrees: Int){
  val description: String
  val color: Int
  if(degrees < 5){
    description = "cold"
    color = Color.BLUE
  } else if (degrees < 23) {
    description = "mild"
    color = Color.YELLOW
  } else {
    description = "hot"
    color = Color.RED
  }
  // ...
}

// 조금 더 좋은 예
fun updateWeather(degrees: Int){
  val (description, color) = when {
    degrees < 5 -> "cold" to Color.BLUE
    degrees < 23 -> "mild" to Color.YELLOW
    else -> "hot" to Color.RED
  }
  // ...
}
```

## 캡처링

에라토스테네스의 체를 구현하여 알아본다.

1. 2부터 시작하는 수자리스트(2~100)을 만든다.
2. 첫 번째 요소를 선택한다. 이는 소수
3. 남아 있는 숫자 중에서 2번에서 선택한 소수로 나눌 수 있는 모든 숫자 제거

```kotlin
// 간단한 구현
var numbers = (2..100).toList()
val primes = mutableListOf<Int>()
while(numbers.isNotEmpty()){
  val prime = numbers.first()
  primes.add(prime)
  numbers = numbers.filter { it%prime != 0 }
}
print(primes) // // [2,3,5,7,11,13,17,19,23,29,31,37,41,43,47,53,59,61,67,71,73,79,83,89,97]

// 시퀀스 할용 예제
val primes: Sequence<Int> sequence {
  val numbers = generateSequence(2) {it + 1}
  
  while(true){
    val prime = numbers.first()
    yield(prime)    // 탈출 조건 : prime이 없다면 종료
    number = numbers.drop(1).filter { it % prime != 0 }
  }
}

print(primes.take(10).toList()) // [2,3,5,7,11,13,17,19,23,29]
```

```kotlin
// 잘못된 최적화
val primes: Sequence<Int> = sequence {
  var numbers = generateSequence(2) { it + 1 }

  var prime: Int
  while (true) {
    prime = numbers.first()
    yield(prime)
    numbers = numbers.drop(1).filter { it % prime != 0 }
  }
}
print(primes.take(10).toList()) // [2,3,5,6,7,8,9,10,11,12]
```

:interrobang:잘못된 이유
- prime 변수를 캡처했기 때문
- 반복문 내부에서 filter를 활용해 숫자를 필터링하지만, 시퀀스를 활용해 필터링이 지연됨. 최종 prime 값으로만 필터링됨
- prime이 2로 설정되어 있을 때 필터링된 4를 제외하면, drop만 동작해 연속된 숫자가 나옴
