# 아이템 17. 이름 있는 아규먼트를 사용하라

코드에서 아규먼트의 의미가 명확하지 않은 경우가 있 예를 살펴보자.
```kotlin
  // 무슨 의미인지 알기 어렵다.
  val text = (1..10).joinToString("|")
 
  // 변수를 사용해 의미를 명확하게 하는 법
  val separator = "|"
  val text = (1..10).joinToString(separator = separator)
```

## 이름 있는 아규먼트는 언제 사용해야 할까?
이름 있는 아규먼트를 사용하면 코드가 길어지지만, 두가지 장점을 얻을 수 있다.
- 이름을 기반으로 값이 무엇을 의미하는지
- 파라미터 입력 순서와 상관없이 안전

```kotlin
  sleep(100) // 100ms 인지 100s 인지 명확하지 않음
  
  sleep(timeMillis = 100) // sleep(Millis(100)), sleep(100.ms)
```
타입은 이러한 정보를 전달하는 굉장히 좋은 방법이다.
만약 성능에 영향을 줄 것 같아 걱정이 된다면, 추 후 `아이템 46`에 나오는 인라인 클래스를 사용하면 된다.

이름 있는 아규먼트를 사용하면 파라미터의 순서를 잘못 입력하는 등의 문제가 발생하지 않는다.
이런 경우 더욱 사용하자.
- 디폴트 아규먼트
- 같은 타입의 파라미터가 많은 경우
- 함수 타입의 파라미터가 있는 경우(마지막 경우 제외)

### 디폴트 아규먼트
프로퍼티가 디폴트 아규먼트를 가질 경우, 항상 이름을 붙여 사용하는 것이 좋다.

### 같은 타입의 파라미터가 많은 경우
파라미터가 모두 다른 타입이라면, 위치를 잘못 입력하면 오류가 발생해 쉽게 문제를 발견할 수 있다.
하지만, 같은 타입이라면 잘못 입력할 경우 오류를 찾기 어렵다. 이름 있는 아규먼트를 사용하자.
```kotlin
  // 아래와 같은 함수가 있다면 이름 있는 아규먼트를 사용하자
  fun sendEmail(to: String, message: String) { /*...*/ }
  
  sendEmail(to = "contact@kt.academy", message= "Hello,...")
```

### 함수 타입 파라미터
함수 타입 파라미터는 조금 특별하게 다뤄야한다. 
일반적으로 함수타입 파라미터는 마지막 위치에 배치하고 이름을 붙여 사용하면 훨씬 이해하기 쉽다.
```kotlin
val view = linerLayout {
    text("Click below")
    button(onClick = {/*...*/ }) {
        /*...*/
    }
}
```
자바에선 람다 표현식과 주석을 활용했다면, 코틀린에서는 이름 있는 아규먼트를 사용해 의미를 명확하게 할 수 있다.
```kotlin
// RxJava
observable.getUsers()
    .subscribe((List<User> users) -> {  // onNext
      // ...
    }, (Throwable throwable) -> {       // onError
      // ...
    }, () -> {                          // onCompleted
      // ...
    })


// kotlin
observable.getUsers()
    .subscribeBy(
        onNext = { user: List<User> -> /*...*/},
        onError = { throwable: Thorwable -> /*...*/ },
        onCompleted = { /*...*/ }
    )
```
## 정리
이름 있는 아규먼트의 장점
- 디폴트 값 생략
- 코드 가독성, 안정성 향상

이름있는 아규먼트 활용의 예외는 마지막 파라미터가 DSL처럼 특별한 의미를 가질 경우이다.
