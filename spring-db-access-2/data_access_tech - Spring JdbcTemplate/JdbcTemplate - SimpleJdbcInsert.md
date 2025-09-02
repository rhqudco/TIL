JdbcTemplate은 INSERT SQL를 직접 작성하지 않아도 되도록 `SimpleJdbcInsert`라는 편리한 기능을 제공한다.

**JdbcTemplateItemRepositoryV3**
```java
package hello.itemservice.repository.jdbctemplate;  
  
import hello.itemservice.domain.Item;  
import hello.itemservice.repository.ItemRepository;  
import hello.itemservice.repository.ItemSearchCond;  
import hello.itemservice.repository.ItemUpdateDto;  
import java.util.List;  
import java.util.Map;  
import java.util.Optional;  
import javax.sql.DataSource;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.dao.EmptyResultDataAccessException;  
import org.springframework.jdbc.core.BeanPropertyRowMapper;  
import org.springframework.jdbc.core.RowMapper;  
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;  
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;  
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;  
import org.springframework.jdbc.core.namedparam.SqlParameterSource;  
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;  
import org.springframework.stereotype.Repository;  
import org.springframework.util.StringUtils;  
  
/*  
* SimpleJdbcInsert  
* */  
@Slf4j  
@Repository  
public class JdbcTemplateRepositoryV3 implements ItemRepository {  
  
  private final NamedParameterJdbcTemplate template;  
  private final SimpleJdbcInsert jdbcInsert;  
  
  public JdbcTemplateRepositoryV3(DataSource dataSource) {  
    this.template = new NamedParameterJdbcTemplate(dataSource);  
    this.jdbcInsert = new SimpleJdbcInsert(dataSource)  
        .withTableName("item")  
        .usingGeneratedKeyColumns("id");  
//        .usingColumns("item_name", "price", "quantity"); // -> 생략 가능  
  }  
  
  @Override  
  public Item save(Item item) {  
    SqlParameterSource param = new BeanPropertySqlParameterSource(item);  
    Number key = jdbcInsert.executeAndReturnKey(param);  
    item.setId(key.longValue());  
    return item;  
  }  
  
  @Override  
  public void update(Long itemId, ItemUpdateDto updateParam) {  
    String sql = "update item " +  
        "set item_name=:itemName, price=:price, quantity=:quantity " +  
        "where id=:id";  
  
    SqlParameterSource param = new MapSqlParameterSource()  
        .addValue("itemName", updateParam.getItemName())  
        .addValue("price", updateParam.getPrice())  
        .addValue("quantity", updateParam.getQuantity())  
        .addValue("id", itemId);  
    template.update(sql, param);  
  }  
  
  @Override  
  public Optional<Item> findById(Long id) {  
    String sql = "select id, item_name, price, quantity from item where id = :id";  
    try {  
      Map<String, Object> param = Map.of("id", id);  
      Item item = template.queryForObject(sql, param, itemRowMapper());  
      return Optional.of(item);  
    } catch (EmptyResultDataAccessException e) {  
      return Optional.empty();  
    }  
  }  
  
  @Override  
  public List<Item> findAll(ItemSearchCond cond) {  
    Integer maxPrice = cond.getMaxPrice();  
    String itemName = cond.getItemName();  
    SqlParameterSource param = new BeanPropertySqlParameterSource(cond);  
  
    String sql = "select id, item_name, price, quantity from item";  
  
    if (StringUtils.hasText(itemName) || maxPrice != null) {  
      sql += " where";  
    }  
  
    boolean andFlag = false;  
  
    if (StringUtils.hasText(itemName)) {  
      sql += " item_name like concat('%',:itemName,'%')";  
      andFlag = true;  
    }  
  
    if (maxPrice != null) {  
      if (andFlag) {  
        sql += " and";  
      }  
      sql += " price <= :maxPrice";  
    }  
  
    log.info("sql={}", sql);  
    return template.query(sql, param, itemRowMapper());  
  }  
  
  private RowMapper<Item> itemRowMapper() {  
    return BeanPropertyRowMapper.newInstance(Item.class); // camel case 변환 지원  
  }  
}
```

