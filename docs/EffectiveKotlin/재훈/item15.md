# 15. 리시버를 명시적으로 참조하라

---

---

# 개요

뭔가를 더 설명하기 위해 명시적으로 긴 코드를 사용하는 경우가 있음

(ex. 함수와 프로퍼티를 지역 변수 or 탑레벨 변수가 아닌 다른 리시버로부터 가져올 때)

예를 들면, 클래스의 메서드라는 것을 나타내기 위한 **`this`**가 있음

```kotlin
class User: Person() { 
    private var beersDrunk: Int = 0
    
    fun drinkBeers(num: Int) {
    	// ...
        this.beersDrunk += num
        // ...
    }
}
```

비슷하게 확장 리시버(확장 메서드의 `this`)를 명시적으로 참조하게 할 수도 있음

- 리시버를 명시적으로 표시하지 않은 경우
    
    ```kotlin
    fun <T : Comparable<T>> List<T>.quickSort(): List<T> { 
        if (size < 2) return this
        val pivot = first()
        val (smaller, bigger) = drop(1).partition { it < pivot }
        return smaller.quickSort() + pivot + bigger.quickSort()
    ```
    

- 리시버를 명시적으로 표시한 경우
    
    ```kotlin
    fun <T : Comparable<T>> List<T>.quickSort(): List<T> {  
        if (this.size < 2) return this
        val pivot = this.first()
        val (smaller, bigger) = this.drop(1).partition { it < pivot }
        return smaller.quickSort() + pivot + bigger.quickSort()
    ```
    

⇒ 두 함수 사용에는 차이가 없음

---

# 여러 개의 리시버

스코프 내부에 둘 이상의 리시버가 있는 경우, 리시버를 명시적으로 나타내는게 좋음

- `apply`, `with`, `run`함수를 사용할 때 예시
    
    ```kotlin
    class Node(val name: String) {
        fun makeChild(childName: String) = 
            create("$name.$childName") 
                .apply{ print("Created ${name}") }
    
        fun create(name: String): Node? = Node(name)
    }
    
    fun main() {
        val node = Node("parent")
        node.makeChild("child")
    }
    ```
    
    ⇒ 출력 결과 ‘Created parent.child’가 아닌, ‘Created parent’가 나옴
    

- 명시적으로 리시버를 붙인 경우
    
    ```kotlin
    class Node(val name: String) { 
        fun makeChild(childName: String) = create("$name.$childName")
            .apply{ print("Created ${this.name}") } // 컴파일 오류
    
        fun create(name: String): Node? = Node(name)
    }
    ```
    
    ⇒ `apply`함수 내부에서 `this`의 타입이 `Node?`라서 직접 사용할 수 없음
    

- 따라서 사용하려면 unpack(언팩)하고 호출해야 함
    
    ```kotlin
    class Node(val name: String) { 
        fun makeChild(childName: String) = create("$name.$childName")
            .apply{ print("Created ${this?.name}") }
    
        fun create(name: String): Node? = Node(name)
    }
    ```
    

사실 이는 `apply`의 잘못된 사용 예시로, 만약 `also` 함수와 파라미터 `name`을 사용했다면 문제가 없음

`also`를 사용하면, 이전과 마찬가지로 명시적으로 리시버를 지정하게 됨

⇒ 일반적으로 `also` 또는 `let`을 사용하는 것이 nullable값을 처리할 때 훨씬 좋음

```kotlin
class Node(val name: String) {
    fun makeChild(childName: String) = create("$name.$childName")
        .also{ print("Created ${it?.name}") }

    fun create(name: String): Node? = Node(name)
}
```

리시버가 명확하지 않다면, 명시적으로 리시버를 적어서 이를 명확하게 해줘야함

- 레이블 없이 리시버를 사용하면, 가장 가까운 리시버를 의미함
- 외부에 있는 리시버를 사용하려면, 레이블을 사용해야 함

둘 모두 사용하는 예시

```kotlin
class Node(val name: String) { 
    fun makeChild(childName: String) = create("$name.$childName").apply{ 
        print("Created ${this?.name} in " + "${this@Node.name}") 
    }
  
    fun create(name: String): Node? = Node(name)
}
```

리시버를 명시적으로 작성하면 어떤 리시버를 활용하는지 의미가 훨씬 명확해짐

→ 코드 안정성 + 가독성 향상

---

# DSL 마커

코틀린 DSL을 사용할 때는 여러 리시버를 가진 요소들이 중첩되더라도, 리시버를 명시적으로 붙이지 않음

⇒ DSL은 원래 그렇게 사용하도록 설계되었기 때문

그런데 DSL에서는 외부의 함수를 사용하는 것이 위험한 경우가 있음

DSL 마커는 가장 가까운 리시버만을 사용하게 하거나, 명시적으로 외부 리시버를 사용하지 못하게 할 때 활용할 수 있는 굉장히 중요한 메커니즘

→ 설계에 따라 사용 여부를 결정

---

# 정리

- 짧게 적을 수 있단 이유만으로 리시버를 제거하면 안됨
- 여러 개의 리시버가 있으면 리시버를 명시적으로 적어 주는 것이 좋음
    
    ⇒ 리시버를 명시적으로 지정하면, 어떤 리시버의 함수인지를 명확하게 알 수 있어서 가독성 향상
    
- DSL에서 외부 스코프에 있는 리시버를 명시적으로 적게 강제하려면, `DslMaker` 메타 어노테이션 사용

---

# 생각

- 리시버를 명시적으로 사용하는 게 코드 안정성과 가독성 향상에 좋다고 하는데 공감됨
- 평소에 은연 중에 헷갈리지 않으려고 명시적으로 써왔던 것 같음 but 의식하면서 쓰는 습관이 필요한듯