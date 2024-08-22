# 스프링 고급편 - Proxy
- Proxy를 통하면 Client와 Server 사이에 대리자를 통해 간접적으로 서버에 요청할 수 있게 해준다
  - 직접 호출
    - Client -> Server
  - 간접 호출 (Proxy)
    - Client -> Proxy -> Server
- Proxy를 통해 간접 호출 하면 중간에서 여러가지 일을 할 수 있다.
  - 특정 요청을 했을 때 이미 Proxy에 있을 경우 Client의 요청이 Server까지 도달하지 않고, Proxy를 통해 요청에 대한 응답 가능 (접근 제어, 캐싱)
  - 특정 요청을 했을 때 Proxy를 통해 Server의 기능 뿐 아니라, 부가 기능 수행 가능 (부가 기능 추가)
  - Proxy가 또 다른 Proxy를 호출할 수 있다.
    - Client는 Proxy를 통해 요청했기 때문에 뒤에 어떤 과정이 있는지 알 수 없고, 응답을 받기만 하면 됨 (Proxy Chain)
- 대체 가능
  - Proxy가 되려면 Client는 Server에 요청한 것인지, Proxy에 요청한 것인지 몰라야 한다.
  - 즉, Server와 Proxy는 같은 인터페이스를 사용해야 한다.
  - Client가 사용하는 Server 객체를 Proxy 객체로 변경해도 클라이언트 코드를 변경하지 않고 동작할 수 있어야 한다.
  - ServerInterface가 존재할 때 Proxy와 Server는 ServerInterface의 구현체여야 하며, Client는 ServerInterface를 사용한다는 의미가 된다.
  - Proxy와 Server가 같은 인터페이스 공유한다는 것은 DI를 통해 대체가 가능하다는 뜻이고, Client -> Server에서 Client -> Proxy -> Server가 되어도 Client 변경이 필요 없다는 것
- Proxy의 주요 기능
  - 접근 제어
    - 권한에 따른 접근 차단
    - 캐싱
    - 지연 로딩
  - 부가 기능 추가
    - 원래 서버가 제공하는 기능에 더하여 부가 기능 수행
      - 예: 들어 요청 값이나, 응답 값을 중간에 변형
      - 예: 실행 시간을 측정해서 추가 로그 기록
- GOF 디자인 패턴
  - 데코레이터 패턴, 프록시 패턴 모두 Proxy를 사용하는 방법이지만, 의도(intent)에 따라 구분한다.
    - 프록시 패턴: 접근 제어가 목적
    - 데코레이터 패턴: 새로운 기능 추가가 목적
- 참고
  - 프록시는 객체만을 대상으로 지칭하는 것이 아니다.
  - Client 요청에 따라 Server에 도달하기 전 Proxy Server를 통해 접근 제어가 들어가는 것도 Proxy의 일종이다.
    - 예를 들어 유튜브의 서버가 어디 있는지 정확히 알 수 없지만(아마 미국에 있지 않을까 예상한다.) 한국에서 영상을 보기 위해 접속하여 어디 있는지 모르는 서버에 요청하고, 만약 그 서버가 멀리 있다면 시간이 더 걸릴 것이다.
    - 하지만 Proxy Server가 일본에 있다면 먼저 일본에 있는 Proxy Server에 요청하여 캐싱된 영상을 받아올 수 있다.

## 예제 코드
먼저 Subject라는 interface를 만들자.
~~~java
public interface Subject {
  String operation();
}
~~~

다음은 RealSubject를 만들고 Subject interface를 구현한다.
~~~java
@Slf4j
public class RealSubject implements Subject {

  @Override
  public String operation() {
    log.info("call real object");
    sleep(1000);
    return "data";
  }

  private void sleep(int millis) {
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
~~~

Client를 구현할 차례인데, Subject를 필드로 가지고 subject의 operation을 실행한다.
~~~java
@RequiredArgsConstructor
public class ProxyPatternClient {
  private final Subject subject;

  public void execute() {
    subject.operation();
  }
}
~~~

RealSubject를 생성하고, Client에 Subject의 구현체인 RealSubject를 통해 생성하고 실행한다.
~~~java
public class ProxyPatternTest {

  @Test
  void noProxyTest() {
    RealSubject subject = new RealSubject();
    ProxyPatternClient client = new ProxyPatternClient(subject);

    client.execute();
    client.execute();
    client.execute();
  }
}
~~~

아래와 같은 결과를 확인할 수 있다.
~~~
20:47:20.816 [main] INFO hello.proxy.pureproxy.proxy.code.RealSubject - call real object
20:47:21.823 [main] INFO hello.proxy.pureproxy.proxy.code.RealSubject - call real object
20:47:22.825 [main] INFO hello.proxy.pureproxy.proxy.code.RealSubject - call real object
~~~

이제 Proxy를 도입하자.
~~~java
@Slf4j
public class CacheProxy implements Subject {

  private final Subject target;
  private String cacheValue;

  public CacheProxy(Subject target) {
    this.target = target;
  }

  @Override
  public String operation() {
    log.info("call proxy");
    if(cacheValue == null) {
      cacheValue = target.operation();
    }
    return cacheValue;
  }
}
~~~
CacheProxy 클래스는 Subject를 구현하는 구현체이며 Subject를 필드로 두는 클래스다.
즉, ProxyPatternClient 생성에 사용될 수 있고 Subject의 구현체인 RealSubject를 생성 시 사용할 수 있다는 것이다.
CacheProxy 이름값을 하기 위해 cacheValue를 필드로 두고 cache에 저장하는 것을 구현한다.

마지막으로 실행 코드다.
~~~java
public class ProxyPatternTest {

  @Test
  void cacheProxyTest() {
    RealSubject realSubject = new RealSubject();
    CacheProxy cacheProxy = new CacheProxy(realSubject);
    ProxyPatternClient client = new ProxyPatternClient(cacheProxy);

    client.execute();
    client.execute();
    client.execute();
  }
}
~~~

이제 실행 결과를 보면
~~~
20:51:48.956 [main] INFO hello.proxy.pureproxy.proxy.code.CacheProxy - call proxy
20:51:48.957 [main] INFO hello.proxy.pureproxy.proxy.code.RealSubject - call real object
20:51:49.958 [main] INFO hello.proxy.pureproxy.proxy.code.CacheProxy - call proxy
20:51:49.958 [main] INFO hello.proxy.pureproxy.proxy.code.CacheProxy - call proxy
~~~

CacheProxy를 생성하는데, RealProxy가 필요하긴 하지만 ProxyPatternClient와 RealSubject의 코드 한 줄 변경하지 않고 Proxy를 도입했다.