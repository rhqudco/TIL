# 스프링 핵심 원리 - CGLIB
- CGLIB: Code Generator Library
  - 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리
  - 인터페이스가 없어도 구체 클래스만 가지고 동적 프록시를 만들 수 있다.
  - 외부 라이브러리이지만, 스프링 프레임워크가 스프링 내부 소스 코드에 포함하여 스프링을 사용한다면 별도로 의존성 추가 없이 사용 가능
- 참고
  - 스프링의 ProxyFactory를 사용하면 되기 때문에 CGLIB를 직접 사용하는 일은 거의 없다.
  - 대략적인 개념만 알아보자.

## 에제 코드
JDK 동적 프록시에서 실행 로직을 위해 `InvocationHandler`를 제공했듯, CGLIB는 `MethodInterceptor`를 제공한다.
~~~java
package org.springframework.cglib.proxy;

import java.lang.reflect.Method;

public interface MethodInterceptor extends Callback {
  Object intercept(Object var1, Method var2, Object[] var3, MethodProxy var4) throws Throwable;
}
~~~
- Object var1: CGLIB가 적용된 객체
- Method var2: 호출된 메서드
- Object[] var3: 메서드 호출 시 전달된 인수
- MethodProxy var4: 메서드 호출에 사용 (더 빠른 성능을 내준다고 함)

~~~java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {
  private final Object target;

  public TimeMethodInterceptor(Object target) {
    this.target = target;
  }

  @Override
  public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
    log.info("execute TimeProxy");
    long startTime = System.currentTimeMillis();

    Object result = methodProxy.invoke(target, objects);

    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("shutdown TimeProxy resultTime:{}", resultTime);
    return result;
  }
}
~~~
- `TimeMethodInterceptor`는 `MethodInterceptor` 인터페이스를 구현해서 CGLIB 프록시의 실행 로직을 정의
- JDK 동적 프록시를 설명할 때 예제와 거의 같은 코드
- `Object target` : 프록시가 호출할 실제 대상
- `proxy.invoke(target, args)`: 실제 대상을 동적으로 호출한다.
  - 참고로 `method`를 사용해도 되지만, CGLIB는 성능상 `MethodProxy proxy`를 사용하는 것을 권장한다.

~~~java
@Slf4j
public class CglibTest {

  @Test
  void cglib() {
    ConcreteService target = new ConcreteService();

    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(ConcreteService.class);
    enhancer.setCallback(new TimeMethodInterceptor(target));
    ConcreteService proxy = (ConcreteService) enhancer.create();

    log.info("targetClass = {}", target.getClass());
    log.info("proxyClass = {}", proxy.getClass());

    proxy.call();
  }

}
~~~

~~~
22:17:42.759 [main] INFO hello.proxy.cglib.CglibTest - targetClass = class hello.proxy.common.service.ConcreteService
22:17:42.760 [main] INFO hello.proxy.cglib.CglibTest - proxyClass = class hello.proxy.common.service.ConcreteService$$EnhancerByCGLIB$$25d6b0e3
22:17:42.760 [main] INFO hello.proxy.cglib.code.TimeMethodInterceptor - execute TimeProxy
22:17:42.765 [main] INFO hello.proxy.common.service.ConcreteService - call ConcreteService
22:17:42.765 [main] INFO hello.proxy.cglib.code.TimeMethodInterceptor - shutdown TimeProxy resultTime:5
~~~

- `Enhancer`
  - CGLIB는 `Enhancer`를 사용해서 프록시를 생성한다.
- `enhancer.setSuperclass(ConcreteService.class)`
  - CGLIB는 구체 클래스를 상속 받아서 프록시를 생성할 수 있다. 어떤 구체 클래스를 상속 받을지 지정한다.
- `enhancer.setCallback(new TimeMethodInterceptor(target))`
  - 프록시에 적용할 실행 로직을 할당한다.
  - `setCallback`인 이유는 `TimeMethodInterceptor`는 `MethodInterceptor`의 구현체인데, `MethodInterceptor`는 `Callback`의 자식 클래스다.
- `enhancer.create()`
  - 프록시를 생성한다. 
  - 앞서 설정한 `enhancer.setSuperclass(ConcreteService.class)`에서 지정한 클래스를 상속 받아서 프록시가 만들어진다.

- `ConcreteService$$EnhancerByCGLIB$$25d6b0e3`
  - CGLIB가 생성한 클래스 이름
  - `대상클래스$$EnhancerByCGLIB$$`
임의코드 규칙으로 생성

<img width="956" alt="image" src="https://github.com/user-attachments/assets/3a15d766-a743-419a-ac00-eb875fd35868">

## CGLIB 제약
- 클래스 기반 프록시는 상속을 사용하기 때문에 JDK Proxy와 유사하게 몇가지 제약 존재
  - 부모 클래스의 생성자를 체크하애 한다. -> CGLIB는 자식 클래스를 동적으로 생성하기 때문에 기본 생성자가 필요
  - 클래스에 final 키워드가 있으면 상송이 불가능하다. -> CGLIB에서는 예외가 발생한다.
  - 메서드에 final 키워드가 붙으면 해당 메서드를 오버라이딩 할 수 없다. -> CGLIB에서는 프록시 로직이 동작하지 않음


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__