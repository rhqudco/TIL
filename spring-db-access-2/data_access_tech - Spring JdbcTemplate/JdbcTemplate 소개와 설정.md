SQL을 직접 사용하는 경우에 스프링이 제공하는 JdbcTemplate은 아주 좋은 선택지다. JdbcTemplate은 JDBC를 매우 편리하게 사용할 수 있게 도와준다.

**장점**
- 설정의 편리함
	- JdbcTemplate `spring-jdbc` 라이브러리에 포함되어 있는데, 이 라이브러리는 스프링으로 JDBC를 사용할 때 기본으로 사용되는 라이브러리이다.
		- 별도의 설정 없이 바로 사용할 수 있다.
- 반복 문제 해결
	- JdbcTemplate은 템플릿 콜백 패턴을 사용해서, JDBC를 직접 사용할 때 발생하는 대부분의 반복 작업을 대신 처리해준다.
	- 개발자는  SQL을 작성하고, 전달할 파라미터를 정의하고 응답 값을 매핑하기만 하면 된다.
	- 우리가 생각할 수 있는 대부분의 반복 작업을 대신 처리해준다.
		- 커넥션 획득
		- `statement`를 준비하고 실행
		- 결과를 반복하도록 루프를 실행
		- 커넥션 종료, `statement`, `resultset`종료
		- 트랜잭션 다루기 위한 커넥션 동기화
		- 예외 발생 시 스프링 예외 변환기 실행

**단점**
- 동적 SQL을 해결하기 어렵다.

이제 직접 JdbcTemplate을 설정하고 적용하면서 이해해보자.

## JdbcTemplate 설정
`build.gradle`

```
//JdbcTemplate 추가  
implementation 'org.springframework.boot:spring-boot-starter-jdbc'  
//H2 데이터베이스 추가  
runtimeOnly 'com.h2database:h2'
```
- `org.springframework.boot:spring-boot-starter-jdbc`를 추가하면 JdbcTemplate이 들어있는 `spring-jdbc`가 라이브러리에 포함된다.
- 여기서는 H2 데이터베이스에 접속해야 하기 때문에 H2 데이터베이스의 클라이언트 라이브러리(Jdbc Driver) 도 추가하자.
	- `runtimeOnly 'com.h2database:h2'`

JdbcTemplate은 `spring-jdbc`라이브러리만 추가하면 된다. 별도의 추가 설정 과정은 없다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 1편__