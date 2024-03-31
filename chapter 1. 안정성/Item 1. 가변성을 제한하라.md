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
위 코드의 BankAccount에는 계좌에 돈이 얼마나 있는지 나나태는 상태가 있다.
위 와 같이 상태를 나타내는 것은 양날의 검이다 -> 유용하지만 상태를 적절히 관리하는 것은 생각보다 어려움

1. 프로그램을 이해하고 추적하기 힘들어진다.
2. 가변성이 있으면, 코드의 실행을 추론하기 어렵다.
3. 멀티스레드 프로그램일 때는 적절한 동기화가 필요하다.
4. 모든상태를 테스트해야 하므로 테스트하기 어렵다.
5. 상태 변경이 일어날 때, 변경을 다른 부분에 알려야 하는 경우가 있다.

즉, 변할 수 있는 지점은 줄일수록 좋다.

## 코틀린에서 가변성 제한하기

코틀린은 가변성을 제한할 수 있게 설계 -> immutable 객체를 만들거나, 프로퍼티를 변경할 수 없게 막는것이 굉장히 쉽다.
- 읽기 전용 프로퍼티(val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분
- 데이터 클래스의 copy


