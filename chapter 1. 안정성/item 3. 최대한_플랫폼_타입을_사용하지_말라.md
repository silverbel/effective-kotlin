# 정리
- 다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 플랫폼 타입이라고 부른다.
- 이러한 플랫폼 타입을 사용하는 코드는 해당 부분만 위험할 뿐만 아니라, 이를 활용하는 곳까지 영향을 줄 수 있는 위험한 코드이니 제거 해리.
- 연결된 java 생성자, 메서드, 필드에 nullable 여부를 지정하는 어노테이션을 활용하는 것도 좋다.
----

# 아이템3. 최대한 플랫폼 타입을 사용하지 말라

> 널 안정성(null-safety)은 코틀린의 주요 기능 중 하나이다.

- @Nullable 어노테이션이 붙어 있다면 -> String?으로 변경하면 된다.
- @Notnull 어노테이션이 붙어있다면   -> String으로 변경하면 된다.

자바에서 모든 것이 nullable일 수 있으므로 최대한 안전하게 접근한다면, 이를 nullable로 가정하고 다루어야 한다. 하지만 어떤 메서드는 null을 리턴하지 않을 것이 확실할 수 있다. 이런 경우 not-null 단정을 나타내는 !!를 붙인다.

```kotlin
// 자바
public class UserRepo{
	public List<User> getUsers() {
		//...
	}
}

// 코틀린
val users: List<User> = UserRepo().users!!.fillterNotNull()

// 만약 List<List<User>>를 리턴한다면
val users: List<List<User>> = UserRepo().groupedUsers!!.map { filterNotNull() } 
```

자바를 코틀린과 함께 사용할 때, 자바 코드를 직접 조작할 수 있다면, 가능한 @Nullabler과 @NotNull 어노테이션을 붙여서 사용해라.

```kotlin
// 자바 
import org.jetbrains.annotations.NotNull;

public class UserRepo {
	public @NotNull User getUser() {
		//...
	}
}
```


```kotlin
// 자바
public class JavaClass {
	public String getValue() {
		return null;
	}
}

// 코틀린
fun statedType() {
    val value: String = JavaClass().value // NPE
    // ...
    println(value.length)
}

fun platformType() {
    val value = JavaClass().value
    // ...
    println(value.length) // NPE
}
```

- 두 가지 모두 NPE가 발생한다.
- statedType에서는 자바에서 값을 가져오는 위치에서 NPE가 발생한다. 이 위치에서 오류가 발생하면, null이 아니라고 예상을 했지만 null이 나온다는 것을 굉장히 쉽게 알 수 있다. (더 좋은방법)
- platformType에서는 값을 활용할 때 NPE가 발생한다. 객체를 사용한다고 해서 NPE가 발생될 거라고 생각하지는 않으므로, 오류를 찾는데 굉장히 오랜 시간이 걸린다.

인터페이스에서 다음과 같이 플랫폼 타입을 사용했다고 해보자

```kotlin
interface UserRepo {
	fun getUserName() = JavaClass().value
}
```

이러한 경우 메서드의 inferred 타입(추론된 타입)이 플랫폼 타입이다. 
- 어떤 사람이 이를 활용해 nullable을 리턴했는데, 사용하는 사람이 nullable이 아닐 거라고 받아드리면 문제다 (즉, 쓰지마라...)

```kotlin
class RepoImpl: UserRepo {
	override fun getUserName(): String? {
		return null
	}
}

fun main() {
	val repo: UserRepo = RepoImpl()
	val text: String = repo.getUserName() // 런타임 NPE
	print("User name length is ${text.length}")
}
```
