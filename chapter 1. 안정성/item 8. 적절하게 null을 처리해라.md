# 아이템8. 적절하게 null을 처리하라

함수가 null을 리턴한다는 것은 함수에 따라 의미가 달라진다.
- String.toIntOrNull()은 String을 Int로 적절하게변환할 수 없을 경우
- Iterable<T>firstOrNull(() -> Boolean)은 주어진 조건에 맞는 요소가 없을 경우
  
즉, null은 최대한 명확한 의미를 갖는 것이 좋다.
nullable 타입은 세 가지 방법으로 처리한다.
- ?., 스마트 캐스팅, Elvis 연산자 등을 활용해 안전하게 처리한다.
- 오류를 throw한다.
- 함수 또는 프로퍼티를 리팩터링해서 nullable 타입이 나오지 않게 바꾼다.
  
## null을 안전하게 처리하기
null을 안전하게 처리하는 방법 중 가장 널리 사용되는 방법
- safe call
  ```kotlin
  printer?.print() 
  ```
- smart casting
  ```kotlin
  if(printer != null) printer.print() 
  ```
- Elvis 연산자
  ```kotlin
  val printName1 = print?.name ?: "Unnamed"
  val printName2 = print?.name ?: return
  val printName3 = print?.name ?: throw Error("Printer must be named")
  ```
- Collection에서 null 처리
  ```kotlin
  val imageUrls: List<String> = imageUrls.orEmpty() // null일 경우 nullable이 아닌 빈 컬렉션 즉, List<String> 리턴
  ```

> ### 방어적 프로그래밍과 공격적 프로그래밍
> 
> 코드의 안전을 위해 모두 필요하다. 둘을 모두 이해하고 적절하게 사용해야한다.
> ### 방어적 프로그래밍
> - 프로덕션 환경에서 발생할 수 있는 많은 것들로부터 프로그램을 방어해 안정성을 높이는 방법을 나타내는 포괄적인 용어
> - 상황을 처리할 수 있는 올바른 방법이 있을 때 굉장히 좋다.
> ### 공격적 프로그래밍
> - 예상치 못한 상황이 발생했을 때, 이런 문제를 개발자에게 알려 수정하게 만드는 것
> - require, check, assert가 공격적 프로그래밍을 위한 도구

## 오류 thorw 하기
- 오류를 강제로 발생시켜 개발자가 알게할 경우 throwo, !!, requireNotNull, checkNotNull 등을 활용하자.
  
## not-null assertion(!!)과 관련된 문제
  - nullable을 처리하기 가장 간단한 방법, 예외가 발생할 때  어떤 설명도 없는 제네릭 에외가 발생하기 때문에 좋지 않은 방법
  - nullability와 관련된 정보가 숨겨져 있어, 굉장히 쉽게 놓칠 수 있다.
  ```kotlin
  fun largestOf(vararg nums: Int): Int = nums.max()!!
  
  largestOf() // NPE
  ```
  
## 의미없는 nullability 피하기
nullability는 적절하게 처리해야 하므로, 추가 비용이 발생한다. 따라서 필요한 경우가 아니라면 자체를 피하는 것이 좋다.
nullability를 피할 때 사용할 수 있는 몇가지 방법
- 클래스에서 nullability에 따라 여러 함수를 만들어 제공.(ex. List의 get과 getOrNull)
- 어떤 값이 클래스 생성 이후 확실하게 설정된다는 보장이 있다면, lateinit 프로퍼티와 notNull 델리게이트 사용
- null 대신 빈 컬렉션 리턴 (ex. List<Int>?와 Set<String?>과 같은 컬렉션을 빈 컬렉션으로 둘 때와 null로 둘 때는 의미가 다르다)
- nullable enum과 None enum 값은 완전히 다른 의미이다. (None enum은 정의에 없으므로 필요한 경우에 사용하는 쪽에서 추가해 활용 가능)

## lateinit 프로퍼티와 notNull 델리게이트
클래스가 클래스 생성 중에 초기화할 수 없는 프로퍼티를 가지는 것은 드문 일은 아니지만 분명 존재하는 일이다.
이런 프로퍼티는 사용 전에 반드시 초기화해 사용해야한다. (ex. Junit의 @BeforeEach)
```kotlin
class UserControllerTest{
  private var dao: UserDao? = null
  private var controller: UserController? = null
  
  @BeforeEach
  fun init() {
    dao = mockk()
    controller = UserController(dao!!)
  }
  
  @Test
  fun test() {
    controller!!.doSomething()
  }
}
```
프로퍼티를 사용할 때 마다 nullable에서 null이 아닌 것으로 타입 변환하는 것은 바람직하지 않다. -> 의미없는 코드
이런 코드에 대한 바람직한 해결책은 나중에 속성을 초기화할 수 있는, lateinit을 사용하는 것이다.
```kotlin
  class UserControllerTest{
  private lateinit var dao: UserDao
  private lateinit var controller: UserController
  
  @BeforeEach
  fun init() {
    dao = mockk()
    controller = UserController(dao)
  }
  
  @Test
  fun test() {
    controller.doSomething()
  }
}
```
물론 lateinit을 사용하는 것도 비용이 발생한다.
만약 초기화 전에 값을 사용하려하면 예외가 발생하지만, 무서워할 필요가 없다.
### lateinit과 nullable의 차이
- !!연산자로 언팩하지 않아도 된다.
- 이후에 어떤 의미를 나타내기 위해 null을 사용하고 싶을 때, nullable로 만들 수 있다.
- 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.
  
### lateinit을 사용할 수 없는 경우
- JVM에서 Int, Long, Double, Boolean과 같은 기본 타입(primitive)과 연결된 타입으로 프로퍼티를 초기화해야하는 경우 -> Delegates.notNull 사용
```kotlin
class DoctorActivity: Activity() {
  private var doctorId: Int by Delegates.notNull()
  private var fromNotification: Boolean by Delegates.notNull()
  
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
    fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
  }
}
```
위 코드처럼 onCreate 때 초기화하는 프로퍼티는 지연 초기화하는 형태로 다음과 같이 프로퍼티 위임을 사용할 수 있다.
```kotlin
class DoctorActivity: Activity() {
  private var doctorId: Int by arg(DOCTOR_ID_ARG)
  private var fromNotification: Boolean by arg(FROM_NOTIFICATION_ARG)
  
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
    fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
  }
}
```
위와 같이 프로퍼티를 위임하면 nullability로 발생하는 여러 가지 문제를 안전하게 처리할 수 있다.
