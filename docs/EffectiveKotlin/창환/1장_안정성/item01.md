# item 1 가변성을 제한하라

## 불변을 사용했을때 장점에 대한 공감 포인트
### 불변은 디버깅에 유리하다  
5p 본문 1,2,3 번 항목  

머리속으로 이해도되고 어느정도는 공감가지만 숨쉬듯이 느끼지는 못한다.  
솔직히 개발하면서 가끔 동시성 관련 이슈도 겪어(캐시워크 걸음수 관리같이 멀티스레드로 들어오는 경우)  
경각심은 있지만 뭔가 안지켰을때 막 찌릿찌릿한 수준은 아니다.  

아직 미친 멀티스레드 환경에서 많은 연산이 있는 주요한(돈관련된 계산 로직 같은)로직을 안겪어서 그런것 같다.  
-> 한번 이슈가 크게터져서 뚜들겨 맞기전에 찌릿찌릿 레이더를 켜야한다.  

솔직히 우리 회사에서 동시성을 겪을만한 부분은 걸음수 동기화가 큰데(멀티스레드에서 하나의 자원을 다룸)  
걸음수는 특정하기가 어려워서 일부 연산이 씹혀도 씹힌지도 모르고있는거 같다.  
웨어를 개발하며 웨어와 동기화 문제에서 동시성 문제를 겪어서 조금은 공감이 가는 상태가 되었을지도?  

돈관련 계산을 다루는 로직이 들어가면 확 공감될 것 같다.  
멀티유저 계산기를 언젠가 연습으로 만들어 봐야겠다.(배민 장바구니에 여러명이 담는 기능 같은거)

이러한 문제를 겪기 이전에 선제 조치로 불변을 애용해서 이런 문제를 틀어막자  
그리고 동시성 관련된 사항은 몸으로 겪기 이전에 좀 학습이 필요할거같다.  
(확실히 서버보다 안드가 동시성 관련하여 둔한 느낌이 든다. 나만 바보인가?)

### 불변은 테스트하기 쉽다
5p 4번 항목

뭐 꼭 불변만이 테스트하기 쉽다기 보다는(어짜피 상태가 바뀌는거 다 각각 개별 함수로 만들어서 만들어야할테니)  
검증의 범위가 좁혀지는거에서 매우 공감한다.  
검증의 범위를 한정하고 좁히는것 자체가 생각보다 머리아픈데 불변을 사용하는것 만으로 유닛테스트 검증범위가 한정될께 예상된다.  
이 얼마나 위력적인가?

### 옵저빙 패턴 같은데서 관리하기 어렵다, 객체내 룰이 있다면 유지하기 어렵다.
5p 5번항목

안드로이드 개발자라면 모두 겪어본 일 아니겠는가?  
옵져버블에다가 뮤터블한 리스트 박아놓고 내부값 바꿔주면 옵져빙 로직 안돈다.  
그리고 관리도 어렵다. 불변짱  
그리고 그 정렬 예시도 적절했다. 동시성과 콜라보하면 생각만해도 머리아플거 같다.

