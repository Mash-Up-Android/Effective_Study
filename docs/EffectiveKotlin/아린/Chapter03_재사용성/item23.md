# **타입 파라미터의 섀도잉을 피하라**

## **섀도잉이란?**

내부 스코프(함수, 메서드)의 변수나 파라미터가 외부 스코프(클래스, 상위 함수)의 변수나 파라미터와 이름이 같을 때,
내부 선언이 외부 선언을 가리는 현상

타입 파라미터의 섀도잉(shadowing)은 클래스의 타입 파라미터와 함수의 타입 파라미터가 같은 이름을 가질 때 발생한다. 이로 인해 코드의 의도가 불분명해지고, 실수로 인한 버그가 발생할 수 있으므로 반드시 피해야 한다

IDE나 컴파일러가 경고를 주지 않기 때문에, 코드의 동작을 오해하거나 버그가 발생할 위험이 높다

```kotlin
interface Tree
class Birch : Tree
class Spruce : Tree

class Forest<T : Tree> {
    fun <T : Tree> addTree(tree: T) {
        // ...
    }
}

val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce())

```

위 코드처럼 프로퍼티, 파라미터가 같은 이름을 가질 수도 있다.

이렇게 지역 파라미터가 외부 스코프 프로퍼티를 가리는 것을 섀도잉이라고 한다

클래스랑 함수 둘 다 타입 파라미터 이름이 T, 두 개가 같은 이름 T를 쓰고 있음

함수 addTree 안에서 쓰는 T는 클래스 Forest의 T를 덮어 쓰고, 클래스의 타입 T는 함수 안에선 완전히 무시된다

## 1. 함수에 새 타입 파라미터를 작성하지 않는다

```kotlin
class Forest<T : Tree> {
    fun addTree(tree: T) {
        // T는 클래스의 T 그대로 사용
    }
}

```

Forest<Birch>에는 Birch만 들어갈 수 있음

## 2. 이름을 다르게 작성한다

```kotlin
class Forest<T : Tree> {
    fun <S : Tree> addTree(tree: S) {
        // 여기서 U는 독립적인 Tree 타입
    }
}

```

Forest<Birch>에도 Spruce나 다른 Tree도 넣을 수 있게 설계 가능