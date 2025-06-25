## 로그 출력 AOP
먼저 로그 출력용 AOP를 만들어보자.
`@Trace` 가 메서드에 붙어 있응면 호출 정보가 출력되는 편리한 기능이다.

__@Trace__
```java
package hello.aop.exam.annotation;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Trace {  
  
}
```

__TraceAspect__
```java
package hello.aop.exam.aop;  
  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.JoinPoint;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Before;  
  
@Slf4j  
@Aspect  
public class TraceAspect {  
    
  @Before("@annotation(hello.aop.exam.annotation.Trace)")  
  public void doTrace(JoinPoint joinPoint) {  
    Object[] args = joinPoint.getArgs();  
    log.info("[trace] {} args = {}", joinPoint.getSignature(), args);  
  }  
  
}
```

`@annotation(hello.aop.exam.annotation.Trace)` 포인트컷을 사용해서 `@Trace`가 붙은 메서드에 어드 바이스를 적용한다.

__ExamService - @Trace 추가__
```java
package hello.aop.exam;  
  
import hello.aop.exam.annotation.Trace;  
import lombok.RequiredArgsConstructor;  
import org.springframework.stereotype.Service;  
  
@Service  
@RequiredArgsConstructor  
public class ExamService {  
  
  private final ExamRepository examRepository;  
  
  @Trace  
  public void request(String itemId) {  
    examRepository.save(itemId);  
  }  
  
}
```

`request()`에 `@Trace`를 붙였다. 이제 메서드 호출 정보를 AOP를 사용해서 로그로 남길 수 있다.

__ExamRepository - @Trace 추가__
```java
package hello.aop.exam;  
  
import hello.aop.exam.annotation.Trace;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public class ExamRepository {  
  
  private static int seq = 0;  
  
  /**  
  * 5번에 1번 실패하는 요청  
  * */  
  @Trace  
  public String save(String itemId) {  
    seq++;  
    if (seq % 5 == 0) {  
      throw new IllegalStateException("예외 발생");  
    }  
    return "ok";  
  }  
  
}
```

`save()`에 `@Trace`를 붙였다.


__ExamTest - 추가__
```java
@SpringBootTest  
@Import(TraceAspect.class)  
public class ExamTest {
	...
}
```

`@Import(TraceAspect.class)`를 사용해서 `TraceAspect`를 스프링 빈으로 추가하자. 이제 애스펙트가 적용 된다.

__실행 결과__
```
[trace] void hello.aop.exam.ExamService.request(String) args = [data0]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data0]
[trace] void hello.aop.exam.ExamService.request(String) args = [data1]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data1]
[trace] void hello.aop.exam.ExamService.request(String) args = [data2]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data2]
[trace] void hello.aop.exam.ExamService.request(String) args = [data3]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data3]
[trace] void hello.aop.exam.ExamService.request(String) args = [data4]
[trace] String hello.aop.exam.ExamRepository.save(String) args = [data4]
```

실행해보면 `@Trace`가 붙은 `request()`, `save()` 호출 시 로그가 잘 남는 것을 확인할 수 있다.


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__