- `JdbcTemplateItemRepositoryV3`은 `ItemRepository` 인터페이스를 구현했다.
- `this.jdbcInsert = new SimpleJdbcInsert(dataSource)`
	- 생성자를 보면 의존관계 주입은 `dataSource`를 받고 내부에서 `SimpleJdbcInsert`을 생성해서 가지고 있다.
	- 스프링에서는 `JdbcTemplate`관련 기능을 사용할 때 관례상 이 방법을 많이 사용한다.
		- 물론 `SimpleJdbcInsert`을 스프링 빈으로 직접 등록하고 주입받아도 된다.

**SimpleJdbcInsert**
```java
this.jdbcInsert = new SimpleJdbcInsert(dataSource)  
	.withTableName("item")  
	.usingGeneratedKeyColumns("id");  
//    .usingColumns("item_name", "price", "quantity"); // -> 생략 가능
```

- `withTableName`: 데이터를 저장할 테이블 명을 지정한다.  
- `usingGeneratedKeyColumns`: `key`를 생성하는 PK 컬럼 명을 지정한다.  
- `usingColumns`: INSERT SQL에 사용할 컬럼을 지정한다. 특정 값만 저장하고 싶을 때 사용한다. 생략할 수 있다.

`SimpleJdbcInsert`는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다.
따라서 어떤 컬럼이 있는지 확인할 수 있으므로 `usingColumns`을 생략할 수 있다. 만약 특정 컬럼만 지정해서 저장하고 싶다면 `usingColumns`를 사용하면 된다.

애플리케이션을 실행해보면 `SimpleJdbcInsert`이 어떤 INSERT SQL을 만들어서 사용하는지 로그로 확인할 수 있다.

```
DEBUG 67438 --- [           main] o.s.jdbc.core.simple.SimpleJdbcInsert    : Compiled insert object: insert string is [INSERT INTO item (ITEM_NAME, PRICE, QUANTITY) VALUES(?, ?, ?)]
```

**save()**
`jdbcInsert.executeAndReturnKey(param)`을 사용해서 INSERT SQL을 실행하고, 생성된 키 값도 매우 편리하게 조회할 수 있다.
```java
public Item save(Item item) {  
  SqlParameterSource param = new BeanPropertySqlParameterSource(item);  
  Number key = jdbcInsert.executeAndReturnKey(param);  
  item.setId(key.longValue());  
  return item;  
}
```
나머지는 코드 부분은 기존과 같다.

**JdbcTemplateV3Config**
```java
package hello.itemservice.config;  
  
import hello.itemservice.repository.ItemRepository;  
import hello.itemservice.repository.jdbctemplate.JdbcTemplateRepositoryV2;  
import hello.itemservice.repository.jdbctemplate.JdbcTemplateRepositoryV3;  
import hello.itemservice.service.ItemService;  
import hello.itemservice.service.ItemServiceV1;  
import javax.sql.DataSource;  
import lombok.RequiredArgsConstructor;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
@RequiredArgsConstructor  
public class JdbcTemplateV3Config {  
  
  private final DataSource dataSource;  
  
  @Bean  
  public ItemService itemService() {  
    return new ItemServiceV1(itemRepository());  
  }  
  
  @Bean  
  public ItemRepository itemRepository() {  
    return new JdbcTemplateRepositoryV3(dataSource);  
  }  
  
}
```

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
//@Import(JdbcTemplateV2Config.class)  
@Import(JdbcTemplateV3Config.class)  
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
- `JdbcTemplateV3Config.class`를 설정으로 사용하도록 변경되었다.
	- `@Import(JdbcTemplateV2Config.class)` -> `@Import(JdbcTemplateV3Config.class)`

#### 실행
- http://localhost:8080
	-  기능이 잘 동작하는지 확인해보자.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__