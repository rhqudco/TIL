스프링과 JPA는 자바 엔터프라이즈(기업) 시장의 주력 기술이다.  
스프링이 DI 컨테이너를 포함한 애플리케이션 전반의 다양한 기능을 제공한다면, JPA는 ORM 데이터 접근 기술을 제공한다.

## JPA 설정
`spring-boot-starter-data-jpa` 라이브러리를 사용하면 JPA와 스프링 데이터 JPA를 스프링 부트와 통합하고, 설정도 아주 간단히 할 수 있다.

`spring-boot-starter-data-jpa` 라이브러리를 사용해서 간단히 설정하는 방법을 알아보자.

`build.gradle`에 다음 의존관계를 추가한다.
```
//JPA, 스프링 데이터 JPA 추가  
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

`build.gradle`에 다음 의존 관계를 제거한다.
```
//JdbcTemplate 추가  
//implementation 'org.springframework.boot:spring-boot-starter-jdbc'
```

`spring-boot-starter-data-jpa`는 `spring-boot-starter-jdbc`도 함께 포함(의존)한다.
따라서 해당 라이브러리 의존관계를 제거해도 된다.
참고로 `mybatis-spring-boot-starter` 도 `spring-boot-starter-jdbc`를 포함하기 때문에 제거해도 된다.

다음과 같은 라이브러리가 추가된다.  
- `hibernate-core`: JPA 구현체인 하이버네이트 라이브러리
- `jakarta.persistence-api`: JPA 인터페이스
- `spring-data-jpa`: 스프링 데이터 JPA 라이브러리

`application.properties`에 다음 설정을 추가하자. (main, test)
```
# JPA log  
logging.level.org.hibernate.SQL=DEBUG  
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```
- `org.hibernate.SQL=DEBUG`
	- 하이버네이트가 생성하고 실행하는 SQL을 확인할 수 있다.
- `org.hibernate.type.descriptor.sql.BasicBinder=TRACE`
	- SQL에 바인딩 되는 파라미터를 확인할 수 있다.
- `spring.jpa.show-sql=true`
	- 이런 설정도 있다.
	- 이전 설정은 `logger`를 통해서 SQL이 출력된다.
	- 이 설정은 `System.out` 콘솔을 통해서 SQL이 출력된다. (권장하지 않음)

**스프링 부트 3.0**
스프링 부트 3.0 이상을 사용하면 하이버네이트 6 버전이 사용되는데, 로그 설정 방식이 달려졌다. 다음과 같이 로그를 설정해야 한다. 

```properties
#JPA log
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.orm.jdbc.bind=TRACE
```



__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__