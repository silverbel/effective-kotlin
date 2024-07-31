# 아이템 41. hashCode의 규약을 지켜라

`hashCode` 함수는 수많은 컬렉션과 알고리즘에 사용되는 자료 구조인 해시테이블 구축시 사용됩니다.

## 해시 테이블
- 선형 자료구조의 탐색 성능이 좋지 않은 점을 해결할 수 있는 방법
- 해시 함수는 각 요소에 숫자를 할당하고 이를 기반으로 요소를 다른 버킷에 넣음
  - 해시함수가 가지면 좋은 특성 : 빠름, 충돌이 적음
- 버킷은 버킷 수와 같은 크기의 배열인 해시 테이블에 보관됨

## 가변성과 관련된 문제
- 요소가 추가될 때만 해시 코드를 계산
- 요소 변경시 해시코드 계산하지 않고 버킷 재배치도 이뤄지지 않음
- 기본적인 `LinkedHashSet`과 `LinkedHashMap`의 키는 한번 추가한 요소를 변경할 수 없음
```kt
data class FullName(
    var name: String,
    var surname: String
)

val person = FullName("Lee", "Chang")
val s = mutableSetOf<FullName>()
s.add(person)
person.surname = "Min"
print(person) // FullName(name=Lee, surname=Min)
print(person in s) // false
print(s.first() == person) true
```
- 세트와 맵의 키로 `mutable` 요소를 사용하면 안됨, 사용해도 요소를 변경하면 안됨

## hashCode의 규약
코틀린 1.3.11 기준 규약
- 어떤 객체를 변경하지 않았다면, hashCode는 여러 번 호출해도 결과가 항상 같다
- equals 메서드의 실행 결과로 두 객체가 같으면, hashCode 함수 호출 결과도 같다

## hashCode 구현하기
일반적으로 data 한정자를 붙이면, 코틀린이 알아서 적당한 equals와 hashCode를 정의해 줘 직접 정의할 일은 거의 없다

1. `equals`를 따로 정의했으면 `hashCode`함수도 같이 정의
2. `equals`로 같은 요소임을 판정하면 `hashCode`도 반드시 같은 값 리턴
3. 코틀린/JVM의 `Objects.hashCode`는 해시를 계산해줌
4. 코틀린 stdlib은 `hashCode`를 직접 구현할 일이 거의 없어 이런 함수를 제공하지 않음 (`data`한정자 사용시 알아서 됨)

가장 중요한 점은 **언제나 equals와 일관된 결과가 나와야 한다**
