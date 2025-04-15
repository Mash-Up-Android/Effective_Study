# use를 사용하여 리소스를 닫아라

AutoCloseable을 상속받는 Closeable 인터페이스를 구현하는 리소스들

- InputStream / OutputStream
- java.sql.Connection
- java.io.Reader(FileReader, BufferedReader, CSSParser)
- java.new.Socket / java.util.Scanner

최종적으로 이러한 리소스에 대한 레퍼런스가 없어질 때 가비지 컬렉터가 이를 처리한다

대신 굉장히 느리고, 그동안 유지 비용이 많이 들어간다

따라서 더 이상 이 리소스가 필요하지 않다면 명시적으로 close 메서드를 호출해준다

try-finally 블록을 사용해 처리하는 것이 전통적이지만, 이러한 코드는 굉장히 복잡하고 좋지 않다

리소스를 닫을 때 예외가 발생한다면 따로 예외 처리를 하지 않기 때문이다

try, finally 둘 중 하나에서만 오류가 전파되기도 한다

두 상황에 전파하는 것을 간단히 구현하기 위해서, `use` 라는 함수를 사용한다

```kotlin
val reader = BufferedReader(FileReader(path))
reader.use {
		return reader.lineSequence().sumBy { it.length }
}
```

람다 매개변수로 줄일 수도 있다

```kotlin
BufferedReader(FileReader(path)).use { reader ->
		return reader.lineSequence().sumBy { it.length }
}
```

파일을 한 줄씩 처리할 때 유용한 useLines 함수

이렇게 처리하면 메모리에서 파일 내용을 한 줄씩 유지하므로 대용량 파일을 적절하게 처리할 수 있다

```kotlin
File(path).useLines { lines ->
		return lines.sumBy { it.length }
}
```

하지만, 파일의 줄을 단 한번만 사용할 수 있기 때문에 여러 번 사용하려면 여러 번 열어야 한다

> use를 사용하여 Closeable/AutoCloseable 구현 객체를 쉽고 안전하게 처리하자
useLines을 사용해 파일을 한 줄씩 효과적으로 읽어 들일 수 있다
>