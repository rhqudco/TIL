이제 이름 지정 파라미터를 사용하도록 구성하고 실행해보자.

**JdbcTemplateV2Config**
```java
package hello.itemservice.config;  
  
import hello.itemservice.repository.ItemRepository;  
import hello.itemservice.repository.jdbctemplate.JdbcTemplateRepositoryV2;  
import hello.itemservice.service.ItemService;  
import hello.itemservice.service.ItemServiceV1;  
import javax.sql.DataSource;  
import lombok.RequiredArgsConstructor;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
@RequiredArgsConstructor  
public class JdbcTemplateV2Config {  
  
  private final DataSource dataSource;  
  
  @Bean  
  public ItemService itemService() {  
    return new ItemServiceV1(itemRepository());  
  }  
  
  @Bean  
  public ItemRepository itemRepository() {  
    return new JdbcTemplateRepositoryV2(dataSource);  
  }  
  
}
```
- 앞서 개발한 `JdbcTemplateItemRepositoryV2`를 사용하도록 스프링 빈에 등록한다.

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
//@Import(JdbcTemplateV1Config.class)  
@Import(JdbcTemplateV2Config.class)  
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")  
public class ItemServiceApplication {  
  
  public static void main(String[] args) {  
   SpringApplication.run(ItemServiceApplication.class, args);  
  }  
  
  @Bean  
  @Profile("local")  
  public TestDataInit testDataInit(ItemRepository itemRepository) {  
   return new TestDataInit(itemRepository);  
  }  
  
}
```

- `JdbcTemplateV2Config.class`를 설정으로 사용하도록 변경되었다.
	- `@Import(JdbcTemplateV1Config.class)` -> `@Import(JdbcTemplateV2Config.class)`

#### 실행
- http://localhost:8080
	-  기능이 잘 동작하는지 확인해보자.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__