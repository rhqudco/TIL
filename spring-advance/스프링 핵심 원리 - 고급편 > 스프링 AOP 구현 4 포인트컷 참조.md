포인트컷을 외부에 모아놓고 공용으로 사용하는 방법을 알아보자.
다음과 같이 포인트컷을 공용으로 사용하기 위해 별도의 외부 클래스에 모아두어도 된다. 참고로 외부에서 호출할 때는 포인트컷의 접근 제어자를 `public`으로 열어두어야 한다.

__Pointcuts__

```java
package hello.aop.order.aop;  
  
import org.aspectj.lang.annotation.Pointcut;  
  
public class Pointcuts {  
  
  // hello.aop.order 패키지와 하위 패키지  
  @Pointcut("execution(* hello.aop.order..*(..))")  
  public void allOrder() {}  
  
  // 타입 패턴이 *Service  
  @Pointcut("execution(* *..*Service.*(..))")  
  public void allService() {}  
  
  // allOrder && allService  
  @Pointcut("allOrder() && allService()")  
  public void orderAndService() {}  
  
}
```

- `orderAndService()` : `allOrder()` 포인트컷와 `allService()` 포인트컷을 조합해서 새로운 포인트컷을 만들었다.

__AspectV4Pointcut__

```java
package hello.aop.order.aop;  
  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
  
@Slf4j  
@Aspect  
public class AspectV4Pointcut {  
  
  @Around("hello.aop.order.aop.Pointcuts.allOrder()")  
  public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {  
    log.info("[log] {}", joinPoint.getSignature());  
    return joinPoint.proceed();  
  }  
  
  @Around("hello.aop.order.aop.Pointcuts.orderAndService()")  
  public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {  
    try {  
      log.info("[트랜잭션 시작] {}", joinPoint.getSignature());  
      Object result = joinPoint.proceed(); // 실제 타겟이 되는 비즈니스 코드 호출  
      log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());  
      return result;  
    } catch (Exception e) {  
      log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());  
      throw e;  
    } finally {  
      log.info("[리소스 릴리즈] {}", joinPoint.getSignature());  
    }  
  }  
    
}
```

사용하는 방법은 패키지명을 포함한 클래스 이름과 포인트컷 시그니처를 모두 지정하면 된다. 
포인트컷을 여러 어드바이스에서 함께 사용할 때 이 방법을 사용하면 효과적이다.

__AopTest - 수정__

```java
@Slf4j  
//@Import(AspectV1.class)  
//@Import(AspectV2.class)  
//@Import(AspectV3.class)  
@Import(AspectV4Pointcut.class)  
@SpringBootTest  
public class AopTest {
	...
}
```

실행 결과는 기존과 같다.

__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__