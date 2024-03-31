# 정리
- var보다 val 사용하는 것이 좋음
- mutable 프로퍼티 보다는 보다 immutable 프로퍼티 사용하는 것이 더 좋음
- mutable 객체와 클래스보단 immutable 객체와 클래스 사용하는 것이 더 좋음
- 변경이 필요한 대상을 만들어야 한다면, immutable 데이터 클래스로 만들고 copy 활용하는 것이 좋음
- collection에 상태를 저장해야 한다면, mutable 컬렉션보다는 읽기 전용 컬렉션을 사용하는 것이 좋음
- 변이 지점을 적절하게 설계하고, 불필요한 변이 지점은 만들지 않는 것이 좋음
- mutable 객체를 외부에 노출하지 않는 것이 좋음
- 예외. 가끔 효율성 때문에 immutable 객체보다 mutable 객체가 좋음(3부 추가 설명)
- immutable 객체를 사용할 때는 언제나 멀티스레드 때에 더 많은 주의
___
# 아이템 1. 가변성을 제한하라

```kotlin
class BankAccount {
	var balance = 0.0
	private set
	fun deposit(depositAmount: Double) {
		balance += depositAmount
	}
	
	@Throws(InsufficientFunds::class)
	fun withdraw(withdrawAmount: Double) {
		if (balance < withdrawAmount) {
			throw InsufficientFunds()
		}
		balance -= withdrawAmount
	}
}
class InsufficientFunds : Exception()
val account = BankAccount()
println(account.balance) // 0.0
account.deposit(100.0)
println(account.balance) // 100.0
account.deposit(50.0)
println(account.balance) // 50.0

```
위 코드의 BankAccount에는 계좌에 돈이 얼마나 있는지 나타내는 상태가 있다.\
위 와 같이 상태를 나타내는 것은 양날의 검이다. -> 유용하지만 상태를 적절히 관리하는 것은 생각보다 어려움

:question: 무엇이 어려운가
1. 프로그램을 이해하고 추적하기 힘들어진다.
2. 가변성이 있으면, 코드의 실행을 추론하기 어렵다.
3. 멀티스레드 프로그램일 때는 적절한 동기화가 필요하다.
4. 모든상태를 테스트해야 하므로 테스트하기 어렵다.
5. 상태 변경이 일어날 때, 변경을 다른 부분에 알려야 하는 경우가 있다.

## 즉, 변할 수 있는 지점은 줄일수록 좋다.

## 코틀린에서 가변성 제한하기

