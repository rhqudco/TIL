## Bean
- 정의
	- `bean`: 스프링 전용 포인트컷 지시자, 빈의 이름으로 지정한다.
- 설명
	- 스프링 빈의 이름으로 AOP 적용 여부 지정
		- 이것은 스프링에서만 사용할 수 있는 특별한 지시자
	- `bean(orderService) || bean(*Repository)`
	- `*`과 같은 패턴 사용 가능

```java
package hello.aop.pointcut;  
  
import hello.aop.order.OrderService;  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.junit.jupiter.api.Test;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.context.SpringBootTest;  
import org.springframework.context.annotation.Import;  
  
@Slf4j  
@Import(BeanTest.BeanAspect.class)  
@SpringBootTest  
public class BeanTest {  
  
  @Autowired  
  OrderService orderService;  
  
  @Test  
  void success() {  
    orderService.orderItem("itemA");  
  }  
  
  @Aspect  
  static class BeanAspect {  
    @Around("bean(orderService) || bean(*Repository)")  
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {  
      log.info("[bean] {}", joinPoint.getSignature());  
      return joinPoint.proceed();  
    }  
  }  
  
}
```

`OrderService`, `*Repository(OrderRepository)`의 메서드에 AOP가 적용된다.

__실행 결과__
```
[bean] void hello.aop.order.OrderService.orderItem(String)
[orderService] 실행
[bean] String hello.aop.order.OrderRepository.save(String)
[orderRepository 실행]
```


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__