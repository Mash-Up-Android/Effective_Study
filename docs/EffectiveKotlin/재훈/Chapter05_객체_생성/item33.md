# 33. 생성자 대신 팩토리 함수를 사용하라

---

---

# 개요

클라이언트가 클래스의 인스턴스를 만들게 하는 가장 일반적인 방법은 기본 생성자(Primary constructor)를 사용하는 방법임

```kotlin
class MyLinkedList<T>{
    val head : T
    val tail : MyLinkedList<T>?
}

val list = MyLinkedList(1, MyLinkedList(2, null))
```

하지만 생성자가 객체를 만들수 있는 유일한 방법은 아님

디자인 패턴으로 다양한 생성 패턴이 있기 때문에, 일반적으로 이러한 생성 패턴은 객체를 생성자로 직접 생성하지 않고, 별도의 함수를 통해 생성함

예를 들어 다음 톱레벨 함수는 `MyLinkedList` 클래스의 인스턴스를 만들어서 제공해 줌

```kotlin
fun <T> myLinkedListOf(

): MyLinkedList<T>? {
    if(elements.isEmpty()) return null 
    val head = elements.first() 
    val elementsTail = elements.copyOfRange(1, elements.size) 
    val tail = myLinkedListOf(*elementsTail)
    return MyLinkedList(head, tail)
}

val list = myLinkedListOf(1, 2)
```

생성자의 역할을 대신 해주는 함수를 팩토리 함수라고 부름

생성자 대신에 팩토리 함수를 사용할 경우 다양한 장점

1. 생성자와 다르게, 함수에 이름을 붙일 수 있음
    
    이름은 객체가 생성되는 방법과 아규먼트로 무엇이 필요한지 설명 가능
    
    동일한 파라미터 타입을 갖는 생성자의 충돌을 줄일 수 있음
    
2. 생성자와 다르게, 함수가 원하는 형태의 타입을 리턴 가능
    
    따라서 다른 객체를 생성할 때 사용할 수 있고, 인터페이스 뒤에 실제 객체의 구현을 숨길 때 유용
    
3. 생성자와 다르게, 호출될 때마다 새 객체를 만들 필요가 없음
    
    함수를 사용해 객체를 생성하면 싱글턴 패턴처럼 하나만 생성하게 강제하거나, 최적화를 위해 캐싱 메커니즘을 사용할 수 있음
    
4. 팩토리 함수는 아직 존재하지 않는 객체를 리턴 가능
    
    이를 통해 프로젝트를 빌드하지 않고도 앞으로 만들어질 객체를 사용 가능
    
5. 객체 외부에 팩토리 함수를 만들면, 그 가시성을 원하는대로 제어 가능
6. 팩토리 함수는 인라인으로 만들 수 있으며, 그 파라미터들을 `reified`로 만들 수 있음
7. 팩토리 함수는 생성자로 만들기 복잡한 객체도 만들 수 있음
8. 생성자는 즉시 슈퍼클래스나 기본 생성자를 호출해야 하지만, 팩토리 함수는 원할 때 생성자 호출 가능

다만 팩토리 함수로 클래스를 만들 때는 약간의 제한이 발생함

서브클래스 생성에는 슈퍼클래스 생성자가 필요하기 때문에 서브클래스를 만들 수 없음

하지만 팩토리 함수로 슈퍼클래스를 만들기로 했다면 그 서브클래스에도 팩토리 함수를 만들면 되기 때문에 아무런 문제가 되지 않음

팩토리 함수의 종류

1. companion 객체 팩토리 함수
2. 확장 팩토리 함수
3. 톱레벨 팩토리 함수
4. 가짜 생성자
5. 팩토리 클래스의 메서드

---

# Companion 객체 팩토리 함수

팩토리 함수를 정의하는 가장 일반적인 방법은 Companion 객체를 사용하는 것임

```kotlin
class MyLinkedList<T>(
    val head : T, 
    val tail : MyLinkedList<T>?
){ 
    companion object{ 
        fun <T> of(vararg elements : T) : MyLinkedList<T>? {
			/*...*/ 
        } 
    }
}

// 사용
val list = MyLinkedList.of(1,2)
```

자바 개발자라면 위 코드가 정적 팩토리 함수 같다는 것을 쉽게 알 수 있음

C++와 같은 프로그래밍언어에서는 이를 이름을 가진 생성자라고 표현함

코틀린에서는 이러한 접근 방법을 인터페이스에서도 구현할 수 있음

```kotlin
class MyLinkedList<T>(
    val head : T,
    val tail : MyLinkedList<T>?
){
    /*...*/
}
interface MyList<T>{
    companion object{ 
        fun <T> of(vararg elements : T) : MyLinkedList<T>? {} 
    }
}

val list = MyLinkedList.of(1,2)
```

함수의 이름만 보면 무엇을 하는 함수인지 잘 모를 수도 있겠지만, 대부분의 개발자는 자바에서 온 규칙 덕분에 이미 이 이름에 익숙할 것이므로 큰 문제 없이 이해할 수 있음

이외에도 다음과 같은 이름을 많이 사용함

