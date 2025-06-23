## within
`within` 지시자는 특정 타입 내의 조인 포인트들로 매칭을 제한한다. 쉽게 말하자면 해당 타입이 매칭되면 그 안의 메서드(조인 포인트)들이 자동으로 매칭된다.
문법은 단순한데 `execution`에서 타입 부분만 사용한다고 보면 된다.

```java
@Test  
void withinExact() {  
  pointcut.setExpression("within(hello.aop.member.MemberServiceImpl)");  
  Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void withinStar() {  
  pointcut.setExpression("within(hello.aop.member.*Service*)");  
  Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void withinSubPackage() {  
  pointcut.setExpression("within(hello.aop..*)");  
  Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
코드를 보면 이해하는 데 어려움 없을 것이다.

#### 주의
`within` 사용 시 주의점이 있다. 표현식에 부모 타입을 지정하면 안된다는 점이다.
정확하게 타입이 맞아야 한다. 이 부분에서 `execution`과 차이가 난다.

```java
@Test  
@DisplayName("타켓의 타입에만 직접 적용, 인터페이스를 선정하면 안된다.")  
void withinSuperTypeFalse() {  
  pointcut.setExpression("within(hello.aop.member.MemberService)");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();  
}  
  
@Test  
@DisplayName("execution은 타입 기반, 인터페이스를 선정 가능.")  
void executionSuperTypeTrue() {  
  pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```

부모 타입(여기서는 `MemberService`인터페이스) 지정시 `within`은 실패하고, `execution`은 성공하는 것을 확인할 수 있다.


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__