# 스프링 핵심 원리 - 고급편 > @Aspect 프록시 - 적용
스프링 애플리케이션에 프록시를 적용하려면 포인트컷과 어드바이스로 구성된 어드바이저를 만들어서 스프링 빈으로 등록하면 된다.
그럼 나머지는 자동 프록시 생성기가 처리해준다. 자동 프록시 생성기는 스프링 빈으로 등록된 어드바이저들을 찾고, 스프링 빈들에 자동으로 프록시를 적용해준다. (당연히 포인트컷에 매치가 되어야 한다.)

스프링은 `@Aspect` 애노테이션으로 매우 편리하게 포인트컷과 어드바이스로 구성되어 있는 어드바이저 생성 기능을 지원한다.

> **참고:** `@Aspect`는 관점 지향 프로그래밍(AOP)을 가능하게 하는 AspectJ 프로젝트에서 제공하는 어노테이션이다. 스프링은 이것을 차용해서 프록시를 통한 AOP를 가능하게 한다. AOP와 AspectJ 관련된 자세한 내용은 다음에 설명한다. 지금은 프록시에 초점을 맞추자. 우선 이 어노테이션을 사용해서 스프링이 편리하게 프록시를 만들어준다고 생각하면 된다.

```java
@Slf4j
@Aspect
public class LogTraceAspect {
  private final LogTrace logTrace;

  public LogTraceAspect(LogTrace logTrace) {
    this.logTrace = logTrace;
  }

  @Around("execution(* hello.proxy.app..*(..))") //포인트 컷
  public Object execute(ProceedingJoinPoint joinPoint) throws Throwable { // 어드바이스
    TraceStatus status = null;

    log.info("target = {}", joinPoint.getTarget()); // 실제 호출 대상
    log.info("getArgs = {}", joinPoint.getArgs()); // 전달 인자
    log.info("getSignature = {}", joinPoint.getSignature()); // join point 시그니처

    try {
      String message = joinPoint.getSignature().toShortString();
      status = logTrace.begin(message);

      // 실제 로직 호출
      Object result = joinPoint.proceed();

      logTrace.end(status);
      return result;
    } catch (Exception e) {
      logTrace.exception(status, e);
      throw e;
    }
  }
}
```
- `@Aspect`: 애노테이션 기반 프록시를 적용할 때 필요하다.
- `@Around("execution(* hello.proxy.app..*(..))")`
  - `@Around`의 값에 포인트컷 표현식을 넣는다. 표현식은 AspectJ 표현식을 사용한다.
  - `@Around`의 메서드는 어드바이스(`Advice`)가 된다.
- `ProceedingJoinPoint joinPoint`: 어드바이스에서 살펴본 `MethodInvocation invocation` 과 유사한 기능이다. 내부에 실제 호출 대상, 전달 인자, 그리고 어떤 객체와 어떤 메서드가 호출되었는지 정보가 포함되어 있다.
- `joinPoint.proceed()`: 실제 호출 대상(`target`)을 호출한다.

```java
@Configuration
@Import({AppConfigV1.class, AppConfigV2.class})
public class AopConfig {
  
  @Bean
  public LogTraceAspect logTraceAspect(LogTrace logTrace) {
    return new LogTraceAspect(logTrace);
  }

}
```
- `@Import({AppV1Config.class, AppV2Config.class})`: V1, V2 애플리케이션은 수동으로 스프링 빈으로 등록해야 동작한다.
- `@Bean logTraceAspect()`: `@Aspect` 가 있어도 스프링 빈으로 등록을 해줘야 한다. 물론 `LogTraceAspect`에 `@Component` 애노테이션을 붙여서 컴포넌트 스캔을 사용해서 스프링 빈으로 등록해도 된다.

```java
@Import(AopConfig.class)
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
```

작동을 시키면 로그를 확인할 수 있다.
실제 로그 내용은 아래 형상과 다르나, 가독성을 위해 임의로 순서를 변경했다.
로그로 찍은 target, args, signature의 내용을 확인할 수 있다.
```
target = hello.proxy.app.v1.OrderControllerV1Impl@47c9f6cb
getArgs = hello
getSignature = String hello.proxy.app.v1.OrderControllerV1Impl.request(String)

target = hello.proxy.app.v1.OrderServiceV1Impl@1528d6a4
getArgs = hello
getSignature = void hello.proxy.app.v1.OrderServiceV1Impl.orderItem(String)

target = hello.proxy.app.v1.OrderRepositoryV1Impl@35da47df
getArgs = hello
getSignature = void hello.proxy.app.v1.OrderRepositoryV1Impl.save(String)

[94890137] OrderControllerV1Impl.request(..)
[94890137] |-->OrderServiceV1Impl.orderItem(..)
[94890137] |   |-->OrderRepositoryV1Impl.save(..)
[94890137] |   |<--OrderRepositoryV1Impl.save(..) time=1006ms
[94890137] |<--OrderServiceV1Impl.orderItem(..) time=1008ms
[94890137] OrderControllerV1Impl.request(..) time=1010ms
```


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__