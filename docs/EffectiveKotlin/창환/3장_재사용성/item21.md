# item 21 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

item 16에서 지속적으로 이야기했던 내용이다.

책을보면서 기본적으로 제공되는 위임용 객체들이 어떻게 생겼을지 정확히는 못맞춰도 동일한 동작은 가능하게 구현할 수 있어야한다.  
이런거 라이브러리 만들다보면 종종 사용되는데 한번만 딥하게 사용해보면 그 이후에는 그냥 눈에 선하게 보인다.

결국 책 내용중 주요하게 가져갈 것들은  

코틀린 기본제공 주요 프로퍼티 델리게이터
- lazy(by viewModels 내부 까보면 보임)
- Delegates.observable
- Delegates.vetoable
- Delegates.notNull

이것들은 안드개발에서 생각보다 자주쓰인다 -> observable 빼고

그리고 결국 프로퍼티 공통 로직을 객체로 분리 가능하다는것이 결국 키포인트이다.