코틀린은 가변성을 제한할 수 있게 설계 -> immutable 객체를 만들거나, 프로퍼티를 변경할 수 없게 막는것이 굉장히 쉽다.
- 읽기 전용 프로퍼티(val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분
- 데이터 클래스의 copy

### 1. 읽기 전용 프로퍼티(val)
> val : getter 생성\
> var : getter/setter 생성

- val을 사용해 읽기전용 프로퍼티를 만들 수 있다.\
(읽고 쓸 수 있는 프로퍼티는 var로 생성)

```kotlin
val a = 10
a = 20 // 오류
```
- 읽기 전용 프로퍼티가 mutable 객체를 담고 있다면, 내부적으로 변할 수 있음
```kotlin
val list = mutableListOf(1,2,3)
list.add(4)

print(list) // [1, 2, 3, 4]
```
- 읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로 정의할 수 있음.\
val 프로퍼티가 var 프로퍼티를 사용하면 var 프로퍼티가 변할 때 변할 수 있음
```kotlin
var name: String = "silverbell"
var surname: String= "Eun"
val fullName
	get() = "$name $surname"
fun main() {
  println(fullName) // silverbell Eun
  name = "Bang"
  println(fullName) // silverbell Bang
}
```
- var은 게터와 세터를 모두 제공하지만, val은 변경이 불가능하므로 게터만 제공한다. 그래서 val을 var로 오버라이드할 수 있다.
```kotlin
interface Element{
  val active: Boolean
}

class ActualElement: Element{
  override var active: Boolean = false
}
```
- val은 읽기 전용이지만, immutable을 의미하는 것은 아니다. 이는 게터 혹 delegate로 정의할 수 있다.
- 완전 변경할 필요가 없다면, final 프로퍼티를 사용하는 것이 좋다.
- val은 정의 옆에 상태가 바로 적히므로, 코드의 실행을 예측하는 것이 훨씬 간단하고 스마트 캐스트 등 추가 기능 활용 가능.
```kotlin
var name: String? = "silverbell"
var surname: String = "Eun"

val fullName: String?
	get() = name?.let {"$it $surname"}
  
val fullName2: String? = name?.let {"$it $surname"}

fun main() {
	if (fullName != null) {
		println(funllName.length) // 오류
	}
	if (fullName2 != null) {
		println(fullName2.length) // silverbell Eun
	}
}
```
위 코드에서 fullName은 게터로 정의했으므로 스마트 캐스트할 수 없음. 게터를 활용하므로, 값을 사용하는 시점의 name에 따라 다른 결과가 나올 수 있기 때문
fullName2처럼 지역 변수가 아닌 프로퍼티(Non-local property)가 final이고, 사용자 정의 게터를 갖지 않을 경우 스마트캐스트 할 수 있다

### 2. 가변 컬렉션과 읽기 전용 컬렉션 구분하기
<img src="https://kotlinlang.org/docs/images/collections-diagram.png" width = 300 height = 300/>

- 인터페이스 읽기 전용: Iterable, Collection, Set, List
- 읽고 쓸 수 있는 컬렉션: MutableIterable, MutableCollection, MutableSet, MutableList
:x: 컬렉션을 읽기전용으로 리턴하면 읽기 전용으로만 사용해야하는데 이때 컬렉션 다운캐스팅해서 그냥 깨버리면\
추상화를 무시하는 행위. 이는 안전하지 않고, 예측하지 못한 결과 초래

```kotlin
val list = listOf(1,2,3)
if(list is MutableList){
  list.add(4)
} // 이렇게 하면 뒤통수를 맞는다.

val mutableList = list.toMutableList() // copy를 통해서 새로운 mutable 객체를 만들어 기존 list 객체는 안전하다
mutableList.add(4) // 이렇게 해야한다.
```
읽기 전용 컬렉션을 mutable 컬렉션으로 다운캐스팅하면 안됨. 읽기 전용에서 mutable로 변경해야 한다면, 데이터클래스의 copy를 통해 새로운 mutable 컬렉션을 만들어야한다

### 3. 데이터 클래스의 copy
- immutable 객체 사용의 장점
  1. 한 번 정의된 상태 유지 -> 코드 이해 쉬움
  2. immutable 객체는 공유했을 때도 충돌 x, 안전한 병렬 처리
  3. immutable 객체에 대한 참조는 변경x, 쉽게 캐시 가능
  4. immutable 객체는 방어적 복사본을 만들 필요 x, 객체 복사시 깊은 복사할 필요 x
  5. immutable 객체는 다른 객체를 만들 때 활용하기 좋음, 실행 예측 쉬움
  6. 'set' or 'map'의 key로 사용 가능. mutable 객체는 사용 불가 -> key가 변경되면  oh.. 끔찍해
```kotlin
val names: SortedSet<FullName> = TreeSet()
val person = FullName("AAA", "AAA")
names.add(person)
names.add(FullName("BBB", "BBB")
names.add(FullName("CCC", "CCC")
print(s) // [AAA AAA, BBB BBB, CCC CCC]
print(person in names) // true
person.name = "ZZZ"
print(names) // [ZZZ AAA, BBB BBB, CCC CCC]
print(person in names) // false
```
- Set 내부에 Person 객체가 있어도 false를 리턴하는 이유는 객체를 변경하여 찾을 수 없기 때문
- mutable 객체는 예측하기 어려우며 위험하다는 단점이 있음. 반면 immutable 객체는 변경할 수 없다는 단점이 있음.\
따라서 immutable 객체는 자신의 일부를 수정한 새로운 객체를 만드는 메서드를 가져야함.
```kotlin
class User (val name:String, val surname: String) {
	fun withSurname(surname: String) = User(name, usrname)
}
var user = User("AAA", "BBB")
user = user.withSurname("CCC")
print(user) // User(name="AAA", surname="CCC")
```
- 모든 프로퍼티를 대상으로 이런 함수를 하나하나 만드는 것은 귀찮기 때문에 data한정자를 사용하면 됨.\
data type은 copy 메소드를 만들어 줌
```kotlin
data class User (val name:String, val surname: String)
var user = User("AAA", "BBB")
user = user.copy("CCC")
print(user) // User(name="AAA", surname="CCC")
```

## 다른 종류의 변경 가능 지점
변경할 수 있는 리스트를 만들 때
1. mutable 컬렉션 만들기
2. var로 읽고 쓸 수 있는 프로퍼티 만들기:o: // 이게 더 좋은 방법이다

위 두가지 모두 변경 가능. but 방법이 다름
```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()

list1.add(1)
list2 = list2 + 1

list1 += 1 // list1.plusAssign(1)로 변경
list2 += 1 // list2 = list2.plus(1)로 변경
```
- list1 의 변경은 구체적인 리스트 구현 내부에 변경 가능 지점이 있어, 멀티스레드 처리가 이뤄질 경우, 내부적으로 적절한 동기화가 되어있는지 알 수 없어 위험
- list2의 변경은 프로퍼티 자체가 변경가능 지점. 따라서 멀티스레드 처리 안정성이 더 좋음

```kotlin
var list = listOf<Int>() 
	for (i in 1..1000) {
		thread {
		list = list + i
	}
}
Thread.sleep(1000)
print(list.size) // 1~1000을 더한 값이 되지 않는다. 실행할 때마다 다른 숫자가 나온다
```
- mutable 리스트 대신 mutable 프로퍼티를 사용하는 형태는 사용자 정의 세터를 활용해 변경 추적 가능

```kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new -> 
	println("Names changed from $old to $new")
}
names += "Fabio" // Names changed from [] to [Fabio]
names += "Bill" // Names changed from [Fabio] to [Fabio, Bill]
```

- 프로퍼티와 컬렉션을 모두 변경 가능한 지점으로 만들면 뒤통수를 맞는다.:x:
```kotlin
var list3 = mutableListOf<Int>() 
```
위와 같이 코드를 작성할 경우 변경가능한 두 지점 모두 동기화를 구현해야 한다. 또한 모호성이 발생해 += 사용 불가

## 변경 가능 지점 노출하지 말기
mutable 객체를 외부에 노출하는 것은 위험하다

```kotlin
data class User(val name: String) 
class UserRepository {
	private val storedUsers: MuatableMap<Int, String> = mutableMapOf()
	fun loadAll(): MutableMap<Int,String> {
		return storedUsers
	}
	//...
}

val userRepository = UserRepository()
val storedUsers = userRepository.loadAll() 
storedUsers[4] = "AAA"
//...

print(userRepository.loadAll()) // {4=AAA}
```
위와 같은 코드는 위험하다.
loadAll을 사용해 private 상태인 UserRepository를 수정할 수 있음
1. 방어적 복제(리턴되는 mutable 객체를 복제)
```kotlin
class UserHolder {
	private val user: MutableUser()
	fun get(): MutableUser {
		return user.copy()
	}
	//...
}
```
2. 가변성 제한(mutable 객체를 immutable 객체로 up cast)
```kotlin
class UserRepository {
	private val storedUsers: MuatableMap<Int, String> = mutableMapOf()
	fun loadAll(): Map<Int,String> {	// 읽기전용(immutable) Map으로 up cast!
		return storedUsers
	}
	//...
}
```
