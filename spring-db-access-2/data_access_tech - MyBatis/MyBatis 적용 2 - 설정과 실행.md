**MyBatisItemRepository**
```java
package hello.itemservice.repository.mybatis;  
  
import hello.itemservice.domain.Item;  
import hello.itemservice.repository.ItemRepository;  
import hello.itemservice.repository.ItemSearchCond;  
import hello.itemservice.repository.ItemUpdateDto;  
import java.util.List;  
import java.util.Optional;  
import lombok.RequiredArgsConstructor;  
import org.springframework.stereotype.Repository;  
  
@Repository  
@RequiredArgsConstructor  
public class MyBatisItemRepository implements ItemRepository {  
  
  private final ItemMapper itemMapper;  
  
  @Override  
  public Item save(Item item) {  
    return itemMapper.save(item);  
  }  
  
  @Override  
  public void update(Long itemId, ItemUpdateDto updateParam) {  
    itemMapper.update(itemId, updateParam);  
  }  
  
  @Override  
  public Optional<Item> findById(Long id) {  
    return itemMapper.findById(id);  
  }  
  
  @Override  
  public List<Item> findAll(ItemSearchCond cond) {  
    return itemMapper.findAll(cond);  
  }  
}
```
- `ItemRepository`를 구현해서 `MyBatisItemRepository`를 만들자.
- `MyBatisItemRepository`는 단순히 `ItemMapper`에 기능을 위임한다.

**MyBatisConfig**
```java
package hello.itemservice.config;  
  
import hello.itemservice.repository.mybatis.ItemMapper;  
import hello.itemservice.repository.mybatis.MyBatisItemRepository;  
import hello.itemservice.service.ItemService;  
import hello.itemservice.service.ItemServiceV1;  
import lombok.RequiredArgsConstructor;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
@RequiredArgsConstructor  
public class MyBatisConfig {  
  
  private final ItemMapper itemMapper;  
  
  @Bean  
  public ItemService itemService() {  
    return new ItemServiceV1(itemRepository());  
  }  
  
  @Bean  
  public MyBatisItemRepository itemRepository() {  
    return new MyBatisItemRepository(itemMapper);  
  }  
}
```
- `MyBatisConfig`는 `ItemMapper`를 주입받고, 필요한 의존관계를 만든다.


**ItemServiceApplication - 변경**
```java
package hello.itemservice;  
  
import hello.itemservice.config.*;  
import hello.itemservice.repository.ItemRepository;  
import javax.sql.DataSource;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Import;  
import org.springframework.context.annotation.Profile;  
import org.springframework.jdbc.datasource.DriverManagerDataSource;  
  
@Slf4j  
//@Import(MemoryConfig.class)  
//@Import(JdbcTemplateV1Config.class)  
//@Import(JdbcTemplateV2Config.class)  
//@Import(JdbcTemplateV3Config.class)  
@Import(MyBatisConfig.class)  
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")  
public class ItemServiceApplication {}
```
- `@Import(MyBatisConfig.class)`: 앞서 설정한 `MyBatisConfig.class` 를 사용하도록 설정했다.

**테스트를 실행하자**  
먼저 `ItemRepositoryTest`를 통해서 리포지토리가 정상 동작하는지 확인해보자. 테스트가 모두 성공해야 한다.

**애플리케이션을 실행하자**
`ItemServiceApplication`를 실행해서 애플리케이션이 정상 동작하는지 확인해보자.

**주의!** H2 데이터베이스 서버를 먼저 실행해야 한다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__