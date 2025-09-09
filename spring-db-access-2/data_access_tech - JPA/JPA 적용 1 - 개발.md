JPA에서 가장 중요한 부분은 객체와 테이블을 매핑하는 것이다. JPA가 제공하는 애노테이션을 사용해서 `Item` 객체와 테이블을 매핑해보자.

```java
package hello.itemservice.domain;  
  
import javax.persistence.Column;  
import javax.persistence.Entity;  
import javax.persistence.GeneratedValue;  
import javax.persistence.GenerationType;  
import javax.persistence.Id;  
import lombok.Data;  
  
@Data  
@Entity  
public class Item {  
  
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column(name = "item_name", length = 10)  
    private String itemName;  
    private Integer price;  
    private Integer quantity;  
  
    public Item() {  
    }  
  
    public Item(String itemName, Integer price, Integer quantity) {  
        this.itemName = itemName;  
        this.price = price;  
        this.quantity = quantity;  
    }  
}
```

- `@Entity`: JPA가 사용하는 객체라는 뜻이다.
	- 이 에노테이션이 있어야 JPA가 인식할 수 있다. 이렇게 `@Entity`가 붙은 객체를 JPA에서는 엔티티라 한다.
- `@Id`: 테이블의 PK와 해당 필드를 매핑한다.
- `@GeneratedValue(strategy = GenerationType.IDENTITY)`: PK 생성 값을 데이터베이스에서 생성하는 `IDENTITY` 방식을 사용한다.
	- 예) MySQL auto increment
- `@Column`: 객체의 필드를 테이블의 컬럼과 매핑한다.
	- `name = "item_name"`: 객체는 `itemName`이지만 테이블의 컬럼은 `item_name`이므로 이렇게 매핑했다.
	- `length = 10`: JPA의 매핑 정보로 DDL(`create table`)도 생성할 수 있는데, 그때 컬럼의 길이 값으로 활용된다.(`varchar 10`)
	- `@Column`을 생략할 경우 필드 이름을 테이블 컬럼 이름으로 사용한다.
	- 참고로 지금처럼 스프링 부투와 통합해서 사용하면 필드 이름을 테이블 컬럼 명으로 변경할 때 객체 필드의 카멜 케이스를 테이블 컬럼의 언더스코어로 자동으로 변환해준다.
		- `itemName` -> `item_name` 따라서 위 예제의 `@Column(name = "item_name")`은 생략해도 된다.

JPA는 `public` 또는 `protected`의 기본 생성자가 필수이다. 기본 생성자를 꼭 넣어주자.
```java
public Item() {}
```

이렇게 하면 기본 매핑은 모두 끝난다. 이제 JPA를 실제 사용하는 코드를 작성해보자. 우선 코드를 작성하고 실행하면서 하나씩 알아보자.

**JpaItemRepositoryV1**
```java
package hello.itemservice.repository.jpa;  
  
import hello.itemservice.domain.Item;  
import hello.itemservice.repository.ItemRepository;  
import hello.itemservice.repository.ItemSearchCond;  
import hello.itemservice.repository.ItemUpdateDto;  
import java.util.List;  
import java.util.Optional;  
import javax.persistence.EntityManager;  
import javax.persistence.TypedQuery;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.stereotype.Repository;  
import org.springframework.transaction.annotation.Transactional;  
import org.springframework.util.StringUtils;  
  
@Slf4j  
@Repository  
@Transactional  
public class JpaItemRepositoryV1 implements ItemRepository {  
  
  private final EntityManager em;  
  
  public JpaItemRepositoryV1(EntityManager em) {  
    this.em = em;  
  }  
  
  @Override  
  public Item save(Item item) {  
    em.persist(item);  
    return item;  
  }  
  
  @Override  
  public void update(Long itemId, ItemUpdateDto updateParam) {  
    Item findItem = em.find(Item.class, itemId);  
    findItem.setItemName(updateParam.getItemName());  
    findItem.setPrice(updateParam.getPrice());  
    findItem.setQuantity(updateParam.getQuantity());  
  }  
  
  @Override  
  public Optional<Item> findById(Long id) {  
    Item item = em.find(Item.class, id);  
    return Optional.ofNullable(item);  
  }  
  
  @Override  
  public List<Item> findAll(ItemSearchCond cond) {  
    String jpql = "select i from Item i";  
  
    Integer maxPrice = cond.getMaxPrice();  
    String itemName = cond.getItemName();  
  
    if (StringUtils.hasText(itemName) || maxPrice != null) {  
      jpql += " where";  
    }  
  
    boolean andFlag = false;  
    if (StringUtils.hasText(itemName)) {  
      jpql += " i.itemName like concat('%', :itemName, '%')";  
      andFlag = true;  
    }  
  
    if (maxPrice != null) {  
      if (andFlag) {  
        jpql += " and";  
      }  
      jpql += " i.price <= :maxPrice";  
    }  
  
    log.info("jpql = {}", jpql);  
  
    TypedQuery<Item> query = em.createQuery(jpql, Item.class);  
    if (StringUtils.hasText(itemName)) {  
      query.setParameter("itemName", itemName);  
    }  
    if (maxPrice != null) {  
      query.setParameter("maxPrice", maxPrice);  
    }  
    return query.getResultList();  
  }  
  
}
```
- `private final EntityManager em`
	- 생성자를 보면 스프링을 통해 엔티티 매니저(EntityManager)라는 것을 주입받는 걸 확인할 수 있다.
	- JPA의 모든 동작은 엔티티 매니저를 통해서 이루어진다.
	- 엔티티 매니저는 내부 소수에 데이터 소스를 가지고 있고, 데이터베이스에 접근할 수 있다.
