# item 10 단위테스트를 만들어라

단위 테스트에 대해서 홍보하는 아이템이였다.  
개인적으로 홍보가 좀 잘되어 안드로이드 진영에서 단위테스트가 보편화 되었으면 좋겠다.  

책에는 이런 내용들이 나온다.

- 단위테스트 방법(어떤 케이스를 테스트 할 것인지)
- 단위테스트의 장점
- 단위테스트의 단점(사실 단점 인데 허울뿐인 단점만 적혀있음)
- 단위테스트는 개발에 대한 이해도가 높을수록 효율이 나온다는 이야기(100%동의)
- 단위테스트를 진행해볼법한 단위

사실 책에 나오는 부분은 a4용지 몇장안에 설명하기위해 고리 타분한 일반적인 부분만 늘어놓았다.  
물론 이 책의 저자도 하고싶은 말이 엄청 많아서 드릉드릉 했을것이다.  

그래서 그냥 내가 테스트에 대해서 평소 떠오르는 부분을 가볍게 정리해보고  
과거에 테스트좀 진흥해보려 회사분들에게 홍보하던 문서 일부를 따오려한다.  

## 테스트는 목적성을 띄어야한다.
테스트를 위한 테스트를 경계하자.  
테스트는 도구일뿐이다. 테스트를 진행할때 목적성을 가지고 이루려는 목표를 설정하자.  

그냥 기술스택이 화려하기 위해 의미도 없는 테스트 커버리지를 늘리는것만큼 바보같은 행위도 없는것 같다.  

조직의 목적에 맞춰서 진행 하려는 테스트의 종류를 고르고 실행하자.  

개인적인 의견으로는 당연한 이야기지만 유닛테스트가 비용대비 효율이 제일 잘 나오며  
테스트를 위한 구조를 잡고 순수 자바코틀린으로 로직을 구성했을때는 거의 그냥 숨쉬듯이 테스트를 짤 수 있다.  
이러한 유닛테스트를 생활화하여 기본적인 코드의 신뢰도, 문서의 역할로 사용하자  
(프로젝트의 성향을 떠나서 일단 진행해도 좋은 결과를 내는 경우가 많았다.)

UI 테스트는 의견이 분분하겠지만 정말 변경되지 않는 영역 혹은 필요한 상황이 생겨 목적성을 가지고 하지 않는 이상  
드는 비용대비 효율이 크게 좋지 못하다고 생각한다.(물론 필요할때는 진행하는것이 맞다고 생각한다.)

이에 이러한 상황들을 직접 생각해보고 상황에 맞춰서 유연하게 행동하자

## 테스트를 짜기 이전에 테스트를 위한 구조를 구축하여 비용을 줄이자  
객체지향이던 함수형이던 각 패러다임에서 항상 고민하는 문제 중 하나는 어떻게 코드를 테스트할 것인가 이다.   
어떤 방향으로 진행하든 비슷한 고민을 하게 되며 같은 문제를 해결하게 되니 각자 흥미있는 분야를 깊게 파보고 고민해보자.  

아키텍처 패턴 또한 마찬가지이다.  
짦은 식견이지만 내가 살펴본 아키텍처 패턴들은 대부분 객체지향의 방향으로 문제를 해결하는 경우가 많았다.  
결론적으로는 무슨 무슨 패턴이던 하려는 목적과 해결하려는 문제는 동일하니 부수적인 구현에 목매지말고 진정 문제를 해결하고있는지   
본질을 살펴보는것이 좋다고 생각한다.  

테스트를 위한 구조를 이 글안에 담기에는 너무 부족하여 생략하지만  
결론적으로 테스트를 위한 구조를 가져가면 그냥 아무렇게나 짜여진 코드보다 테스트를 짜는 비용이 기하급수적으로 줄어든다.  
정말 유명한 책들이 많다. 해당 책들을 정독하고 고민해보자   
ex) 클린아키텍처, 좋은코드 나쁜코드, 클린코드, TDD, DDD(이건 나도 전혀 접점도 읽어보지도 않았는데 주변의 조언으로 객체지향을 접근하는 확장된 시각을 제공하는것으로 알고있다.)

## 단위 테스트 관련 장점 홍보글
기존에 써놨던 [테스트 진흥 문서](https://github.com/Mash-Up-Android/Effective_Study/blob/main/docs/EffectiveKotlin/%EC%B0%BD%ED%99%98/1%EC%9E%A5_%EA%B0%80%EB%8F%85%EC%84%B1/%ED%85%8C%EC%8A%A4%ED%8A%B8_%ED%99%8D%EB%B3%B4_%EC%B6%94%EA%B0%80%EA%B8%80.md)이다.  
테스트를 짜면서 체감되는 장점을 쭉써놨는데 식견이 짧은 나지만 뭐 당장 단위테스트를 짤 이유는 충분히 설명된다고 생각한다.  
심심하다면 한번 봐보자
