# 스프링 핵심 원리 - 고급편 > 하나의 프록시, 여러 Advisor 적용
스프링 빈이 `advisor1`, `advisor2`가 제공하는 포인트컷의 조건을 모두 만족하면 프록시 자동 생성기는 프록시를 몇 개 생성할까?
프록시 자동 생성기는 프록시를 하나만 생선한다. 왜냐면 프록시 팩토리가 생성하는 프록시는 내부에 여러 `advisor`를 포함할 수 있기 때문이다.
따라서 프록시를 여러 개 생성하여 비용을 낭비할 이유가 없다.

**프록시 자동 생성기 상황별 정리**
- `advisor1` 의 포인트컷만 만족 프록시1개 생성, 프록시에 `advisor1` 만 포함
- `advisor1` , `advisor2` 의 포인트컷을 모두 만족 프록시1개 생성, 프록시에 `advisor1` , `advisor2` 모두 포함
- `advisor1` , `advisor2` 의 포인트컷을 모두 만족하지 않음 프록시가 생성되지 않음

그렇다면 아래 경우는 어떻게 될까
1. `A` 객체에 `Advisor1`, `B` 객체에 `Advisor2`
2. `A` 객체에 `Advisor1`, `B` 객체에 `Advisor1`

위 상황은 서로 다른 객체이기 때문에 프록시가 각각 생성된다. 기본적으로 스프링 프록시 전략은 한 객체(스프링 Bean)에 대해서만 하나의 프록시 객체를 생성하기 때문이다. 주의하도록 하자.

<img width="760" alt="image" src="https://github.com/user-attachments/assets/de76510c-37c5-45fb-a972-561bfb701d98">

<img width="764" alt="image" src="https://github.com/user-attachments/assets/2a177a7c-f6a4-4d3c-8d9d-2d1f7870cffe">

# 정리
자동 프록시 생성기인 `AnnotationAwareAspectJAutoProxyCreator` 덕분에 개발자는 매우 편리하게 프록시를 적용할 수 있다. 
이제 `Advisor` 만 스프링 빈으로 등록하면 된다.
`Advisor` = `Pointcut` + `Advice`
