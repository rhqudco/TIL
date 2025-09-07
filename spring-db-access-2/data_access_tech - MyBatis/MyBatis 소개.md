MyBatis는 앞서 설명한 JdbcTemplate보다 더 많은 기능을 제공하는 SQL Mapper 이다.
기본적으로 JdbcTemplate이 제공하는 대부분의 기능을 제공한다.
JdbcTemplate과 비교해서 MyBatis의 가장 매력적인 점은 SQL을 XML에 편리하게 작성할 수 있고 또 동적 쿼리를 매우 편리하게 작성할 수 있다는 점이다.

먼저 SQL이 여러 줄에 걸쳐 있을 때 둘을 비교하면 아래와 같다

**JdbcTemplate - SQL 여러 줄**
```java
String sql = "update item " +
		"set item_name=:itemName, price=:price, quantity=:quantity " +
		"where id=:id"l
```

**MyBatis - SQL 여러 줄**
```xml
<update id="update">
     update item
     set item_name=#{itemName},
         price=#{price},
         quantity=#{quantity}
     where id = #{id}
</update>
```

MyBatis는 XML에 작성하기 때문에 라인이 길어져도 문자 더하기에 대한 불편함이 없다.
다음으로 상품을 검색하는 로직을 통해 동적 쿼리를 비교해보자.

**JdbcTemplate - 동적 쿼리**
```java
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
```

**MyBatis - 동적 쿼리**
```xml
<select id="findAll" resultType="Item">
     select id, item_name, price, quantity
     from item
     <where>
         <if test="itemName != null and itemName != ''">
             and item_name like concat('%',#{itemName},'%')
         </if>
         <if test="maxPrice != null">
             and price &lt;= #{maxPrice}
         </if>
     </where>
 </select>
```

JdbcTemplate은 자바 코드로 직접 동적 쿼리를 작성해야 한다. 반면에 MyBatis는 동적 쿼리를 매우 편리하게 작성할 수 있는 다양한 기능들을 제공해준다.

**설정의 장단점**  
JdbcTemplate은 스프링에 내장된 기능이고, 별도의 설정없이 사용할 수 있다는 장점이 있다. 반면에 MyBatis는 약간의 설정이 필요하다.

**정리**
프로젝트에서 동적 쿼리와 복잡한 쿼리가 많다면 MyBatis를 사용하고, 단순한 쿼리들이 많으면 JdbcTemplate을 선택해서 사용하면 된다. 물론 둘을 함께 사용해도 된다. 하지만 MyBatis를 선택했다면 그것으로 충분할 것이다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__