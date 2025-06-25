## 재시도 AOP
좀 더 의미있는 재시도 AOP를 만들어보자.
`@Retry` 애노테이션이 있으면 예외가 발생했을 때 다시 시도해서 문제를 복구한다.

__@Retry__
```java
package hello.aop.exam.annotation;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Retry {  
  int value() default 3;  
}
```

이 애노테이션에는 재시도 횟수로 사용할 값이 있다. 기본값으로 `3`을 사용한다.

__RetryAspect__
```java
package hello.aop.exam.aop;  
  
import hello.aop.exam.annotation.Retry;  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
  
@Slf4j  
@Aspect  
public class RetryAspect {  
    
  @Around("@annotation(retry)")  
  public Object doRetry(ProceedingJoinPoint joinPoint, Retry retry) throws Throwable {  
    log.info("[retry] {} retry={}", joinPoint.getSignature(), retry);  
      
    int maxRetry = retry.value();  
    Exception exceptionHolder = null;  
      
    for (int retryCount = 1; retryCount <= maxRetry; retryCount++) {  
      try {  
        log.info("[retry] try count= {}/{}", retryCount, maxRetry);  
        return joinPoint.proceed();  
      } catch (Exception e) {  
        exceptionHolder = e;  
      }  
    }  
    throw exceptionHolder;  
  }  
  
}
```
- 재시도하는 애스팩트
- `@annotation(retry)`, `Retry retry`를 사용해서 어드바이스에 애노테이션을 파라미터로 전달한다.
	- Retry 정보를 파라미터로 전달하는 것 (이미 import된 Retry가 있음)
- `retry.value()`를 통해서 애노테이션에 지정한 값을 가져올 수 있다.
- 예외가 발생해서 결과가 정상 반환되지 않으면 `retry.value()`만큼 재시도한다.

__ExamRepository - @Retry 추가__
```java
package hello.aop.exam;  
  
import hello.aop.exam.annotation.Retry;  
import hello.aop.exam.annotation.Trace;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public class ExamRepository {  
  
  private static int seq = 0;  
  
  /**  
  * 5번에 1번 실패하는 요청  */  
  @Trace  
  @Retry(value = 4)  
  public String save(String itemId) {  
    seq++;  
    if (seq % 5 == 0) {  
      throw new IllegalStateException("예외 발생");  
    }  
    return "ok";  
  }  
  
}
```

`ExamRepository.save()`메서드에 `@Retry(value = 4)`를 적용했다. 이 메서드에서 문제가 발생하면 4번 재시도 한다.

__ExamTest - 추가__
```java
@SpringBootTest  
//@Import(TraceAspect.class)  
@Import({TraceAspect.class, RetryAspect.class})  
public class ExamTest {
	...
}
```

__실행 결과__
```
[trace] void hello.aop.exam.ExamService.request(String) args = [data0]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data0]
[retry] String hello.aop.exam.ExamRepository.save(String) retry=@hello.aop.exam.annotation.Retry(4)
[retry] try count= 1/4

[trace] void hello.aop.exam.ExamService.request(String) args = [data1]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data1]
[retry] String hello.aop.exam.ExamRepository.save(String) retry=@hello.aop.exam.annotation.Retry(4)
[retry] try count= 1/4

[trace] void hello.aop.exam.ExamService.request(String) args = [data2]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data2]
[retry] String hello.aop.exam.ExamRepository.save(String) retry=@hello.aop.exam.annotation.Retry(4)
[retry] try count= 1/4

[trace] void hello.aop.exam.ExamService.request(String) args = [data3]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data3]
[retry] String hello.aop.exam.ExamRepository.save(String) retry=@hello.aop.exam.annotation.Retry(4)
[retry] try count= 1/4

[trace] void hello.aop.exam.ExamService.request(String) args = [data4]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data4]
[retry] String hello.aop.exam.ExamRepository.save(String) retry=@hello.aop.exam.annotation.Retry(4)
[retry] try count= 1/4
[retry] try count= 2/4
```

> 참고
> `retry`로 인한 성공 시 `trace` 로그가 찍히지 않는 이유는, 스프링 AOP의 `joinPoint.proceed()`의 호출 구조 때문이다.
> `joinPoint.proceed()`는 하나의 호출 안에서 체이닝하기 때문에 한 번만 실행하도록 설계되어 있다.
> 즉, `proceed()`를 여러 번 호출해도, 첫 번째 `proceed()` 호출 때 이미 체이닝 내부에서 `@Before`가 소진되었고 타겟 메서드만 실행되기 때문이다.




__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__