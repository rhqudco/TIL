# 스프링 핵심 원리 - 고급편 > JDK 동적 프록시 예제 코드

__A, B Interface / Impl__
~~~java
public interface AInterface {
  String call();
}

@Slf4j
public class AImpl implements AInterface {

  @Override
  public String call() {
    log.info("A call");
    return "a";
  }
}
// ===========================================================
public interface BInterface {
  String call();
}

@Slf4j
public class BImpl implements BInterface {

  @Override
  public String call() {
    log.info("call B");
    return "b";
  }
}
~~~

JDK를 통한 동적 프록시를 사용하는 방법은 `InvocationHandler` 인터페이스를 구현해서 작성하면 된다.

~~~java
public interface InvocationHandler {
   // 생략
  @CallerSensitive
    public static Object invokeDefault(Object proxy, Method method, Object... args)
            throws Throwable {
        Objects.requireNonNull(proxy);
        Objects.requireNonNull(method);
        return Proxy.invokeDefault(proxy, method, args, Reflection.getCallerClass());
    }
}
~~~

위 인터페이스에 제공되는 파라미터는 다음과 같다.
- Object proxy: 프록시 자기 자신
- Method method: 호출한 메서드
- Object[] args: 메서드 호출 시 전달한 인수

이제 위 `InvocationHandler`를 구현하는 코드를 보자

~~~java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

  private final Object target;

  public TimeInvocationHandler(Object target) {
    this.target = target;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    log.info("Time Proxy Execute");
    long startTime = System.currentTimeMillis();

    Object result = method.invoke(target, args);

    long endTime = System.currentTimeMillis();

    long resultTime = endTime - startTime;
    log.info("Time Proxy Shutdown resultTime = {}", resultTime);
    return result;
  }
}
~~~

원본 클래스를 사용하기 위해 `private final Object target`를 통해 생성자로 생성한다.
그리고 `invoke` 메서드를 구현해주는데, 인자로 받은 메서드를 invoke하여 실행해준다.
정리하자면

- `Object target`: 동적 프록시가 호출할 대상
- `method.invoke(target, args)`: 리플렉션을 사용해서 `target` 인스턴스의 메서드를 실행. `args` 는 메서드 호출시 넘겨줄 인수

가 된다.

이제 테스트 코드를 통해 프록시를 사용해보자.

~~~java
@Slf4j
public class JdkDynamicProxyTest {

  @Test
  void dynamicA() {
    AInterface target = new AImpl();
    TimeInvocationHandler handler = new TimeInvocationHandler(target);

    AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);
    proxy.call();
    log.info("target = {}", target.getClass());
    log.info("proxy = {}", proxy.getClass());
  }

  @Test
  void dynamicB() {
    BInterface target = new BImpl();
    TimeInvocationHandler handler = new TimeInvocationHandler(target);

    BInterface proxy = (BInterface) Proxy.newProxyInstance(BInterface.class.getClassLoader(), new Class[]{BInterface.class}, handler);
    proxy.call();
    log.info("target = {}", target.getClass());
    log.info("proxy = {}", proxy.getClass());
  }
}
~~~

- `new TimeInvocationHandler(target)` : 동적 프록시에 적용할 핸들러 로직이다.
- `Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[] {AInterface.class}, handler)`
  - 동적 프록시는 `java.lang.reflect.Proxy` 를 통해서 생성할 수 있다.
  - 클래스 로더 정보, 인터페이스, 그리고 핸들러 로직을 넣어주면 해당 인터페이스를 기반으로 동적 프록시를 생성하고 그 결과를 반환
  - Proxy.newProxyInstance를 통해 반환되는 값은 Object 타입이기 때문에 타입 캐스팅이 필요하다.

__출력 로그__

~~~java
22:41:01.146 [main] INFO hello.proxy.jdkdynamic.code.TimeInvocationHandler - Time Proxy Execute
22:41:01.147 [main] INFO hello.proxy.jdkdynamic.code.AImpl - A call
22:41:01.147 [main] INFO hello.proxy.jdkdynamic.code.TimeInvocationHandler - Time Proxy Shutdown resultTime = 0
22:41:01.147 [main] INFO hello.proxy.jdkdynamic.JdkDynamicProxyTest - target = class hello.proxy.jdkdynamic.code.AImpl
22:41:01.147 [main] INFO hello.proxy.jdkdynamic.JdkDynamicProxyTest - proxy = class jdk.proxy2.$Proxy8
~~~

출력 로그에서 프록시가 정상 수행된 것을 확인할 수 있다.
`proxy = class jdk.proxy2.$Proxy8`를 보면 이 부분이 동적으로 생성된 프록시 클래스의 정보를 출력하는 로그를 통해 출력된 로그다.
직접 만든 클래스의 정보가 있는 것이 아니라, JDK 동적 프록시가 이름 그대로 `동적`으로 만들어준 프록시고, 이 프록시는 `TimeInvocationHandler` 로직을 실행해준다.

__실행 순서__
1. 클라이언트는 JDK 동적 프록시의 `call()` 실행
2. JDK 동적 프록시는 `InvocationHandler.invoke()`를 호출. (`TimeInvocationHandler`가 구현체로 있기 때문에 `TimeInvocationHandler.invoke()`가 호출됨)
3. `TimeInvocationHandler`가 내부 로직 수행하고, `method.invoke(target, args)`를 호출하여 `target`인 `실제 객체 AImpl`을 호출한다.
4. `AImpl` 인스턴스의 `call()`이 실행
5. `AImpl` 인스턴스의 `call()`의 실행이 끝나면 TimeInvocationHandler로 응답이 돌아와 시간 로그를 출력하고 결과 반환

그림을 통해 살펴보면

<img width="945" alt="image" src="https://github.com/user-attachments/assets/14008aa1-e4e3-4fd2-8cdb-ecc530ffef9b">

위와 같은 형상이다.

__동적 프록시 클래스 정보__
`dynamicA()` 와 `dynamicB()` 둘을 동시에 함께 실행하면 JDK 동적 프록시가 각각 다른 동적 프록시 클래스를 만들어주는 것을 확인할 수 있다.

~~~
proxy = class jdk.proxy2.$Proxy8
proxy = class jdk.proxy2.$Proxy9
~~~

## 정리
예제를 보면 `AImpl`, `BImpl` 모두 각각 프록시를 만들지 않고, JDK 동적 프록시를 통해 동적으로 만들고 `TimeInvocationHandler`는 공통으로 사용했다.

JDK 동적 프록시를 사용하므로써 프록시 적용 대상 객체만큼 인터페이스를 만들지 않아도 되고, 같은 부가 기능 로직을 한번만 개발해서 공통으로 사용할 수 있다.
적용 대상이 100개 1000개가 되더라도 현재 내용상으로는 `InvocationHandler`만 만들어서 넣어주면 된다.
결과적으로 보았을 때 프록시 클래스 생성 문제와 부가 기능 로직 중복 문제 모두 해결되어 SRP(단일 책임 원칙)을 지킬 수 있게 됐다.

그림을 통해 보자

<img width="935" alt="image" src="https://github.com/user-attachments/assets/077ef488-5728-4884-a46b-30c08919f57c">

<img width="953" alt="image" src="https://github.com/user-attachments/assets/58b924f6-3e20-42fd-9b6d-1ec7fd13ed1d">
아래 그림(JDK 동적 프록시 도입 후)에서 점선은 개발자가 직접 만드는 클래스가 아닌 JDK가 만들어 주는 클래스다.

<img width="943" alt="image" src="https://github.com/user-attachments/assets/7b5f29c9-9798-4422-9230-26c620bfa6cb">


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__