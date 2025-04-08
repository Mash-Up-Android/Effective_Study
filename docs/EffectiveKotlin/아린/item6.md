# 사용자 정의 오류보다는 표준 오류를 사용하라

require, check, assert 함수로 대부분의 코틀린 오류를 처리할 수 있다

하지만 이 외 예측하지 못한 상황이 있다면?

JSON 형식의 문제가 있다면 JSONParsingException을 발생시킨다.

```kotlin
inline fun <reified T> String.readObject(): T {
		if (incorrectSign) {
				throw JsonParsingException()
		}
		
		return result
}
```

표준 라이브러리에는 이를 나타내는 적절한 오류가 없으므로

이렇게 사용자 정의 오류를 사용한다.

하지만, 가능하면 표준 라이브러리의 오류를 사용하자

왜냐하면 많은 개발자가 이를 알고 있기 때문에 재사용하는 것이 좋다

잘 만들어진 규약의 널리 알려진 요소를 재사용함으로써

다른 사람들이 API를 더 쉽게 이해하는 지름길이다

## 주로 사용되는 예외들

- **IllegalArgumentException**
    - 메서드에 전달된 인수가 예상된 형식이나 범위를 벗어나는 경우
        - 인수의 값이 허용된 범위를 벗어난 경우 (예: 음수 값 전달)
        - 인수의 형식이 올바르지 않은 경우
        - **`null`** 값이 허용되지 않음에도 **`null`**이 전달된 경우
- **IllegalStateException**
    - 메서드가 호출되었을 때 시스템이나 애플리케이션의 상태가 적절하지 않을 경우
        - 큐가 가득 찬 상태에서 추가 요소를 삽입하려는 경우
- **IndexOutOfBoundException**
    - 배열, 리스트, 문자열 등에서 유효하지 않은 인덱스를 사용하려고 할 때 발생
        - 인덱스가 음수이거나 컬렉션 크기보다 큰 경우.
        - 문자열 메서드(**`substring`**, **`charAt`**)에서 잘못된 인덱스를 사용하는 경우
- **ConcurrentModificationException**
    - 컬렉션을 반복(iterate)하는 동안 동시 수정을 금지했는데 동시에 수정하려고 하면 발생
- **UnsupportedOperationException**
    - 사용하려고 했던 메서드가 현재 객체에서는 사용할 수 없을 때 발생
    - 특정 클래스에서 지원되지 않는 작업을 수행하려고 할 때 발생
        - 사용할 수 없는 메서드를 클래스에 없는 것이 좋다
- **NoSuchElementException**
    - 사용하려고 했던 요소가 현재 객체에서 사용할 수 없을 때 발생
    - 내부에 요소가 없는 iterable에 대해 next를 호출할 때
    - **`Iterator.hasNext()`**를 사용하여 다음 요소가 있는지 확인 후 접근하기