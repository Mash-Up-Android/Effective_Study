### close로 닫아야 하는 애들

InputStream, OutputStream, Socket 이런거는 close로 명시적으로 닫아줘야 함

AutoCloseable을 상속받는 Closeable이 close를 구현하고 있음, 결국 저 위에 애들이 Closeable을 상속하나보군.

close를 안하면 나중에 GC가 알아서 모셔가는데 이게 생각보다 쉽지않대 그래서 close를 해줘야 함

이떄 try-catch를 해줘야하는데 굉장히 번거로워.

use를 쓰자~!

### use, useLines

use를 쓰면 통짜로 읽고, useLines는 한줄씩 읽어들인다

useLine은 메모리에 파일의 한줄씩 유지하기 때문에 대용량 파일 처리에 용이한 장점이 있지만 속도가 느리곘지?