# item 23 타입 파라미터의 섀도잉을 피하라  
사실 어떻게 보면 당연한 이야기이다.  
클래스에 종속된 메서드인데 타입이 클래스에 지정 되어있다면 클래스의 것을 사용하는것이 당연하지 않는가?  

결국 개발자의 의도가 중요하다 해당 타입파라미터가 독립적일지 혹은 종속적일지  
이러한 제너릭은 팀내 공통코드 혹은 sdk 개발 혹은 라이브러리 개발하다보면 딥하게 사용하게 된다.  

나는 그러한 개발을 하기전에 이책을 읽고 타입 파라미터 섀도잉을 지양하고있어서 그런지 와닿는 문제케이스를 만나지 못했다.  
하지만 쉽게 일어나는 실수임은 맞는것 같다 마음속에 품어놓고 항상 타입 섀도잉을 지양하자.  


