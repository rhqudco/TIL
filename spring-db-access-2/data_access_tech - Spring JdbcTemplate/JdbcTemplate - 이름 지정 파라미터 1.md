# 순서대로 바인딩
JdbcTemplate을 기본으로 사용하면 파라미터를 순서대로 바인딩 한다.
예를 들어 아래 코드를 보자
```java
String sql = "update item set item_name=?, price=?, quantity=? where id=?";
template.update(sql, itemName, price, quantity, itemId);
```
여기서는 `itemName`, `price`, `quantity`가 SQL에 있는 `?`에 순서대로 바인딩 된다.
따라서 순서만 잘 지키면 문제가 될 것은 없다. 그런데 문제는 변경시점에 발생한다.

누군가 다음과 같이 SQL 코드의 순서를 변경했다고 가정해보자. (`price`와 `quantity`의 순서를 변경했다.)
```java
String sql = "update item set item_name=?, quantity=?, price=? where id=?";
template.update(sql, itemName, price, quantity, itemId);
```

이렇게 되면 다음과 같은 순서로 데이터가 바인딩 된다. `item_name=itemName, quantity=price, price=quantity`

결과적으로 `price`와 `quantity`가 바뀌는 매우 심각한 문제가 발생한다.
이럴일이 없을 것 같지만, 실무에서는 파라미터가 10~20개가 넘어가는 일도 아주 많다. 그래서 미래에 필드를 추가하거나, 수정하면서 이런 문제가 충분히 발생 할 수 있다.
버그 중에서 가장 고치기 힘든 버그는 데이터베이스에 데이터가 잘못 들어가는 버그다.
이것은 코드만 고치는 수준이 아니라 데이터베이스의 데이터를 복구해야 하기 때문에 버그를 해결하는데 들어가는 리소스가 어마어마하다.

실제로 수많은 개발자들이 이 문제로 장애를 내고 퇴근하지 못하는 일이 발생한다.

**개발을 할 때는 코드를 몇줄 줄이는 편리함도 중요하지만, 모호함을 제거해서 코드를 명확하게 만드는 것이 유지보수 관점에서 매우 중요하다.**

이처럼 파라미터를 순서대로 바인딩 하는 것은 편리하기는 하지만, 순서가 맞지 않아서 버그가 발생할 수도 있으므로 주의해서 사용해야 한다.

# 이름 지정 바인딩
JdbcTemplate은 이런 문제를 보완하기 위해 `NamedParameterJdbcTemplate`라는 이름을 지정해서 파라미터를 바인딩 하는 기능을 제공한다.

코드로 알아보자.

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
import org.springframework.jdbc.support.GeneratedKeyHolder;  
import org.springframework.jdbc.support.KeyHolder;  
import org.springframework.stereotype.Repository;  
import org.springframework.util.StringUtils;  
  
/*  
* NamedParameterJdbcTemplate  
* SqlParameterSource  
*  - BeanPropertySqlParameterSource  
*  - MapSqlParameterSource  
* Map  
*  
* BeanPropertyRowMapper  
* */  
@Slf4j  
@Repository  
public class JdbcTemplateRepositoryV2 implements ItemRepository {  
  
  private final NamedParameterJdbcTemplate template;  
  
  public JdbcTemplateRepositoryV2(DataSource dataSource) {  
    this.template = new NamedParameterJdbcTemplate(dataSource);  
  }  
  
  @Override  
  public Item save(Item item) {  
    String sql = "insert into item (item_name, price, quantity) values (:itemName, :price, :quantity)";  
    SqlParameterSource param = new BeanPropertySqlParameterSource(item); // 인자로 받는 클래스의 필드 이름으로 파라미터 매핑  
    KeyHolder keyHolder = new GeneratedKeyHolder();  
    template.update(sql, param, keyHolder);  
    Long key = keyHolder.getKey().longValue();  
    item.setId(key);  
    return item;  
  
  }  
  
  @Override  
  public void update(Long itemId, ItemUpdateDto updateParam) {  
    String sql = "update item " +  
        "set item_name=:itemName, price=:price, quantity=:quantity " +  
        "where id=:id";  
  
    SqlParameterSource param = new MapSqlParameterSource() // Map SQL을 통해 세팅하여 파라미터 매핑  
        .addValue("itemName", updateParam.getItemName())  
        .addValue("price", updateParam.getPrice())  
        .addValue("quantity", updateParam.getQuantity())  
        .addValue("id", itemId); //이 부분이 별도로 필요하다.  
    template.update(sql, param);  
  }  
  
  @Override  
  public Optional<Item> findById(Long id) {  
    String sql = "select id, item_name, price, quantity from item where id = :id";  
    try {  
      Map<String, Object> param = Map.of("id", id);  
      Item item = template.queryForObject(sql, param, itemRowMapper()); // 파라미터를 Map을 통해 설정할 수 있음  
      return Optional.of(item);  
    } catch (EmptyResultDataAccessException e) {  
      return Optional.empty();  
    }  
  }  
  
  @Override  
  public List<Item> findAll(ItemSearchCond cond) {  
    Integer maxPrice = cond.getMaxPrice();  
    String itemName = cond.getItemName();  
    SqlParameterSource param = new BeanPropertySqlParameterSource(cond); // 인자로 받는 클래스의 필드 이름으로 파라미터 매핑  
  
    String sql = "select id, item_name, price, quantity from item"; //동적 쿼리  
  
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

**기본**
- `JdbcTemplateItemRepositoryV2`는 `ItemRepository` 인터페이스를 구현했다.
- `this.template = new NamedParameterJdbcTemplate(dataSource)`
	- `NamedParameterJdbcTemplate`도 내부에 `dataSource`가 필요하다.
	- `JdbcTemplateItemRepositoryV2` 생성자를 보면 의존관계 주입은 `dataSource`를 받고 내부에서 `NamedParameterJdbcTemplate`을 생성해서 가지고 있다.
	- 스프링에서는 `JdbcTemplate` 관련 기능을 사용할 때 관례상 이 방법을 많이 사용한다.
	- 물론 `NamedParameterJdbcTemplate`을 스프링 빈으로 직접 등록하고 주입받아도 된다.

**save()**
SQL에서 다음과 같이 `?` 대신에 `:파라미터이름`을 받는 것을 확인할 수 있다.
```java
insert into item (item_name, price, quantity) values (:itemName, :price, :quantity)"
```
추가로 `NamedParameterJdbcTemplate`은 데이터베이스가 생성해주는 키를 매우 쉽게 조회하는 기능도 제공해준다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__