## 코틀린에서 가변성 제한하기(val, 가변/읽기전용 컬렉션, copy)
### val, 컬렉션 이야기는 시사하는 바가 같다.
요즘 미장이 박살나면서 이런밈이 있다.   
![image](https://github.com/user-attachments/assets/7c70fb1b-f83f-42a1-86ee-8dc2e114a140)  
이거랑 같은 이야기다.

val쓴다고 imutable 쓴다고 불변이 아니다.  
불변의 의미가 있어야 그곳이 불변이다.

제발 의도한대로 쓰고 이상한짓좀 하지마라.  
캐스팅, 리플렉션 등등 우회하지 말아라 제발  

코드 짜는사람은 방어운전처럼 사고 안나게 방어적 복사 하면서 안전 운행하고  
api 사용하는 사람도 솔직히 상식선을 지키면 되는데 그게 생각보다 안일어나니 이런 책들이 잘팔리는거 같다.  
상식선에서 움직이자.  

근데 책에서는 다운캐스팅에 대한 경계만 나와있는데  
캐스팅 자체가 객체지향을 좀 깨버리는 행위 아닌가?  
이거는 근데 공부 명확히 아직 안해서 모르겠다.  

대충 예상가기로는 객체가 본인의 행동을 결정해야한다 이런 이야기겠지만  
이런 내용 엘레강트 오브젝트 3.7 장에 나온다는데 난 돈이없어서 그책을 못샀다 흑흑흑.  
인터넷에 정리된글은 생략되서 도통이해가 안된다.  
연봉오르면 사서 읽어야지  

### val 에 대한 이야기
val 썻다고 다 불변은 아니고  
진짜 불변이여야지 불변이다.  

책에 나와있듯이 워낙 코틀린은 이것저것 다 제공해서 변수인데 by로 객체에 위임도 떄릴수 있고  
getter setter 커스텀해서 var를 바라보게 할수도 있는거고  
어쩃든 val에 들어있는게 가변일 수 있는 가능성이 은근 많다.  

그니까 불변이라는 개념을 잘 활용하자  

ps. 근데 코틀린 커스텀 getter setter가 getter setter 라고 할수 있나?
얼마전에 우테코5기 였던것 스레드에서 서로 또 이야기가 오갔는데 
일단 getter setter 명확한 정의가 어디있는지 뭘 따라야하는지도 모호하고(나는 걍 자바빈을 예시로 들었었다. 토론할떄는 이유: 유명하니까)  

이때 제이슨이 말씀해주신 내용이 좀 이런것들을 푸는 키워드인것 같긴하다.(객체는 계약에의해 작동해야한다.)  
그때 받은 객체관련글 여기다 넣어놓고 두고 두고 읽어야겠다 스레드는 3개월밖에 못사니까 ㅠㅠ  
[객체 관련글](https://codingnuri.com/seven-virtues-of-good-object/)

근데 커스텀 getter, setter에 로직같은게 들어가도 코틀린은 언어단에서 너무 유명하게 설정해놨으니 사실상 계약이라고 봐도 무방하지 않을까?  
(뭐 서로 통하면 장땡 아니겠는가?)  
![image](https://github.com/user-attachments/assets/38030acf-636a-498a-b695-3335c1f29f9b)


### 컬렉션 이야기
여기 이야기는 위에 나왔다 다.  
플렛폼 별로 실제 자료구조도 다를테고 불변으로 인터페이스를 설정해놨으면  
인터페이스 위쪽으로 추상화해서 사용해주자 그거 꺠면 오류나는거 예시로도 나와있으니  

그리고 방어적 복사는 뭐 숨쉬듯이 해야지(리사이클러뷰만 써도 공감가지 않나?)  

### Copy 이야기
코틀린은 진짜 짱이다.  
data클래스 너무 편하고 짱짱 기능 불변까지 신경쓰는 킹갓 제너럴 언어  

자바 보니까 대충 롬복 쓰거나 뭐 자바 14쯤에나 이런기능이 있는것 같다.(저는 자바 싫어요 ㅎ)  
코틀린에 감사하자.(그리고 코틀린이 함수형을 지원한다는 큰 반증)  

## 다른 종류의 변경 가능 지점
### val/mutablecollection vs var/immutablecollection
var/immutablecollection win  
간만에 들었을때 멍했는데 이유 설명듣고 빡공감되었다.  

1. 코드 변경지점이 컬렉션 구현 내부에 있다.
   - 동시성 찌릿찌릿한가요?
   - 블랙박스(플렛폼따라 다름)
2. private set으로 변경점 제한
   - 코틀린이 또 사랑스러워 보임

솔직히 옵저버블 이거는 공감안감 -> 이렇게 딱히 안쓸거같음 

## 변경가능 지점 노출하지 말기
이거는 당연한 이야기아님?  
캡슐화 합시다.  
방어적 복사도 합시다.


## 기술부채
이거하면서 함수형좀 알아볼라고 글좀 찾았는데 시간없어서 다 못읽었는데 나중에 읽어야겠다.
[불변 및 상태 관련글 자바스크립트 예시랑 조큼 킹받](https://evan-moon.github.io/2020/01/05/what-is-immutable/)
