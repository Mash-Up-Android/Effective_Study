# 6. 사용자 정의 오류보다는 표준 오류를 사용하라

---

---

# 개요

require, check, assert 함수를 사용하면 대부분의 코틀린 오류를 처리 가능

하지만 이외에도 예측하지 못한 상황을 나타내야 하는 경우가 있음

기본적으로 입력된 JSON 파일의 형식에 문제가 있다면, `JsonParsingException` 등을 발생시키는 것이 좋음

```kotlin
inline fun <reified T> String.readOjbect(): T {
    // ...
    if (incorrectSign) {
    	throw JsonParsingException()
    }
    // ...
    return result
}
```

표준 라이브러리에는 이를 나타내는 적절한 오류가 없으므로, 사용자 정의 오류를 사용했음

하지만 가능하다면, 직접 오류를 정의하는 것보다는 최대한 표준 라이브러리의 오류를 사용하는게 좋음

표준 라이브러리의 오류는 많은 개발자들이 알고 있으므로, 이를 재사용하는게 좋음

잘만들어진 규약을 가진 널리 알려진 요소를 재사용하면, 다른 사람들이 API를 더 쉽게 배우고 이해 가능

일반적으로 사용되는 예외

- `IllegalArgumentException`, `IllegalStateException` : require와 check를 사용해 throw 할 수 있는 예외
- `IndexOutOfBoundsException` : 인덱스 파라미터의 값이 범위를 벗어났다는 것을 나타냄
    
    ⇒ 일반적으로 컬렉션 또는 배열과 함께 사용 (ex. `ArrayList.get(Int)`를 사용할 때 throw됨)
    
- `ConcurrentModificationException` : 동시 수정(concurrent modification)을 금지했는데, 발생해 버렸다는 것을 나타냄
    - 동시수정 : iterator가 컬렉션을 반복하는데 컬렉션 요소에 대한 추가 및 삭제가 발생해서 생기는 예외 (RuntimeException에 속함)
- `UnsupportedOperationException` : 사용하려 했던 메서드가 현재 객체에서는 사용할 수 없음을 나타냄
    
    ⇒ 기본적으로 사용할 수 없는 메서드는 클래스에 없는 것이 좋음 (ISP 원칙 위반)
    
- `NoSuchElementException` : 사용자가 사용하려고 했던 요소가 존재하지 않음을 나타냄
    
    ⇒ (ex. 내부에 요소가 없는 Iterable에 대해 next를 호출할 때 발생)
    

---