- `@Transactional`
	- JPA의 모든 데이터 변경(등록, 수정, 삭제)은 트랜잭션 안에서 이루어져야 한다.
	- 조회는 트랜잭션이 없어도 가능하다.
	- 변경의 경우 일반적으로 서비스 계층에서 트랜잭션을 시작하기 때문에 문제가 없다.
	- 하지만 이번 예제에는 복잡한 비즈니스 로직이 없어서 서비스 계층에서 트랜잭션을 걸지 않았다.
	- JPA에서는 데이터 변경 시 트랜잭션이 필수다.
	- 따라서 리포지토리에 트랜잭션을 걸었다.
	- 다시 강조하자면, 일반적으로 비즈니스 로직을 시작하는 서비스 계층에 트랜잭션을 걸어주는 것이 맞다.
		- 복잡한 비즈니스 로직은 데이터의 변경이 많이 발생하고, 이 데이터 변경들이 모두 한 비즈니스 로직 안에서 동작한다는 것은 해당 변경들이 모두 연관되어 있다고 할 수 있다.
		- 그렇다면, 리포지토리 계층에 트랜잭션이 걸릴 경우 각 메서드가 끝나는 시점에 commit이 발생하여, 서비스 계층의 비즈니스 로직에서 예외가 터질 경우 일부 데이터는 수정되었으나, 일부 데이터는 수정되지 않는 문제가 발생할 수 있다.
	- JPA는 트랜잭션이 커밋되는 시점(메서드 종료 시점)에 커밋을 하여 DB에 영속화 한다.

> 참고
> JPA를 설정하려면 `EntityManagerFactory`, JPA 트랜잭션 매니저(`JpaTransactionManager`), 데이터 소스 등등 다양한 설정을 해야 한다. 스프링 부트는 이 과정을 모두 자동화 해준다.

**JpaConfig**
```java
package hello.itemservice.config;  
  
import hello.itemservice.repository.ItemRepository;  
import hello.itemservice.repository.jpa.JpaItemRepositoryV1;  
import hello.itemservice.service.ItemService;  
import hello.itemservice.service.ItemServiceV1;  
import javax.persistence.EntityManager;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
public class JpaConfig {  
    
  private final EntityManager em;  
  
  public JpaConfig(EntityManager em) {  
    this.em = em;  
  }  
    
  @Bean  
  public ItemService itemService() {  
    return new ItemServiceV1(itemRepository());  
  }  
    
  @Bean  
  public ItemRepository itemRepository() {  
    return new JpaItemRepositoryV1(em);  
  }  
}
```

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
//@Import(MyBatisConfig.class)  
@Import(JpaConfig.class)  
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")  
public class ItemServiceApplication {  

}
```

**테스트를 실행하자**
먼저 `ItemRepositoryTest`를 통해서 리포지토리가 정상 동작하는지 확인해보자. 테스트가 모두 성공해야 한다.

**애플리케이션을 실행하자**
`ItemServiceApplication`를 실행해서 애플리케이션이 정상 동작하는지 확인해보자.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__