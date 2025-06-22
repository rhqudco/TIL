#### 타입 매칭 - 부모 타입 허용

```java
@Test  
void typeExactMatch() {  
  pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.*(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}

@Test  
void typeMatchSuperType() {  
  pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```

`typeExactMatch()`는 타입 정보가 정확하게 일치하기 때문에 매칭된다.
`typeMatchSuperType()`을 주의해서 보아야 한다. `execution`에서는 `MemberService`처럼 부모 타입을 선언해도 그 자식 타입은 매칭된다. 다형성에서 `부모타입 = 자식타입`이 할당 가능하다는 점을 떠올리자.

#### 타입 매칭 - 부모 타입에 있는 매서드만 허용
```java
@Test  
void typeMatchInternal() throws NoSuchMethodException {  
  pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.*(..))");  
  Method internalMethod = MemberServiceImpl.class.getMethod("internal", String.class);  
  assertThat(pointcut.matches(internalMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void typeMatchNoSuperTypeMethodFalse() throws NoSuchMethodException {  
  pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");  
  Method internalMethod = MemberServiceImpl.class.getMethod("internal", String.class);  
  assertThat(pointcut.matches(internalMethod, MemberServiceImpl.class)).isFalse();  
}
```

`typeMatchInternal()`의 경우 `MemberServiceImpl`를 표현식에 선언했기 때문에 그 안에 있는 `internal(String)` 메서드도 매칭 대상이 된다.  

하지만, `typeMatchNoSuperTypeMethodFalse()`를 주의해서 보아야 한다.
이 경우 표현식에 부모 타입인 `MemberService`를 선언했다. 그런데 자식 타입인 `MemberServiceImpl`의 `internal(String)` 메서드를 매칭하려 한다.
이 경우 매칭에 실패한다. `MemberService`에는 `internal(String)` 메서드가 없다

부모 타입을 표현식에 선언한 경우 부모 타입에서 선언한 메서드가 자식 타입에 있어야 매칭에 성공한다. 그래서 부모 타입에 있는 `hello(String)` 메서드는 매칭에 성공하지만, 부모 타입에 없는 `internal(String)`는 매칭에 실패한다.


#### 파라미터 매칭
```java
// String 타입의 파라미터 허용  
// (String)  
@Test  
void argsMatch() {  
  pointcut.setExpression("execution(* *(String))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
// 파라미터가 없어야 함  
// ()  
@Test  
void argsMatchNoArgs() {  
  pointcut.setExpression("execution(* *())");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();  
}  
  
// 정확히 하나의 파라미터 허용, 모든 타입 허용  
// (X xx)  
@Test  
void argsMatchStar() {  
  pointcut.setExpression("execution(* *(*))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
// 숫자와 무관하게 모든 파라미터, 모든 타입 허용  
// 파라미터가 없어도 됨  
// (), (X xx) (X xx, C cc)  
@Test  
void argsMatchAll() {  
  pointcut.setExpression("execution(* *(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
// String 타입으로 시작, 숫자와 무관하게 모든 파라미터, 모든 타입 허용  
// (String), (String, Xxx), (String, X xx, C cc) 허용  
@Test  
void argsMatchComplex() {  
  pointcut.setExpression("execution(* *(String, ..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```

- execution 파라미터 매칭 규칙
	- `(String)`: 정확하게 String 타입의 파라미터
	- `()`: 파라미터가 없어야 한다.
	- `(*)`: 정확히 하나의 파라미터, 단 모든 타입을 허용
	- `(*, *)`: 정확히 두 개의 파라미터, 단 모든 타입을 허용
	- `(..)`: 숫자와 무관한 모든 파라미터, 모든 타입을 허용. 파라미터가 없어도 된다. `(0..*)`로 이해하면 된다.
	- `(String, ..)`: String 타입으로 시작해야 한다. 숫자와 무관하게 모든 파라미터, 모든 타입을 허용
		- 예: `(String)`, `(String, Xxx)`, `(String, Xxx, Xxx)` 허용

__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__