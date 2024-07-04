# 아이템 30. 요소의 가시성을 최소화하라

간결한 API를 선호하는 이유
- 작은 인터페이스는 배우기 쉽고 이해하기 쉬움
  - 기능이 많은 클래스보다 적은 클래스가 이해하기 쉬움
  - 유지보수, 테스트 쉬움
- 변경시 기존의 것을 숨기는 것보다 새로운 것을 노출하는 것이 쉬움
- 클래스의 상태를 나타내는 프로퍼티를 외부에서 변경할 수 있다면, 클래스는 자신의 상태를 보장할 수 없음

```kotlin
class CounterSet<T>(
    private val innerSet: MutableSet<T> = setOf()
) : MutableSet<T> by innerSet {

    // 외부에서 set 접근 못하게, but 접근 하게 하고 싶으면 private set 지우면 됨
    var elementsAdded : Int = 0
        private set
        
    override fun add(element: T): Boolean {
        elementsAdded++
        return inerSet.add(element)
    }
    
    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdded == elements.size
        return innerSet.addAll(elements)
    }
}
```
- 일반적으로 코틀린에서 구체 접근자의 가시성을 제한해 모든 프로퍼티를 캡슐화하는 것이 좋음
- 서로 의존하는 프로퍼티가 있을 때, 객체 상태를 보호하는 것이 더 중요
- 많은 것을 제한할수록 병렬 프로그래밍을 할 때 안전함

## 가시성 한정자 사용하기
클래스 멤버 가시성 한정자
- public
- private
- protected
- internal

톱레벨 요소 가시성 한정자
- public
- private
- internal

### 정리
- 인터페이스가 작을수록 공부하고 유지하기 쉬움
- 최대한 제한이 되어야 변경하기 쉬움
- 클래스 상태를 나타내는 프로퍼티가 노출되어 있으면, 클래스가 자신의 상태를 책임질 수 없음
- 가시성이 제한되면 API 변경을 쉽게 추적할 수 있음
