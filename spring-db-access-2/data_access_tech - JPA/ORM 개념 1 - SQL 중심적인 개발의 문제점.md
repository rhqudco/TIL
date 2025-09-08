SQL 중심적인 개발의 문제점은 테이블은 상속 관계가 없기 때문에 애플리케이션 로직 상에서 객체들 사이 상속 관계를 표현하기 매우 어렵다는 점이다.
반복적으로 SQL문들을 계속 작성해줘야 하며, `Item`이라는 객체가 있다고 가정할 경우 해당 객체 하위에 있는 `Book`, `Album`과 같은 하위 타입의 객체를 저장하려면 객체를 분해하여 `Item` 테이블도 INSERT 쿼리를 작성해야 하며, Book 혹은 Album 테이블도 INSERT 쿼리를 작성해야 하는 문제가 있다.

또한 연관관계에서도 차이가 있다.
객체는 참조를 사용하여 만약 `member`가 있을 경우 `member.getTeam()`을 통해 값을 가져올 수 있지만, 테이블은 외래 키를 사용하기 때문에 `JOIN ON M.TEAM_ID = T.TEAM_ID`과 같이 Join이 필요하다.

만약 객체를 테이블에 맞춰 모델링을 한다면
```java
class Member {
	String id; // MEMBER_ID
	Long teamId; // TEAM_ID FK 컬럼 사용
	String username; // USERNAME 컬럼 사용
}

class Team {
	Long id; // TEAM_ID PK 사용
	String name; // NAME 컬럼 사용
}
```

위와 같은 형상이겠지만, 객체는 참조가 가능하기 때문에 테이블을 객체에 맞춰 모델링한다면 아래와 같은 형상이 나올 수 있다.

```java
class Member {
	String id; // MEMBER_ID
	Team team; // 참조로 연관관계를 맺는다.
	String username; // USERNAME 컬럼 사용

	Team getTeam(){
		return team;
	}
}

class Team {
	Long id; // TEAM_ID PK 사용
	String name; // NAME 컬럼 사용
}
```

하지만, 이렇게 해도 객체에 테이블을 맞췄기 때문에 저장, 조회 시 복잡한 쿼리가 발생한다.
또한, 객체는 연관된 객체들을 탐색할 수 있어야 하는데 조회 시 연관된 객체들을 SQL을 통해 모두 조회하지 않으면 자유롭게 연관 객체들을 탐색할 수 없다.
그렇다고, 상황에 따라 역할 별로 메서드를 분리해서 조회 메서드를 작성하기엔 반복 작업이 너무 많아지게 된다.
이는 엔티티의 신뢰를 저하시키는 문제를 발생한다.
진정한 의미의 계층 분할이 어려워진다.
객체답게 모델링 할 수록 매핑 작업, 반복 작업이 늘어나게 되는 문제가 있는데
이를 해결하기 위해 객체를 자바 컬렉션에 저장하듯, DB에 저장하는 JPA가 나오게 됐다.


__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__