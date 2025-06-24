## @annotation, @args

### @annotation
- 정의
	- `@annotation`: 메서드가 주어진 애노테이션을 가지고 있는 조인 포인트를 매칭
- 설명
	- `@annotation(hello.aop.member.annotation.MethodAop)`

다음과 같이 메서드(조인 포인트)에 애노테이션이 있으면 매칭한다.
```java
public class MemberServiceImpl {  
  
  @MethodAop("test value")  
  public String hello(String param) {  
    return "ok";  
  }  

}
```

__AtAnnotationTest__
```java
package hello.aop.pointcut;  
  
import hello.aop.member.MemberService;  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.junit.jupiter.api.Test;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.context.SpringBootTest;  
import org.springframework.context.annotation.Import;  
  
@Slf4j  
@Import(AtAnnotationTest.AtAnnotationAspect.class)  
@SpringBootTest  
public class AtAnnotationTest {  
  
  @Autowired  
  MemberService memberService;  
  
  @Test  
  void success() {  
    log.info("memberService Proxy = {}", memberService.getClass());  
    memberService.hello("helloA");  
  }  
  
  @Slf4j  
  @Aspect  static class AtAnnotationAspect {  
  
    @Around("@annotation(hello.aop.member.annotation.MethodAop)")  
    public Object doAtAnnotation(ProceedingJoinPoint joinPoint) throws Throwable {  
      log.info("[@annotation] {}", joinPoint.getSignature());  
      return joinPoint.proceed();  
    }  
  
  }  
  
}
```

__실행 결과__
```
memberService Proxy = class hello.aop.member.MemberServiceImpl$$SpringCGLIB$$0
[@annotation] String hello.aop.member.MemberServiceImpl.hello(String)
```

### @args
- 정의
	- `@args`: 전달된 실제 인수의 런타임 타입이 주어진 타입의 애노테이션을 갖는 조인 포인트
- 설명
	- 전달된 인수의 런타임 타입에 `@Check` 애노테이션이 있는 경우 매칭한다.
		- `@args(test.Check)`

__AtArgsTest__
```java
package hello.aop.pointcut;  
  
import hello.aop.member.MemberDTO;  
import hello.aop.member.MemberServiceImpl;  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.junit.jupiter.api.Test;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.context.SpringBootTest;  
import org.springframework.context.annotation.Import;  
  
@Slf4j  
@Import(AtArgsTest.AtArgsAspect.class)  
@SpringBootTest  
public class AtArgsTest {  
  
  @Autowired  
  MemberServiceImpl memberServiceImpl;  
  
  @Test  
  void success() {  
    log.info("memberServiceImpl Proxy = {}", memberServiceImpl.getClass());  
    MemberDTO test = new MemberDTO("test");  
    memberServiceImpl.external(test);  
  }  
  
  @Slf4j  
  @Aspect  static class AtArgsAspect {  
  
    @Around("execution(* hello.aop.member..*(..)) && @args(hello.aop.member.annotation.ClassAop)")  
    public Object doAtArgs(ProceedingJoinPoint joinPoint) throws Throwable {  
      log.info("[@args] {}", joinPoint.getSignature());  
      return joinPoint.proceed();  
    }  
  
  }  
  
}
```

__MemberServiceImpl__
```java
package hello.aop.member;  
  
import hello.aop.member.annotation.ClassAop;  
import hello.aop.member.annotation.MethodAop;  
import org.springframework.stereotype.Component;  
  
@ClassAop  
@Component  
public class MemberServiceImpl implements MemberService {  
  
  @Override  
  @MethodAop("test value")  
  public String hello(String param) {  
    return "ok";  
  }  
  
  public String internal(String param) {  
    return "ok";  
  }  
  
  public String external(MemberDTO memberDto) {  
    return "ok";  
  }  
}
```

__MemberDTO__
```java
package hello.aop.member;  
  
import hello.aop.member.annotation.ClassAop;  
  
@ClassAop  
public class MemberDTO {  
  
  private String name;  
  
  public MemberDTO(String name) {  
    this.name = name;  
  }  
  public String getName() {  
    return name;  
  }  
  
  public void setName(String name) {  
    this.name = name;  
  }  
  
}
```

__ClassAOP__
```java
package hello.aop.member.annotation;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface ClassAop {  
  
}
```

__실행 결과__
```
memberServiceImpl Proxy = class hello.aop.member.MemberServiceImpl$$SpringCGLIB$$0
[@args] String hello.aop.member.MemberServiceImpl.external(MemberDTO)
```

s
__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__