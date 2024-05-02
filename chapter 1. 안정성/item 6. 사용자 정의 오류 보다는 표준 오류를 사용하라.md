#아이템6. 사용자 정의 오류보다는 표준 오류를 사용하라
> 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다.

Why❓) 많은 개발자가 알고 있으므로, 이를 재사용하는 것이 좋다.
- IllegalArgumentException          : require를 사용해 throw 할 수 있는 예외
- IllegalStateException             : check를 사용해 throw 할 수 있는 예외
- IndexOutOfBoundsException         : 인덱스 파라미터의 값이 범위를 벗어났을때 발생하는 예외
- UnsupportedOperationException     : 사용자가 사용하려고 했던 메서드가 현재 객체에는 사용 할 수 없다는 것을 나타내는 예외
- NoSuchElementException            : 사용자가 사용하려고 했던 요소가 없을때 발생하는 예외
---
❓) ConcurrentModificationException : 동시수정을 금지 했는데 동시수정이 발생했을때 throw 되는 예외

 ㄴ 이해가 안되서 예제를 찾아봤는데... 진짜 이해가 안된다 ㅠ_ㅠ
 
```kotlin
// ConcurrentModificationException이 발생하는 예제

fun main() {
    // Create a mutable list with some elements
    val list = mutableListOf("A", "B", "C", "D", "E")

    println("Initial list: $list")

    // Use a list iterator to iterate over the list safely
    val iterator = list.listIterator()

    while (iterator.hasNext()) {
        val item = iterator.next()
        
        // Attempt to remove an item from the list safely
        if (item == "C") {
            iterator.remove() // Safe way to remove during iteration
        }
    }

    println("Final list: $list")
}
```
- for 루프는 list를 반복합니다.
- 현재 항목이 "C"인 경우 반복 중에 목록에서 제거됩니다.
- 반복하는 동안 이러한 수정은 ConcurrentModificationException을 트리거합니다.
- try-catch 블록은 예외를 처리하고 오류 메시지를 인쇄합니다.

```kotlin
// ConcurrentModificationException이 발생하지 않도록 리팩토링 된 코드

fun main() {
    // Create a mutable list with some elements
    val list = mutableListOf("A", "B", "C", "D", "E")

    println("Initial list: $list")

    // Use a list iterator to iterate over the list safely
    val iterator = list.listIterator()

    while (iterator.hasNext()) {
        val item = iterator.next()
        
        // Attempt to remove an item from the list safely
        if (item == "C") {
            iterator.remove() // Safe way to remove during iteration
        }
    }

    println("Final list: $list")
}
```
- listIterator를 사용하면 반복 중에 안전하게 목록을 수정할 수 있습니다.
- iterator.remove()를 사용하여 반복 중에 항목을 제거할 수 있습니다.
- 이 코드는 ConcurrentModificationException이 발생하지 않고 의도한 대로 동작합니다.
