# 23. 타입 파라미터의 섀도잉을 피하라

---

---

# 개요

섀도잉 **:** 프로퍼티와 파라미터가 같은 이름인 경우, 지역 파라미터가 외부 스코프에 있는 프로퍼티를 가림

쉽게 찾을 수 있어서 경고를 발생시키지 않음

```kotlin
class Forest(val name: String) {
    fun addTree(name : String) {
    	// ...
    }
}
```

섀도잉 현상은 클래스 타입 파라미터와 함수 타입 파라미터 사이에서도 발생

개발자가 제네릭을 제대로 이해하지 못할 때, 이와 관련된 다양한 문제들이 발생

심각한 문제가 될 수 있고, 문제를 찾아내기 힘듦

이렇게 코드를 작성하면 `Forest`와 `addTree`의 타입 파라미터가 독립적으로 동작함

```kotlin
interface Tree
class Birch: Tree
class Spruce: Tree

class Forest<T: Tree> { 
    fun <T: Tree> addTree(tree: T) {
        // ... 
    }
}

val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce())
```

이러한 상황을 의도하는 경우는 거의  없을 것이고, 코드만 봐서는 둘이 독립적으로 동작한다는 것을 빠르게 알아내기 힘듦

따라서 `addTree`가 클래스 타입 파라미터인 `T`를 사용하는 것이 좋음

```kotlin
class Forest<T: Tree> { 
    fun addTree(tree: T) { 
        // ...
    }
}

// Usage
val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce())  // ERROR, type mismatch
```

만약 독립적인 타입 파라미터를 의도했다면, 이름을 아예 다르게 다는 것이 좋음

타입 파라미터를 사용해서 다른 타입 파라미터에 제한주는 것도 가능

```kotlin
class Forest<T: Tree> { 
    fun <ST: T>addTree(tree: ST) {
    	// ...
    }
}
```

---

# 정리

타입 파라미터 섀도잉이 발생한 코드는 이해하기 어려우므로 코드를 주의해서 살펴보고 최대한 피하라

---