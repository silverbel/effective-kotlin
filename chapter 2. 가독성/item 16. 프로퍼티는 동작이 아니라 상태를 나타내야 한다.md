# 아이템 16. 프로퍼티는 동작이 아니라 상태를 나타내야 한다

코틀린의 프로퍼티는 자바의 필드와 비슷해 보인다. 하지만 서로 완전 다른 개념이다.

```kotlin
// kotlin property
var name: String? = null

// java field
String name = null;
```
둘 다 데이터를 저장한다는 점은 같지만, 프로퍼티에는 더 많은 기능이 있다.
일단 기본적으로 프로퍼티는 사용자 정의 getter와 setter를 가질 수 있다.

```kotlin
// 기본 getter와 setter
class Person(var name: String)

fun main() {
  val person = Person("ABC")
  val name = person.name // person.getName()
  person.name = "EFG" // person.setName()
}

// custom getter setter
class Person(name: String){
  var name: String? = null
    get() = field?.toUpperCase()
    set(value) {
      if(!value.isNullOrBlank()) {
        field = value
      }
    }
}
```
위 코드에서 프로퍼티의 데이터를 저장해 두는 backing field에 대한 레퍼런스인 `field`를 확인할 수 있다.
배킹 필드는 세터와 게터의 디폴트 구현에 사용되므로, 따로 만들지 않아도 디폴트로 생성된다. val을 사용해 읽기 전용 프로퍼티를 만들 때는 field가 생성되지 않는다.

프로퍼티를 통한 `getter, setter`접근은 필드와 다를 수 있다.
```kotlin
var date: Date
    get() = Date(millis) // 매번 Date 객체를 생성합니다.
    set(value){
        // 데이터를 millis라는 별도의 프로퍼티로 옮기고 Date 프로퍼티에 저장하지 않는다.
        millis = value.time 
    }
```
프로퍼티는 필드가 필요 없다. 오히려 프로퍼티는 개념적으로 접근자(val의 경우 getter, var의 경우 getter 와 setter)를 나타낸다.
따라서 코틀린은 인터페이스에도 프로퍼티를 정의할 수 있다.
```kotlin
interface Person {
  val name: String // 프로퍼티
}
```
```kotlin
open class SuperComputer {
  open val theAnswer: Long = 42
}

class AppleComputer : SuperComputer() {
  override val theAnswer: Long = 1_800_275_2273
}
```
#

이러한 이유로 코틀린은 프로퍼티를 위임(property delegation) 을 할 수 있다.
프로퍼티 위임은 아이템21에서 자세하게 설명된다.
```kotlin
  //property delegation
  val db: Database by lazy { connectToDb }
```
프로퍼티는 본질적으로 함수이므로, 확장 프로퍼티를 만들 수 있다.
```kotlin
  val Context.preferences: SharedPreferences
      get() = PreferenceManager
          .getDefaultSharedPreferences(this)
          
  val Context.inflater: LayoutInflater
      get() = getSystemService(
          Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
          
  val Context.notificationManager: NotificationManager
      get() = getSystemService(Context.NOTIFICATION_SERVICE)
          as NotificationManager
```
위에서 확인할 수 있는 것 처럼 프로퍼티는 필드가 아니라 접근자를 나타낸다.
프로퍼티를 함수 대신 사용할 수도 있지만, 그렇다고 완전히 대체해 사용하는 것은 좋지 않다.
```kotlin
// 프로퍼티를 함수처럼 사용할 수 있지만 완전히 대체하지 마라
val Tree<Int>.sum: Int
    get() = when(this) {
        is Leaf -> value
        is Node -> left.sum + right.sum
    }
    
// 함수로 구현해라
fun Tree<Int>.sum(): Int = when (this) {
    is Leaf -> value
    is Node -> left.sum() + right.sum()
}
```
프로퍼티에 알고리즘 동작이 들어가는 것은 좋지 않다.
프로퍼티 대신 함수를 사용하는 것이 좋은 경우
-  연산 비용이 높거나 복잡도가 O(1)보다 큰 경우
-  비즈니스 로직을 포함하는 경우
-  결정적이지 않은 경우
-  변환의 경우
-  게터에서 프로퍼티의 상태 변경이 일어나야 하는 경우

#

상태를 추출/설정할 때는 프로퍼티를 사용해야 한다. 특별한 이유가 없다면 함수를 사용하면 안된다.
