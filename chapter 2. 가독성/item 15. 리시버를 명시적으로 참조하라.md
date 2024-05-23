# 아이템 15. 리시버를 명시적으로 참조하라

코틀린에서 리시버를 사용하는 경우가 있다. ex. this
```kotlin
Class User: Person(){
  private var beersDrunk: Int = 0
  
  fun drinknBeers(num: Int){
    this.beersDrunk += num
  }
}
```
확장 메소드에서 this를 명시적으로 참조할 수 있다.
아래는 리시버를 명시적으로 표시하지 않은 퀵소트와 명시적으로 표시한 퀵소트이다.
```kotlin
// 명시적으로 표시하지 않은 퀵소트
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
  if (size < 2) return this
  val pivot = first()
  val (smaller, bigger) = drop(1)
    .partition { it < pivot }
  return smaller.quickSort() + pivot + bigger.quickSort()
}

// 명시적으로 표시한 퀵소트
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
  if (size < 2) return this
  val pivot = this.first()
  val (smaller, bigger) = this.drop(1)
    .partition { it < pivot }
  return smaller.quickSort() + pivot + bigger.quickSort()
}
```
두 함수의 사용 결과는 차이가 없다.
this를 생략하면 코드가 간결해져 읽기 좋아진다.

## 여러 개의 리시버
스코프 내부에 아래와 같이 둘 이상의 리시버가 있는 경우, 리시버를 명시적으로 나타내면 좋다.
```kotlin
class Node(val name: String){
  fun makeChild(childName: String) =
    // 클래스의 this.name 인지 새로 생성한 자식 node.name 인지 알 수 없다.
    create("$name.$childName")
      .apply { print("Created ${name}") }
      
      fun create(name: String): Node? = Node(name)
}

fun main(){
  val node = Node("parent")
  node.makeChild("child") // created parent
}
```
위 코드의 결과는 일반적으로 Created parent.child 라고 예상하지만, 실제로는 Created parent이다.
이해하기 쉽게 앞에 리시버를 붙여보자.

```kotlin
class Node(val name: String){
  fun makeChild(childName: String) =
    create("$name.$childName")
      .apply { print("Created ${this.name}") } // compile error
      
      fun create(name: String): Node? = Node(name)
}

fun main(){
  val node = Node("parent")
  node.makeChild("child") // created parent
}
```
기존의 코드에서 this가 붙었다. 이런 경우 this의 타입이 Node?여서 컴파일 에러가 발생한다.
그러면 this?.name 을 사용하면 되지않나 라고 할 수 있다. 이는 apply의 잘못된 사용이다.

> 일반적으로 **nullable 값을 처리할 때 also 또는 let을 사용**하는 것이 더 좋다.

```kotlin
class Node(val name: String){
    fun makeChild(childName: String) {
        create("$name.$childName")
            .also { print("Created ${it?.name}") }
    }

    fun create(name: String) = Node(name)
}
```

```kotlin
class Node(val name: String){
    fun makeChild(childName: String) =
        create("$name.$childName").apply {
            println("Created ${this?.name} in " + "${this@Node.name}")
        }

    fun create(name: String): Node? = Node(name)
}

fun main(){
    val node = Node("eun")
    node.makeChild("child") // created parent
}
```
이렇게 명확하게 작성하면, 코드를 안전하게 사용할 수 있을 뿐만 아니라 가독성도 향상됩니다...?
-> 솔직히 말해서.. 가독성이 매우매우 안좋다!!!

## DSL 마커
코틀린 DSL을 사용할 때 여러 리시버를 가진 요소들이 중첩되더라도, 리시버를 명시적으로 붙이지 않는다. DSL은 리시버를 생략하도록 설계되었기 때문에..
그렇지만, DSL 외부의 함수를 사용하는 것이 위험한 경우가 있다.
```kotlin
table {
    tr{
        td{ + "Column1"}
        td{ + "Column2"}
    }
      tr{
          td{ + "Value1"}
          td{ + "Value2"}
      }
}
```
이런 잘못된 사용을 막으려면, 암묵적으로 외부 리시버를 사용하는 것을 막는 DslMarker라는 메타 어노테이션을 사아ㅛㅇ한다.
```kotlin
@DslMarker
annotation class HtmlDsl

fun table(f: TableDsl.() -> Unit) { /*...*/ }

@HtmlDsl
class TableDsl { /*...*/ }
```
이렇게 하면 암묵적으로 외부 리시버를 사용하는 것이 금지된다.

```kotlin
table {
    tr{
        td{ + "Column1"}
        td{ + "Column2"}
    }
      tr{ // 컴파일 오류
          td{ + "Value1"}
          td{ + "Value2"}
      }
}

// 사용하고 싶으면
table {
    tr{
        td{ + "Column1"}
        td{ + "Column2"}
    }
      this@table.tr{
          td{ + "Value1"}
          td{ + "Value2"}
      }
}
```

## 정리
- 짧게 적을 수 있따는 이유만으로 리시버를 제거하지 말자.
- 여러 개의 리시버가 있는 상황 등에서는 리시버를 명시적으로 적어 주는 것이 좋다. -> 가독성 향상
- DSL 외부 스코프에 있는 리시버를 명시적으로 적게 강제하고 싶다면, DslMarker 메타 어노테이션을 사용한다.
