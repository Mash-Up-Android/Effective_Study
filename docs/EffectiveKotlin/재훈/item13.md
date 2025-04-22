# 13. Unit?을 리턴하지 말라

---

---

# 개요

Boolean과 Unit? 타입은 서로 바꿔서 사용 가능(Boolean은 true/false Unit?은 Unit/null)

일반적으로 Unit?을 사용한다는 것은 다음과 같은 경우임

```kotlin
fun keyIsCorrect(key: String): Boolean = //...

if(!keyIsCorrect(key)) return
```

다음 코드처럼 사용 가능

```kotlin
fun verifyKey(key: String): Unit? = //...

verifyKey(key) ?: return
```

Unit?으로 Boolean을 표현하는 것은 오해의 소지가 있으며, 예측하기 어려운 오류를 만들 수 있음

```kotlin
if(!keyIsCorrect(key)) return   // Boolean

verifyKey(key) ?: return    // Unit?
```

→ Unit?은 오해를 불러 일으키기 쉬우므로 Boolean을 사용하는 형태로 변경하는 것이 좋음

---