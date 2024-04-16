# 정리
- 타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정한다는 원칙만 가지면 된다.
- 안전을 위해 외부 API를 만들 때는 반드시 타입을 지정하고, 이렇게 지정한 타입을 특별한 이유와 확실한 확인 없이 제거하지 말아라.
- inferred 타입은 프로젝트가 진전될 때, 제한이 너무 많아지거나 예측하지 못한 결과가 나온다.
-----

# 아이템4. inferred 타입으로 리턴하지 말라
> 코틀린의 타입추론(type inference)은 JVM 세계에서 가장 널리 알려진 코틀린의 특징이다.

infereed 타입은 정확하게 오른쪽에 있는 피연산자에 맞게 설정된다는 것을 기억해야 한다. 절대 슈퍼클래스 또는 인터페이스로는 설정되지 않는다.
```kotlin
open class Animal
class Zebra: Animal()

fun main() {
	var animal = Zebra()
	animal = Animal() // 오류: Type mismatch

fun main() {
	var animal: Animal = Zebra()  // Type을 명시적으로 선언
	animal = Animal() // 성공
```
직접 라이브러리(모듈)를 조작할 수 없는 경우 위와 같이 타입을 명시적으로 지정해 해결할 수 없다.
inferred 타입을 노출하면, 위험한 일이 발생할 수 있다.

```kotlin
interface CarFactory {
	fun produce(): Car
}

val DEFAULT_CAR: Car = Fiat126P()
```

코드를 작성하다 보니 DEFAULT_CAR는 Car로 명시적으로 지정되어 있으므로 따로 필요 없다고 판단해서, 함수의 리턴 타입을 제거했다고 해보자.
이후, 다른 사람이 DEFAULT_CAR는 타입 추론에 의해 자동으로 타입이 지정될 것이므로, Car를 명시적으로 지정하지 않아도 된다고 생각해서, 다음과 같이 코드를 변경했다고 해보자.
```kotlin
interface CarFactory {
	fun produce() = DEFAULT_CAR
}

val DEFAULT_CAR = Fiat126()
```
위와 같은 코드가 완성된다.
CarFactory에서는 이제 Fiat126P 이외의 자동차를 생산하지 못한다.
외부 API의 경우 위와 같은 문제를 쉽게 해결할 수 없다.
리턴 타입은 API를 잘 모르는 사람에게 전달해 줄 수 있는 중요한 정보이다. 따라서 리턴 타입은 외부에서 확인할 수 있게 명시적으로 지정해주는 것이 좋다.