- `from`
    
    파라미터를 하나 받고, 같은 타입의 인스턴스를 리턴하는 타입변환 함수
    
    `val data: Data = Date.from(instant)`
    
- `of`
    
    파라미터를 여러 개 받고, 인스턴스를 만들어 주는 함수
    
    `val faceCards : Set<Rank> = EnumSet.of(JACK , QUEEN , KING)`
    
- `valueOf`
    
    `from` 또는 `of`와 비슷한 기능을 하면서도, 의미를 좀 더 쉽게 읽을 수 있게 이름을 붙인 함수
    
    `val prime :BigInteger = BigInteger.valueOf(Integer.MAX_VALUE)`
    
- `instacne` 또는 `getInstance`
    
    싱글턴으로 인스턴스 하나를 리턴하는 함수
    
    `val StackWalker = StackWalker.getInsetance(option)`
    
- `createInstance` 또는 `newInstance`
    
    함수를 호출할때마다, 새로운 인스턴스를 반환함
    
    `val newArray.newInstance(classObject , arrayLeen)`
    
- `getType`
    
    `getInstance`처럼 작동하지만, 팩토리함수가 다른 클래스에 있을 때 사용
    
    타입은 팩토리 함수에서 리턴하는 타입
    
    `val fs : FileStore = File.getFilesStore(path)`
    
- `newType`
    
    `newInstance`처럼 작동하지만, 팩토리함수가 다른 클래스에 있을 때 사용
    
    타입은 팩토리 함수에서 리턴하는 타입
    
    `val br : BuffedReader = File.newBufferedReader(path)`
    

경험이 없는 코틀린 개발들은 companion 개체를 단순한 정적 멤버처럼 다루는 경우가 많음

companion 객체는 인터페이스를 구현할 수 있고, 클래스를 상속받을 수 있음

일반적으로 다음과 같은 형태로 companion 객체를 만드는 팩토리 함수를 만듦

```kotlin
abstract class ActivityFactor{ 
    abstract fun getIntent(context: Cotnext) : Intent

    fun start(context : Context)[
    	val intent = getIntent(context)
        context.startActivity(intent)
    }

}

class MainActivity : xxx{

companion object : ActivityFactory(){
    overide fun getIntext(cotnext: Context) Intent =
        Intent(context ,MainActivity::class.java
    }
}

//사용
val intent = MainActivity.getIntent(context)
MainActivity.start(context)
MainAcitivity.startForResult(activity, requestCode)
```

추상 compainon 객체 팩토리는 값을 가질수 있어서, 캐싱을 구현하거나, 테스트를 위한 가짜 객체를 생성할 수 있음

---

# 확장 팩토리 함수

이미 compainon 객체가 존재할때, 이 객체의 함수처럼 사용할 수 있는 팩토리 함수를 만들어야 할 때가 있는데, 이때 companion 객체를 직접 수정할 수는 없고, 다른 파일에 함수를 만들어야 한다면 어떻게 해야할까?

⇒ 이런 경우 확장 함수를 활용하면 됨

다음과 같은 `Tool` 인터페이스를 교체할 수는 없다고 가정

```kotlin
interface Tool{
    compainon obejct { /*...*/ }
}
```

그래도 companion 객체를 활용해서 확장 함수를 정의 가능

```kotlin
fun Tool.Compainon.createBigTool( /*...*/ ) : BigTool {
    //...
}
```

---

# 톱레벨 팩토리 함수

객체를 만드는 방법 중 `listOf`, `setOf`, `mapOf` 등과 같은 톱레벨 팩토리 함수를 이용하는 방법이 있음

```kotlin
class MainActivity: Activity { 
    companion object { 
        fun getIntent(context: Context) = 
            Intent(context, MainActivity::class.java)
    }
}
```

하지만 `public` 톱레벨 함수는 모든 곳에서 사용될 수 있으므로, IDE가 제공하는 팁을 복잡하게 만드는 단점이 있어서, 톱레벨의 함수를 만들 때는 꼭 이름을 신중하게 생각해서 잘 지정해야 함

---

# 가짜 생성자

코틀린의 생성자는 톱레벨 함수와 같은 형태로 사용됨

```kotlin
class A
val a = A()
```

따라서 다음과 같이 톱레벨 함수처럼 참조될 수 있음(생성자 레퍼런스는 함수 인터페이스로 구현함)

```kotlin
val reference: () -> A = ::A
```

일반적인 사용의 관점에서 대문자로 시작하는지 아닌지는 생성자, 함수를 구분하는 기준임

함수도 대문자로 시작할 수 있지만, 이는 특수한 다른 용도로 쓰임

예를 들어 `List`, `MutableList`는 인터페이스이므로 생성자를 가질 수 없음

하지만 `List`를 생성자처럼 쓰는 코드를 봤을 것임

```kotlin
fun main() {
    List(4) { "User$it" }   // [User0, User1, User2, User3]
}
```

이는 다음과 같은 함수가 코틀린 1.1부터 stdlib에 포함됐기 때문임

```kotlin
public inline fun <T> List(
    size: Int,
    init: (index: Int) -> T
): List<T> = MutableList(size, init)

public inline fun <T> MutableList(size: Int, init: (index: Int) -> T): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
}
```

