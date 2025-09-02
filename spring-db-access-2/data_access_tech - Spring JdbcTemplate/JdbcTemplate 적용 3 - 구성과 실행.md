실제 코드가 동작하도록 구성하고 실행해보자.

**JdbcTemplateV1Config**
```java
package hello.itemservice.config;  
  
import hello.itemservice.repository.ItemRepository;  
import hello.itemservice.repository.jdbctemplate.JdbcTemplateRepositoryV1;  
import hello.itemservice.service.ItemService;  
import hello.itemservice.service.ItemServiceV1;  
import javax.sql.DataSource;  
import lombok.RequiredArgsConstructor;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
@RequiredArgsConstructor  
public class JdbcTemplateV1Config {  
  
  private final DataSource dataSource;  
  
  @Bean  
  public ItemService itemService() {  
    return new ItemServiceV1(itemRepository());  
  }  
  
  @Bean  
  public ItemRepository itemRepository() {  
    return new JdbcTemplateRepositoryV1(dataSource);  
  }  
  
}
```
- `ItemRepository` 구현체로 `JdbcTemplateItemRepositoryV1`이 사용되도록 했다.
	- 이제 메모리 저장소가 아니라 실제 DB에 연결하는 JdbcTemplate이 사용된다.

**ItemServiceApplication - 변경**
```java
package hello.itemservice;  
  
import hello.itemservice.config.*;  
import hello.itemservice.repository.ItemRepository;  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Import;  
import org.springframework.context.annotation.Profile;  
  
  
//@Import(MemoryConfig.class)  
@Import(JdbcTemplateV1Config.class)  
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")  
public class ItemServiceApplication {
  // ...
}
```

**데이터베이스 접근 설정**
```
spring.profiles.active=local  
spring.datasource.url=jdbc:h2:tcp://localhost/~/test  
spring.datasource.username=sa  
spring.datasource.password=test
```
- 이렇게 설정만 하면 스프링 부트가 해당 설정을 사용해서 커넥션 풀과 `DataSource`, 트랜잭션 매니저를 스프링 빈으로 자동 등록한다.

#### 실행
- 실제 DB에 연결해야 하므로 H2 데이터베이스 서버를 먼저 실행하자.
	- 앞서 만든 item 테이블이 잘 생성되어 있는지 다시 확인하자.
- `ItemServiceApplication.main()`을 실행해서 애플리케이션 서버를 실행하자.
- 웹 브라우저로 다음에 접속하자
	- http://localhost:8080 
- 실행해보면 잘 동작하는 것을 확인할 수 있다.
	- 그리고 DB에 실제 데이터가 저장되는 것도 확인할 수 있다.  
- 참고로 메모리와 다르게 서버가 내려가도 데이터베이스는 유지되기 때문에 서버를 다시 시작할 때 마다 `TestDataInit`이 실행되기 때문에 `itemA`, `itemB`도 데이터베이스에 계속 추가된다.

#### 로그 추가
JdbcTemplate이 실행하는 SQL 로그를 확인하려면 `application.properties`에 다음을 추가하면 된다.
```
#jdbcTemplate sql log
logging.level.org.springframework.jdbc=debug
```
`main` , `test` 설정이 분리되어 있기 때문에 둘다 확인하려면 두 곳에 모두 추가해야 한다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__