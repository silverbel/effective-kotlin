# 아이템 13. Unit? 을 리턴하지 말라

Boolean 과 Unit? 타입은 서로 바꿔 사용할 수 있다.
마치 Boolean이 true / false 를 갖는 것 처럼, Unit?은 Unit / null 값을 가지기 때문이다.

```kotlin
// Boolean
fun keyIsCorrect(key: String): Boolean = //...

if(!keyIsCorrect(key)) return

// Unit?
fun verifyKey(key: String): Unit? = //...
verifyKey(key) ?: return
```
위와 같이 코드를 사용할 수 있다.
즉, Unit?으로 boolean을 표현하는 것은 오해의 소지가 발생하며, 예측하기 어려운 경우가 발생할 수 있다.
=> 차라리 Boolean을 사용하도록 하자.

❓ 필자가 말하려는 정확한 의미는 모르겠지만.. 함수를 작성할때 오해의 소지가 적게 작성하도록 노력해보자!
