# 아이템 20. 일반적인 알고리즘을 반복해서 구현하지 말라

많은 개발자는 같은 알고리즘을 여러번 반복해 구현한다. 예로 복잡하거나 간단한 알고리즘이 있을 수 있다.
```kotlin
val percent = when {
  numberFromUser > 100 -> 100
  numberFromUser < 0 -> 0
  else -> numberFromUser
}
```
이 알고리즘은 사실 stdlib의 coerceIn 확장 함수로 이미 존재한다. 따로 구현하지 않아도 된다.
```kotlin
val percent = numberFromUser.coerceIn(0, 100)
```
이렇게 이미 있는 것을 활용하면, 코드가 짧아지는 것 외에 아래 장점이 있다.
- 코드 작성 속도가 빨라진다. 알고리즘을 만들지 않아도 되기 때문에
- 구현을 따로 읽지 않아도, 무엇을 하는지 확실하게 알 수 있다.
- 직접 구현할 때 발생할 수 있는 실수를 줄일 수 있다.
- 제작자들이 한 번만 최적화하면, 이런 함수를 활용하는 모든 곳이 최적화의 혜택을 받을 수 있다.

## 표준 라이브러리 살펴보기
일반적인 알고리즘은 대부분 이미 다른 사람들이 정의해 놓았다. 대표적으로 stdlib가 있다.

```kotlin
override fun saveCallResult(item: SourceResponse) {
	var sourceList = ArrayList<SourceEntity>()
	item.sources.forEach {
		var sourceEntity = SourceEntity()
		sourceEntity.id = it.id
		sourceEntity.category = it.category
		sourceEntity.country = it.country
		sourceEntity.description = it.description
		sourceList.add(sourceEntity)
	}
	db.insertSources(sourcesList)
}
```
앞의 코드에서 forEach를 사용하는 것은 사실 좋지 않다.  이런 코드는 for을 사용하는 것과 아무 차이가 없다.
sourceEntity 를 처리하는 과정이 어설프다. 이는 코틀린에서 작성된 코드는 더이상 볼 수 없는 자바빈(JavaBean) 패턴이기 때문이다.
이런 형태보다 팩토리 메소드를 활용하거나, 기본 생성자를 활용하는 것이 좋다. 그래도 위와 같은 패턴을 써야겠다면 
다음과 같이 apply를 활용해 모든 단일 객체들의 프로퍼티를 암묵적으로 설정하는 것이 좋다.
```kotlin
override fun saveCallResult(item: SourceResponse) {
	val sourceEntries = item.sources.map(::sourceToEntry)
	db.insertSources(sourceEntries)
}

private fun sourceTnEntry(source: Source) = SourceEntity() {
	.apply {
		id = source.id
		category = source.category
		country = source.country
		description = source.description
	}
}
```

## 나만의 유틸리티 구현하기
상황에 따라 표준 라이브러리에 없는 알고리즘이 필요할 수도 있다.
예를 들어 컬렉션에 있는 모든 숫자의 곱을 계산하는 라이브러리가 필요하다면 유틸리티 함수로 정의하는 것이 좋다.
```kotlin
fun Iterable<Int>.product() = 
    fold(1) { acc, i -> acc * i }
```
**여러 번 사용되지 않아도 이렇게 따로 정의하는 것이 좋다.**

동일한 결과를 얻는 함수를 여러 번 만드는 것은 잘못된 일이다.
모든 함수는 테스트되어야 하고, 기억되어야 하며, 유지보수되어야 한다. 따라서 함수를 만들 때는 이러한 비용이 들어갈 수 있다는 것을 전제해야 한다.

많이 사용되는 알고리즘을 추출하는 방법으로는 톱레벨 함수, 프로퍼티 위임, 클래스 등이 있다. 확장 함수는 이런 방법들과 비교해서, 다음과 같은 장점을 갖고 있다.
- 함수는 상태를 유지하지 않으므로, 행위를 나타내기 좋다. 특히 side-effect(상태 변경)가 없는 경우에 더 좋다.
- 톱레벨 함수와 비교해, 확장 함수는 구체적인 타입이 있는 객체에만 사용을 제한할 수 있어 좋다.
- 수정할 객체를 아규먼트로 전달받아 사용하는 것보다 확장 리시버로 사용하는 것이 가독성 측면에서 좋다.
- 확장 함수는 객체에 정의한 함수보다 객체를 사용할 때, 자동 완성 기능 등으로 제안이 이루어지므로 쉽게 찾을 수 있다.

## 정리
- 일반적인 알고리즘을 반복해 만들지 말자. 대부분 stdlib에 정의되어 있다.
- stdlib에 없는 일반적인 알고리즘이 필요하거나, 특정 알고리즘을 반복해 사용하는 경우 프로젝트 내부에 직접 정의해라
- 일반적으로 이런 알고리즘들은 확장 함수로 정의하는 것이 좋다.

-----
# kotlin stdlib 더 알아보기

### 컬렉션(Collection) 관련 함수
List, Set, Map: 다양한 컬렉션 타입을 제공하고, 각 컬렉션에 대한 유용한 확장 함수들이 많습니다.
- map, filter, reduce, forEach, find, groupBy 등

### 문자열(String) 관련 함수
문자열 조작: 문자열을 처리하고 변환하는데 유용한 함수들입니다.
- substring, split, trim, replace, toUpperCase, toLowerCase, contains 등

### 고차 함수(Higher-order Functions)
람다와 함수형 프로그래밍: 함수형 프로그래밍을 지원하는 고차 함수들입니다.
- let, run, apply, also, takeIf, takeUnless 등

### 범위(Range)와 진행(Progression)
범위 연산: 숫자나 문자 등의 범위를 다루는 함수들입니다.
- rangeTo, downTo, step, until 등

### 표준 입출력(Standard I/O)
기본 입출력: 콘솔 입출력 함수들입니다.
- println, readLine 등

### 예외 처리(Exception Handling)
예외 처리: 예외를 다루기 위한 함수들입니다.
- try, catch, finally, throw, runCatching 등

### 기타 유틸리티 함수들
객체 조작 및 기타: 다양한 상황에서 유용한 함수들입니다.
- check, require, lazy, assert, repeat, with, when 등

```kotlin
fun main() {
    // 컬렉션 예시
    val numbers = listOf(1, 2, 3, 4, 5)
    val doubled = numbers.map { it * 2 }
    println(doubled) // [2, 4, 6, 8, 10]

    // 문자열 예시
    val text = "Hello, Kotlin!"
    val words = text.split(" ")
    println(words) // [Hello,, Kotlin!]

    // 고차 함수 예시
    val result = numbers.run {
        filter { it % 2 == 0 }
    }
    println(result) // [2, 4]

    // 범위 예시
    for (i in 1..5) {
        println(i) // 1, 2, 3, 4, 5
    }

    // 표준 입출력 예시
    println("Enter your name:")
    val name = readLine()
    println("Hello, $name!")

    // 예외 처리 예시
    try {
        val x = 10 / 0
    } catch (e: ArithmeticException) {
        println("Caught an exception: ${e.message}")
    }

    // 기타 예시
    repeat(3) {
        println("This is repetition number $it")
    }
}
```

