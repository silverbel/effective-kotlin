# 아이템 37. 데이터 집합 표현에 data 한정자를 사용하라

데이터들을 한번에 전달할 때 data class 를 사용한다.

data 한정자를 붙이면, 아래 함수가 자동으로 생성된다.
- toString
- equals, hashCode
- copy
- componentN(component1, component2 등)

data class 를 사용하면 componentN을 활용해 객체를 해체하는 장점이 있다.

순서를 잘못 지정하는 경우, 값을 하나만 갖는 경우에는 객체를 해체하지 않는 것이 좋다.
람다와 함께 활용할 때 문제가 된다.
```kt
data class User(val name: String)

fun main() {
	val user = User("John")
	user.let { a -> print(a) } // 사용자(name = John)
	user.let { (a) -> print(a) } // John
}
```

## 튜플 대신 데이터 클래스 사용하기

데이터 클래스는 튜플보다 많은 것을 제공한다.

구체적으로 코틀린의 튜플은 Serializable을 기반으로 만들어지며, toString을 사용할 수 있는 제네릭 데이터 클래스이다.
```kt
public data class Pair<out A, out B> (
    public val first: A,
    public val second: B
) : Serializable {

    public override fun toString(): String =
        "($first, $second)"
}

public data class Triple<out A, out B, out C>(
    public val first: A,
    public val second: B,
    public val third: C
) : Serializable {

    public override fun toString(): String =
        "($first, $second, $third)"
}
```
튜플은 데이터 클래스와 같은 역할 but, 가독성이 나쁘다.

튜플만 보곤 어떤 타입을 나타내는 지 예측할 수 없음.

Pair와 Tipble은 몇가지 지역적 목적으로 인해 남아있음
- 값에 간단한 이름을 붙일 때
- 표준 라이브러리에서 볼 수 있는 것처럼 미리 알 수 없는 aggregate를 표현

이 경우들을 제외하면 무조건 데이터 클래스가 좋다.

----------
(추가 내용)

## 데이터 클래스의 문제점
- 엔티티 클래스로는 사용안하는 것을 권장
- 1:N 관계에서 순환 참조로 인해 StackOverFlow Error 발생할 수 있다.
- 해결 방안: toString, equals, hashCode를 오버라이드해서 순환 참조를 방지한다. 또는 한쪽에서 삭제한다.
> https://stackoverflow.com/questions/48926704/kotlin-data-class-entity-throws-stackoverflowerror
- copy의 얕은 복사로 인해 참조가 복사되어 문제가 발생할 수 있다.
- Hibernate의 Lazy Loading을 사용하기 위해서는 데이터 클래스를 사용할 수 없다. (프록시 객체 문제)
> https://velog.io/@donghyunele/kotlin-jpa-2-%EC%9A%B0%EC%95%84%ED%95%9C%ED%98%95%EC%A0%9C%EB%93%A4-%EA%B8%B0%EC%88%A0%EB%B8%94%EB%A1%9C%EA%B7%B8
- 자동으로 만들어주는 함수 때문에 용량이 크다. (App의 경우는 용량에 대한 이슈가 있을 수 있다.)



## 데이터 클래스를 사용하면 추가 비용은 거의 들지 않고, 오히려 함수를 명확하게 만들어 준다.
- 함수의 리턴 타입 명확
- 리턴 타입 짧아지며, 전달이 쉬워짐
- 사용자가 데이터 클래스에 적혀 있는 것과 다른 이름을 활용해 변수를 해체하면, 경고 출력
