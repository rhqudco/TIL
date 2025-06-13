
![[./images/Pasted_image_20250611214532.png]]

#### 조인 포인트(Join Point)
- 어드바이스가 적용될 수 있는 위치, 메서드 실행, 생성자 호출, 필드 값 접근, static 메서드 접근 같은 `프로그램 실행 중 지점`
- 조인 포인트는 추상적인 개념
	- AOP를 적용할 수 있는 모든 지점이라고 생각하면 된다.
- 스프링 AOP는 프록시 방식을 사용하므로 조인 포인트는 항상 __메서드 실행 지점__ 으로 제한된다

#### 포인트컷(Pointcut)
- 조인 포인트 중에서 어드바이스가 적용될 위치를 선별하는 기능
- 주로 AspectJ 표현식을 사용해서 지정
	- `@Pointcut("execution(* hello.aop.order..*(..))")`
- 프록시를 사용하는 스프링 AOP에서는 메서드 실행 지점만 포인트컷으로 선별 가능

#### 타겟(Target)
- 어드바이스를 받는 객체, 포인트컷을 통해 결정
	- 실제 어드바이스가 적용 받는 객체

#### 어드바이스(Advice)
- 부가 기능
- 특정 조인 포인트에서 Aspect에 의해 취해지는 조치
- Around(주변), Before(전), After(후)와 같은 다양한 종류의 어드바이스 존재

#### 애스팩트(Aspect)
- 어드바이스 + 포인트컷을 모듈화 한 것
- `@Aspect`를 생각하면 됨
- 여러 어드바이스와 포인트 컷이 함께 존재

#### 어드바이저(Advisor)
- 하나의 어드바이스와 하나의 포인트컷으로 구성
- 스프링 AOP에서만 사용되는 특별한 용어

#### 위빙(Weaving)
- 포인트컷으로 결정한 타겟의 조인 포인트에 어드바이스를 적용하는 것
- 위빙을 통해 핵심 기능 코드에 영향을 주지 않고 부가 기능을 추가 할 수 있음
- AOP 적용을 위해 애스펙트를 객체에 연결한 상태
	- 컴파일 타임(AspectJ Compiler)
	- 로드 타임
	- 런타임, 스프링 AOP는 런타임, 프록시 방식

#### AOP 프록시
- AOP 기능을 구현하기 위해 만든 프록시 객체, 스프링에서 AOP 프록시는 JDK 동적 프록시 또는 CGLIB 프록시이다.


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__
