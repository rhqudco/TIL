#### execution 문법
```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)throws-pattern?)
-> 
execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
```
- 메소드 실행 조인 포인트를 매칭한다.
- `?`가 붙어 있는 패턴은 생략 가능
- `*`같은 패턴을 지정할 수 있다.

#### 가장 정확한 포인트컷
먼저 `MemberServiceImpl.hello(String)` 메서드와 가장 정확하게 모든 내용이 매칭되는 표현식이다.

```java
@Test  
void exactMatch() {  
  //public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)  
  pointcut.setExpression("execution(public String hello.aop.member.MemberServiceImpl.hello(String))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
- `AspectJExpressionPointcut`에 `pointcut.setExpression`을 통해서 포인트컷 표현식을 적용할 수 있다.
- `pointcut.matches(메서드, 대상 클래스)`를 실행하면 지정한 포인트컷 표현식의 매칭 여부를 `true`, `false`로 반환한다.

__매칭 조건__
- `접근 제어자?`: `public`
- `반환 타입`: `String`
- `선언 타입?`: `hello.aop.member.MemberServiceImpl`
- `메서드 이름`: `hello`
- `파라미터`: `(String)`
- `예외?`: `생략`

`MemberServiceImpl.hello(String)` 메서드와 포인트컷 표현식의 모든 내용이 정확하게 일치하기 때문에 `true`를 반환한다.

#### 가장 많이 생략한 포인트컷
```java
@Test  
void allMatch() {  
  pointcut.setExpression("execution(* *(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```

__매칭 조건__
- `접근 제어자?`: `생략`
- `반환 타입`: `*`
- `선언 타입?`: `생략`
- `메서드 이름`: `*`
- `파라미터`: `(..)`
- `예외?`: 없음

- `*`은 아무 값이 들어와도 된다는 뜻
- 파라미터에서 `..`은 파라미터 타입과 파라미터 수가 상관 없다는 뜻이다. `(0..*)` 파라미터는 나중에 설명

#### 메서드 이름 매칭 관련 포인트컷
```java
@Test  
void nameMatch() {  
  pointcut.setExpression("execution(* hello(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void nameMatchStar1() {  
  pointcut.setExpression("execution(* hel*(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void nameMatchStar2() {  
  pointcut.setExpression("execution(* *el*(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void nameMatchFalse() {  
  pointcut.setExpression("execution(* nono(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();  
}
```

#### 패키지 매칭 관련 포인트컷
```java
@Test  
void packageExactMatch1() {  
  pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.hello(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
@Test  
void packageExactMatch2() {  
  pointcut.setExpression("execution(* hello.aop.member.*.*(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
@Test  
void packageExactMatchFalse() {  
  pointcut.setExpression("execution(* hello.aop.*.*(..))"); // hello.aop 패키지 하위에 * 클래스의 * 메서드로 매칭
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();  
}  
@Test  
void packageMatchSubPackage1() {  
  pointcut.setExpression("execution(* hello.aop.member..*.*(..))"); // hello.aop 패키지 하위의 모든 패키지와 클래스로 매칭이 됨
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
@Test  
void packageMatchSubPackage2() {  
  pointcut.setExpression("execution(* hello.aop..*.*(..))");  
  assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```

- `hello.aop.member.*(1).*(2)`
	- (1): 타입
	- (2): 메서드 이름
- 패키지에서 `.`과 `..`의 차이를 이해해야 한다.
	- `.`: 정확하게 해당 위치의 패키지
	- `..`: 해당 위치의 패키지와 . 그 하위 패키지도 포함


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__