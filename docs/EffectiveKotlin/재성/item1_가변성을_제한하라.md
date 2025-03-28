## 아이템 1. 가변성을 제한하라
- 객체가 상태를 갖게 되면 관리가 힘들어진다.
- 가변성을 갖는 상태에는 문제가 많을 수 있다. (var, var mutableList)
- 상태가 가변성을 띄게되면 상태를 사용하는 시점에 따라 값이 다를수가 있고, 동일한 값이 유지된다고 보기 힘들다.
    - 요건 멀티스레드에서의 동기화와 관련 있음
    - 동기화 잘 할 수 있으면 mutable하게 사용해보던가~

- 결국 가변성을 적당히 잘 제한하는게 중요해보이는데 코틀린에는 Immutable Collection이 있음
- 프로퍼티를 읽기전용으로 만들던가 Immuable Collection을 사용하던가 data class copy를 사용하던가
    - 그럼 val mutableList는 불변인가? X
        - val 은 읽기 전용인거고 mutableList는 값이 변할수 있기 때문에 요걸 주의해야 한다.
        - 리스트 자체에 값을 담는거는 가능하지만 새로운 리스트로 대체하는건 안되는 뜻.
- 프로퍼티는 캡슐화 되어있어서 getter, setter를 가지게 되는데 val은 읽기전용이라 getter만, var은 읽고쓸수 있어서 getter, setter를 가짐
    - 그래서 val을 var로 오버라이드 할 수 있고, 둘다갖는 var보다 val을 많이 씀

- mutableCollection은 Collection을 상속하여 변경을 위한 메서드가 추가된 것
- 읽기 전용으로 반환하는 경우 읽기 전용으로 사용해야 한다.

    ```kotlin
    val list = listOf(1, 2, 3)
    if (list is MutableList) { list.add(4) }  // 쓰지마!
    
    val mutableList = list.toMutableList()  // 이렇게 써라~
    mutableList.add(4)  
    ```

- 데이터클래스는 immutable의 특성을 갖도록 만들고 변경시에는 copy해서 쓰는게 좋다
- var list3 = mutableList<Int>() 는 하지마라

```kotlin
val list = mutableList<Int>()
var list2: List<Int> = listOf()

list += 1  // plusAsign으로 동작 -> 기존 리스트에 변경
list2 += 1  // plus로 동작 -> 새로운리스트로 반환
```

- 변경가능한 부분을 객체 외부로 노출하지 말기