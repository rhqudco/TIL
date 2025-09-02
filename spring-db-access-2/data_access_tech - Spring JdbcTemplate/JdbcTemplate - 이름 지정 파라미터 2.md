# 이름 지정 파라미터
파라미터를 전달하려면 `Map` 처럼 `Key`, `Value` 데이터 구조를 만들어서 전달해야 한다.
여기서 `Key`는 `:파라미터 이름`으로 지정한 파라미터의 이름이고, `Value`는 해당 파라미터의 값이 된다.

다음 코드를 보면 이렇게 만든 파라미터(`param`)를 전달하는 것을 확인할 수 있다.
`template.update(sql, param, keyHolder);`

이름 지정 바인딩에서 주로 사용하는 파라미터의 종류는 크게 3가지가 있다.
- `Map`
- `SqlParameterSource`
	- `MapSqlParameterSource`
	- `BeanPropertySqlParameterSource`

#### Map
단순히 `Map`을 사용한다.
`findById()` 코드에서 확인할 수 있다.

```java
Map<String, Object> param = Map.of("id", id);
Item item = template.queryForObject(sql, param, itemRowMapper());
```

#### MapSqlParameterSource
`Map`과 유사한데, SQL 타입을 지정할 수 있는 등 SQL에 좀 더 특화된 기능을 제공한다.
`SqlParameterSource` 인터페이스의 구현체이다.  
`MapSqlParameterSource` 는 메서드 체인을 통해 편리한 사용법도 제공한다.

`update()` 코드에서 확인할 수 있다.
```java
SqlParameterSource param = new MapSqlParameterSource()
    .addValue("itemName", updateParam.getItemName())  
    .addValue("price", updateParam.getPrice())  
    .addValue("quantity", updateParam.getQuantity())  
    .addValue("id", itemId);
template.update(sql, param);
```

#### BeanPropertySqlParameterSource
자바빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체를 생성한다.
예시: `getXxx() -> xxx, getItemName() -> itemName`

예를 들어서 `getItemName()`, `getPrice()`가 있으면 다음과 같은 데이터를 자동으로 만들어낸다.
- `key=itemName, value=상품명 값`
- `key=price, value=가격 값`
`SqlParameterSource` 인터페이스의 구현체이다.

`save()` , `findAll()` 코드에서 확인할 수 있다.

```java
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
KeyHolder keyHolder = new GeneratedKeyHolder();  
template.update(sql, param, keyHolder);
```

- 여기서 보면 `BeanPropertySqlParameterSource`가 많은 것을 자동화 해주기 때문에 가장 좋아 보이지만, `BeanPropertySqlParameterSource`를 항상 사용할 수 있는 것은 아니다.
- 예를 들어서 `update()`에서는 SQL에 `:id`를 바인딩 해야 하는데, `update()`에서 사용하는 `ItemUpdateDto`에는 `itemId`가 없다.
	- 따라서 `BeanPropertySqlParameterSource`를 사용할 수 없고, 대신에 `MapSqlParameterSource`를 사용했다.

#### BeanPropertyRowMapper
이번 코드에서 `V1`과 비교해서 변화된 부분이 하나 더 있다. 바로 `BeanPropertyRowMapper`를 사용한 것이다.

**JdbcTemplateItemRepositoryV1 - itemRowMapper()**
```java
private RowMapper<Item> itemRowMapper() {
  return (rs, rowNum) -> {
	  Item item = new Item();
	  item.setId(rs.getLong("id"));
	  item.setItemName(rs.getString("item_name"));
	  item.setPrice(rs.getInt("price"));
	  item.setQuantity(rs.getInt("quantity"));
	  return item;
  };
}
```

**JdbcTemplateItemRepositoryV2 - itemRowMapper()**
```java
private RowMapper<Item> itemRowMapper() {  
  return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원
}
```

`BeanPropertyRowMapper`는 `ResultSet`의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환한다.
예를 들어서 데이터베이스에서 조회한 결과가 `select id, price`라고 하면 다음과 같은 코드를 작성해준다.
(실제로는 리플렉션 같은 기능을 사용한다.)

```java
Item item = new Item();
item.setId(rs.getLong("id"));
item.setPrice(rs.getInt("price")); 
```

데이터베이스에서 조회한 결과 이름을 기반으로 `setId()`, `setPrice()`처럼 자바빈 프로퍼티 규약에 맞춘 메서드를 호출하는 것이다.

#### 별칭
그런데 `select item_name`의 경우 `setItem_name()`이라는 메서드가 없기 때문에 골치가 아프다.
이런 경우 개발자가 조회 SQL을 다음과 같이 고치면 된다.
`select item_name as itemName`

별칭 `as`를 사용해서 SQL 조회 결과의 이름을 변경하는 것이다.
실제로 이 방법은 자주 사용된다. 특히 데이터베이스 컬럼 이름과 객체 이름이 완전히 다를 때 문제를 해결할 수 있다.
예를 들어서 데이터베이스에는 `member_name`이라고 되어 있는데 객체에 `username`이라고 되어 있다면 다음과 같이 해결할 수 있다.
`select member_name as username`  

이렇게 데이터베이스 컬럼 이름과 객체의 이름이 다를 때 별칭(`as`)을 사용해서 문제를 많이 해결한다.
`JdbcTemplate`은 물론이고, `MyBatis`같은 기술에서도 자주 사용된다.

#### 관례의 불일치
자바 객체는 카멜(`camelCase`) 표기법을 사용한다.
`itemName`처럼 중간에 낙타 봉이 올라와 있는 표기법이다.

반면에 관계형 데이터베이스에서는 주로 언더스코어를 사용하는 `snake_case`표기법을 사용한다. `item_name`처럼 중간에 언더스코어를 사용하는 표기법이다.

이 부분을 관례로 많이 사용하다 보니 `BeanPropertyRowMapper`는 언더스코어 표기법을 카멜로 자동 변환해준다.
따라서 `select item_name`으로 조회해도 `setItemName()`에 문제 없이 값이 들어간다.

정리하면 `snake_case`는 자동으로 해결되니 그냥 두면 되고, 컬럼 이름과 객체 이름이 완전히 다른 경우에는 조회 SQL에서 별칭(`as`)을 사용하면 된다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__