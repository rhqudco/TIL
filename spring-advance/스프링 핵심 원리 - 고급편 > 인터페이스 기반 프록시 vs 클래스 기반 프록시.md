# 스프링 핵심 원리 - 고급편 > 인터페이스 기반 프록시 vs 클래스 기반 프록시
- 인터페이스가 없어도 클래스 기반으로 프록시를 생성할 수 있는 것을 확인했다.
- 클래스 기반 프록시는 해당 클래스에만 적용할 수 있다.
- 인터페이스 기반 프록시는 인터페이스만 같으면 모든 곳에 적용이 가능하다.
- 클래스 기반 프록시는 상속을 사용하지 못하기 때문에 Java 기본 문법에 존재하는 몇 가지 제약이 존재한다.
  - 부모 클래스의 생성자를 호출해야 한다. (super(null))
  - 클래스에 final이 붙은 경우 상속이 불가능하다.
  - 메서드에 final이 붙은 경우 메서드 오버라이딩이 불가능하다.

이렇게 보면 인터페이스를 사용한 프록시가 무조건 좋아 보이지만, 인터페이스를 생성이 강제되는 것 자체가 단점이 된다.
인터페이스가 없다면 인터페이스 기반 프록시를 만들 수가 없다.
인터페이스를 도입하는 다양한 이유들이 있겠지만, 인터페이스 도입이 효과적인 경우는 변경 가능성이 있을 때 인데 변경할 일이 없는 코드에도 인터페이스를 도입해야 인터페이스 기반 프록시를 적용할 수 있다는 것이다.
이럴 때는 실용적 관점에서 클래스 프록시를 도입하면 된다.

핵심은 항상 필요하지 않은 인터페이스를 프록시 적용을 위해 억지로 도입할 필요가 없다는 것이다.

지금까지 프록시를 사용해서 기존 코드를 변경하지 않고, 로그 추적기라는 부가 기능을 적용할 수 있었다.
그런데 문제는 프록시 클래스를 너무 많이 만들어야 한다는 점이다.
잘 보면 프록시 클래스가 하는 일은 `LogTrace` 를 사용하는 것인데, 그 로직이 모두 똑같다.대상 클래스만 다를 뿐이다. 
만약 적용해야 하는 대상 클래스가 100개라면 프록시 클래스도 100개를 만들어야 한다.
프록시 클래스를 하나만 만들어서 모든 곳에 적용하는 방법은 동적 프록시 기술이 이 문제를 해결해준다.


__출처: 김영한 지식공유자의 스프링 핵심 원리 고급편__