이런 톱레벨 함수는 생성자처럼 보이고 생성자처럼 작동함

하지만 팩토리 함수와 같은 모든 장점을 가짐

많은 개발자가 이 함수가 톱레벨 함수인지 잘 모름

그래서 이걸 가짜 생성자(fake constructor)라고 부름

개발자가 진짜 생성자 대신 가짜 생성자를 만드는 이유는 아래와 같음

- 인터페이스를 위한 생성자를 만들고 싶을 때
- reified 타입 아규먼트를 갖게 하고 싶을 때

이를 제외하면 가짜 생성자는 진짜 생성자처럼 동작해야 함

생성자처럼 보여야 하고 생성자와 같은 동작을 해야 함

캐싱, nullable 타입 리턴, 서브클래스 리턴 등의 기능까지 포함해 객체를 만들고 싶다면 companion 객체 팩토리 메서드처럼 다른 이름을 가진 팩토리 함수를 쓰는 게 좋음

가짜 생성자를 선언하는 다른 방법으로 `invoke` 연산자를 갖는 companion 객체를 쓰는 방법이 있음

```kotlin
fun main() {
    Tree(10) { "$it" }
}

class Tree<T> {
    companion object {
        operator fun <T> invoke(
            size: Int,
            generator: (Int) -> T
        ): Tree<T> {
            // ...
        }
    }
}
```

다만 이와 같은 방식은 거의 사용되지 않고, 추천하지 않음(item 12에 위배)

companion 객체가 `invoke`를 가지면 아래와 같은 코드를 사용할 수 있음

참고로 함수명을 활용해서도 연산자의 기능을 활용할 수 있다는 걸 기억해야 함

```kotlin
Tree.invoke(10) { "$it" }
```

invoke는 호출한다는 의미이므로, 객체 생성과 의미가 다름

이런 식으로 연산자를 오버로드하면 원래 의미와 차이가 발생함

또한 이런 방식은 톱레벨 함수로 만드는 코드보다 훨씬 복잡함

리플렉션을 보면 지금까지 봤던 생성자, 가짜 생성자, invoke()의 복잡성을 확인할 수 있음

```kotlin
fun main() {
    // 생성자
    val f: () -> Tree = ::Tree

    // 가짜 생성자
    val f2: () -> Tree = ::Tree

    // invoke()를 갖는 companion 객체
    val f3: () -> Tree = Tree.Companion::invoke
}
```

따라서 가짜 생성자는 톱레벨 함수를 쓰는 게 좋음

기본 생성자를 만들 수 없는 상황 또는 생성자가 제공하지 않는 기능(ex. `reified` 타입 파라미터 등)으로 생성자를 만들어야 하는 상황에만 가짜 생성자를 쓰는 게 좋음

---

# 팩토리 클래스의 메서드

팩토리 클래스와 관련된 추상 팩토리, 프로토타입 등의 수많은 생성 패턴이 있음

이런 패턴들은 각각 다양한 장점이 있음

이러한 패턴 중 일부는 코틀린에선 적합하지 않음

예를 들어 점층적 생성자 패턴, 빌더 패턴은 코틀린에선 의미가 없음

팩토리 클래스는 클래스의 상태를 가질 수 있다는 특징 때문에 팩토리 함수보다 다양한 기능을 가짐

아래 코드는 다음 ID(`nextId`)를 갖는 학생을 생성하는 팩토리 클래스임

```kotlin
fun main() {
    val factory = StudentFactory()
    val s1 = factory.next(name = "김", surName = "철수")
    println(s1)
    val s2 = factory.next(name = "박", surName = "미희")
    println(s2)
}
// Student(id=0, name=김, surName=철수)
// Student(id=1, name=박, surName=미희)

data class Student(
    val id: Int,
    val name: String,
    val surName: String
)

class StudentFactory {
    var nextId = 0
    fun next(name: String, surName: String) =
        Student(nextId++, name, surName)
}
```

팩토리 클래스는 프로퍼티를 가질 수 있음

이를 활용하면 다양한 종류로 최적화하고, 다양한 기능을 도입 가능

예를 들어 캐싱을 활용하거나, 이전에 만든 객체를 복제해서 객체를 생성하는 방법으로 객체 생성 속도를 높일 수 있음

---

# 정리

- 코틀린은 팩토리 함수를 만들 수 있는 다양한 방법들을 제공함
- 이러한 다양한 방법은 각각 여러 특징을 갖고 있음
- 객체를 생성할 때는 이런 특징을 잘 파악하고 사용해야 함
- 가짜 생성자, 톱레벨 팩토리 함수, 확장 팩토리 함수 등 일부는 신중하게 사용 해야 함
- 팩토리 함수를 정의하는 가장 일반적인 방법은 companion 객체를 사용하는 것임
- 또한 이 방식은 자바 정적 팩토리 메서드 패턴과 굉장히 유사하고 코틀린은 자바의 스타일과 관습을 대부분 상속하고 있으므로, 대부분의 개발자에게 안전하고 익숙함

---