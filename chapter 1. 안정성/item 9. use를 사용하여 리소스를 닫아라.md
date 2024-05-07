## 정리
- use를 사용하면 Closeable/AutoCloseable을 구현한 객체를 쉽고 안전하게 처리 가능
- 파일을 처리할 때는 한 줄씩 읽는 useLines 사용하는 것이 좋음
---

# 아이템 9. user를 사용하여 리소스를 닫아라
> close 메소드를 사용해 명시적으로 닫아야하는 리소스들의 예시
> - InputStream, OutputStream
> - java.sql.Connection
> - java.io.Reader(FileReader, BufferedReader, CSSParser)
> - java.new.Socket, java.util.Scanner

전통적으로 이런 리소스들은 아래와 같이 try-finally 블록을 사용함
```kotlin
  val reader = BufferedReader(FileReader(path))
  try {
    return reader.lineSequence().sumBy {it.lentgh}
  } finally {
    reader.close()
  }
```
  - 리소스를 닫을 때 예외가 발생할 수 있는데, 이런 예외를 따로 처리하지 않기 때문에 좋지 않다.
  - try 블록과 finally 블록 내부에서 오류가 발생하면, 둘 중 하나만 전파된다.(둘다 되면 좋을 것)
  - 아래와 같이 리팩토링할 수 있다. 

## use 함수
```kotlin
  BufferedReader(FileReader(path)).user { reader ->
    return reader.lineSequence().sumBy { it.length }
  }
```
- use 함수 내부
    - try와 finally 블록 각각에서 오류 핸들링
    - 직접 구현하려면 코드가 매우 복잡하고 길어짐
```kotlin
@InlineOnly
@RequireKotlin("1.2", versionKind = RequireKotlinVersionKind.COMPILER_VERSION, message = "Requires newer compiler version to be inlined correctly.")
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    var exception: Throwable? = null
    try {
        return block(this)
    } catch (e: Throwable) {
        exception = e
        throw e
    } finally {
        when {
            apiVersionIsAtLeast(1, 1, 0) -> this.closeFinally(exception)
            this == null -> {}
            exception == null -> close()
            else ->
                try {
                    close()
                } catch (closeException: Throwable) {
                    // cause.addSuppressed(closeException) // ignored here
                }
        }
    }
}
```

## useLines 함수
```kotlin
  File(path).useLines { lines ->
    return lines.sumBy { it.length }
  }
```
- 메모리에 파일의 내용을 한 줄 씩만 유지해 대용량 파일 적절하게 처리 가능 (장점이자 단점)
