### 예외(Exception)와 에러(Error)의 차이

에러는 복구할 수 없는 프로그램 오류

OOM이나 StackOverFlowError 같은거 ㅇㅇ

예외는 runCatching 같은걸로 잡아서 족칠수있는거

예외는 정보 전달용으로 사용되면 안된다.

잘못된 상황을 나타내는거고 안정성을 위해 반드시 처리되어야 하는 놈

<img src="https://github.com/user-attachments/assets/a7a71d3a-da35-4051-8729-2d55354691ad" />

### UncheckedExcpetion vs CheckedException

Exception은 RuntimeException과 그 외 Exception으로 나뉨

RuntimeException은 UncheckedException

그 외는 CheckedException임

CheckedException은 말 그대로 컴파일러가 체크 가능한거

FileNotFound나 ClassNotFound 이런거는 빌드하면서 컴파일러가 잡을 수 있음

UncheckedException은 컴파일러가 체크할 수 없는거

NullPointerException 이런거는 빌드 후 코드 실행하면서 어? 이자식 null이잖아? 하고 뱉는거

보통 이런거는 개발자 실수로 발생함

둘을 구분하는 기준은 컴파일 타임에 예외가 체크 가능하냐 안하냐 이차이로 보면 됨 (요게 정리 문장인듯)

### try-catch를 내부에 코드를 쓰면 컴파일러가 할 수 있는 최적화가 제한된다

.. 요게 무슨말일까?

요게 블록안의 코드에서 뱉어내는 Exception들이 어쨌든 한정되어있을 텐데 이걸 다 일일이 파악해서 catch하는 것도 빡세고 그렇다고 e: Throwable 이렇게 하기도 뭐하고 약간 그런 상황을 의미하는건가?

### null과 Failure는 예상되는 오류를 표현하기에 좋은 방식

간단하게 처리가 가능하기 때문에 예상되는 오류는 null과 Failure를 사용하고

그외의 오류들은 throw하는게 좋다

- 근데 클라개발은 창환쓰가 이야기한 fail-safe하게 개발을 하는게 일반적인데 이걸 throw한다? 어차피 다시 catch 해줘야 하지않나?? 요거 궁금 포인트

### null 처리 어케함

SafeCall, 얼리리턴, 엘비스 등등

추가정보 없이 단순 에러 처리일 때 사용

get보다 getOrNull 같은거는 뭔가 찜찜하다 싶을 때 사용하니까 개발자가 알잘딱깔센 처리할 수 있음

get은 바로 예외 토해내니까

### Result 처리 어케함

when절 써 ㅋㅋ~

근데 이게 try-catch 보다 효율적이다.

명시적인건 ㅇㅈ

추가정보가 필요할 때 사용하면 좋음