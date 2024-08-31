# 스프링 핵심 원리 - 고급편 > JDK 동적 프록시 적용 1
이전 내용을 바탕으로 실제 적용 코드를 작성해보자.

~~~java
@RequiredArgsConstructor
public class LogTraceBasicHandler implements InvocationHandler {

  private final Object target; // 프록시가 호출할 대상이 되는 클래스
  private final LogTrace logTrace;

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    TraceStatus status = null;
    try {
      // Logtrace에 사용할 메시지
      // 호출되는 클래스 정보와 메서드 이름을 동적으로 기록
      String message = method.getDeclaringClass().getSimpleName() + "." + method.getName() + "()";
      status = logTrace.begin(message);

      Object result = method.invoke(target, args);

      logTrace.end(status);
      return result;
    } catch (Exception e) {
      logTrace.exception(status, e);
      throw e;
    }
  }
}
~~~

~~~java
@Configuration
public class DynamicProxyBasicConfig {

  @Bean
  public OrderControllerV1 orderControllerV1(LogTrace logTrace) {
    OrderControllerV1Impl orderController = new OrderControllerV1Impl(orderServiceV1(logTrace));
    OrderControllerV1 proxy = (OrderControllerV1) Proxy.newProxyInstance(
        OrderControllerV1.class.getClassLoader(),
        new Class[]{OrderControllerV1.class},
        new LogTraceBasicHandler(orderController, logTrace));

    return proxy;
  }

  @Bean
  public OrderServiceV1 orderServiceV1(LogTrace logTrace) {
    OrderServiceV1Impl orderService = new OrderServiceV1Impl(orderRepositoryV1(logTrace));

    OrderServiceV1 proxy = (OrderServiceV1) Proxy.newProxyInstance(
        OrderServiceV1.class.getClassLoader(),
        new Class[]{OrderServiceV1.class},
        new LogTraceBasicHandler(orderService, logTrace));

    return proxy;
  }

  @Bean
  public OrderRepositoryV1 orderRepositoryV1(LogTrace logTrace) {
    OrderRepositoryV1Impl orderRepository = new OrderRepositoryV1Impl();

    OrderRepositoryV1 proxy = (OrderRepositoryV1) Proxy.newProxyInstance(
        OrderRepositoryV1.class.getClassLoader(),
        new Class[]{OrderRepositoryV1.class},
        new LogTraceBasicHandler(orderRepository, logTrace));

    return proxy;
  }

}
~~~

~~~java
@Import(DynamicProxyBasicConfig.class)
@SpringBootApplication(scanBasePackages = "hello.proxy.app") //주의
public class ProxyApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProxyApplication.class, args);
	}

	@Bean
	public LogTrace logTrace() {
		return new ThreadLocalLogTrace();
	}

}
~~~

코드 수정 후 v1-controller를 실행하면 정상적으로 작동하는 것을 확인할 수 있다.
~~~
[3047c634] OrderControllerV1.request()
[3047c634] |-->OrderServiceV1.orderItem()
[3047c634] |   |-->OrderRepositoryV1.save()
[3047c634] |   |<--OrderRepositoryV1.save() time=1005ms
[3047c634] |<--OrderServiceV1.orderItem() time=1007ms
[3047c634] OrderControllerV1.request() time=1008ms
~~~

그림으로 보면 아래와 같다.

<img width="752" alt="image" src="https://github.com/user-attachments/assets/b04581c0-4264-43f6-a9af-eb52c27ac9a9">

<img width="756" alt="image" src="https://github.com/user-attachments/assets/57a7c81f-9348-4d78-a7b7-1f043a305924">

<img width="761" alt="image" src="https://github.com/user-attachments/assets/fcd36fea-c26d-41d2-8784-a8e10d2b15e1">

그림이 자세하기 때문에 그림을 통해 의존 관계를 파악하고 어떻게 돌아가는지 알기 쉽기 때문에 코멘트는 하지 않겠다.