## 애노테이션으로 SQL 작성
다음과 같이 XML 대신에 애노테이션에 SQL을 작성할 수 있다.

```java
@Select("select id, item_name, price, quantity from item where id=#{id}")
Optional<Item> findById(Long id);
```

- `@Insert`, `@Update`, `@Delete`, `@Select` 기능이 제공된다.
- 이 경우 XML에는 `<select id="findById"> ~ </select>`는 제거해야 한다.
- 동적 SQL이 해결되지 않으므로 간단한 경우에만 사용한다.

애노테이션으로 SQL 작성에 대한 더 자세한 내용은 다음을 참고하자.
https://mybatis.org/mybatis-3/ko/java-api.html

## 문자열 대체(String Substitution)
`#{}` 문법은 `?`를 넣고 파라미터를 바인딩하는 `PreparedStatement`를 사용한다.
때로는 파라미터 바인딩이 아니라 문자 그대로를 처리하고 싶은 경우도 있다. 이때는 `${}`를 사용하면 된다.

다음 예제를 보자
`ORDER BY ${columnName}`

```java
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```

**주의**  
`${}`를 사용하면 SQL 인젝션 공격을 당할 수 있다. 따라서 가급적 사용하면 안된다. 사용하더라도 매우 주의깊게 사용해야 한다.

## 재사용 가능한 SQL 조각
`<sql>`을 사용하면 SQL 코드를 재사용 할 수 있다.
```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```

```xml
<select id="selectUsers" resultType="map">
   select
     <include refid="userColumns"><property name="alias" value="t1"/></include>,
     <include refid="userColumns"><property name="alias" value="t2"/></include>
   from some_table t1
     cross join some_table t2
 </select>
```

- `<include>`를 통해서 `<sql>` 조각을 찾아서 사용할 수 있다.

```xml
<sql id="sometable">
   ${prefix}Table
</sql>
 <sql id="someinclude">
   from
     <include refid="${include_target}"/>
 </sql>
 <select id="select" resultType="map">
   select
     field1, field2, field3
   <include refid="someinclude">
     <property name="prefix" value="Some"/>
     <property name="include_target" value="sometable"/>
   </include>
</select>
```
- 프로퍼티 값을 전달할 수 있고, 해당 값은 내부에서 사용할 수 있다.

## Result Maps
결과를 매핑할 때 테이블은 `user_id`이지만 객체는 `id`이다.
이 경우 컬럼명과 객체의 프로퍼티 명이 다르다. 그러면 다음과 같이 별칭(`as`)을 사용하면 된다.

```xml
<select id="selectUsers" resultType="User">
  select
    user_id as "id",
    user_name as "userName",
    hashed_password as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```
별칭을 사용하지 않고도 문제를 해결할 수 있는데, 다음과 같이 `resultMap`을 선언해서 사용하면 된다.

```xml
<resultMap id="userResultMap" type="User">
   <id property="id" column="user_id" />
   <result property="username" column="user_name"/>
   <result property="password" column="hashed_password"/>
 </resultMap>

 <select id="selectUsers" resultMap="userResultMap">
   select user_id, user_name, hashed_password
   from some_table
   where id = #{id}
</select>
```

**복잡한 결과매핑**
MyBatis도 매우 복잡한 결과에 객체 연관관계를 고려해서 데이터를 조회하는 것이 가능하다.  
이때는 `<association>`, `<collection>` 등을 사용한다.  
이 부분은 성능과 실효성에서 측면에서 많은 고민이 필요하다.  
JPA는 객체와 관계형 데이터베이스를 ORM 개념으로 매핑하기 때문에 이런 부분이 자연스럽지만, MyBatis에서는 들어가는 공수도 많고, 성능을 최적화하기도 어렵다.
따라서 해당기능을 사용할 때는 신중하게 사용해야 한다.  
해당 기능에 대한 자세한 내용은 공식 매뉴얼을 참고하자.

> **참고**
> 결과 매핑에 대한 자세한 내용은 다음을 참고하자.
> https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#Result_Maps

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__