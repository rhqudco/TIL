이제부터 본격적으로 MyBatis를 사용해서 데이터베이스에 데이터를 저장해보자.
XML에 작성한다는 점을 제외하고는 JDBC 반복을 줄여준다는 점에서 기존 JdbcTemplate과 거의 유사하다.

**ItemMapper**
```java
package hello.itemservice.repository.mybatis;  
  
import hello.itemservice.domain.Item;  
import hello.itemservice.repository.ItemSearchCond;  
import hello.itemservice.repository.ItemUpdateDto;  
import java.util.List;  
import java.util.Optional;  
import org.apache.ibatis.annotations.Mapper;  
import org.apache.ibatis.annotations.Param;  
  
@Mapper  
public interface ItemMapper {  
  void save(Item item);  
  void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto updateParam);  
  Optional<Item> findById(Long id);  
  List<Item> findAll(ItemSearchCond itemSearchCond);  
}
```

- 마이바티스 매핑 XML을 호출해주는 매퍼 인터페이스이다.
- 이 인터페이스에는 `@Mapper` 애노테이션을 붙여주어야 한다. 그래야 MyBatis에서 인식할 수 있다.
- 이 인터페이스의 메서드를 호출하면 다음에 보이는 `xml`의 해당 SQL을 실행하고 결과를 돌려준다.
- `ItemMapper` 인터페이스의 구현체에 대한 부분은 뒤에 별도로 설명한다.

이제 같은 위치에 실행할 SQL이 있는 XML 매핑 파일을 만들어주면 된다.
참고로 자바 코드가 아니기 때문에 `src/main/resources` 하위에 만들되, 패키지 위치는 맞추어 주어야 한다.

`src/main/resources/hello/itemservice/repository/mybatis/ItemMapper.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
  
<mapper namespace="hello.itemservice.repository.mybatis.ItemMapper">  
  <insert id="save" useGeneratedKeys="true" keyProperty="id">  
    insert into item (item_name, price, quantity)  
    values (#{itemName}, #{price}, #{quantity})
  </insert>  
  
  <update id="update">  
    update item  
    set item_name=#{updateParam.itemName},
	    price=#{updateParam.price},
		quantity=#{updateParam.quantity}
		where id = #{id}
  </update>  
  
  <select id="findById" resultType="Item">  
    select id, item_name, price, quantity
    from item
    where id = #{id}  
  </select>  
  
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
</mapper>
```
- `namespace` : 앞서 만든 매퍼 인터페이스를 지정하면 된다.
- **주의!** 경로와 파일 이름에 주의하자.

**참고 - XML 파일 경로 수정하기**
XML 파일을 원하는 위치에 두고 싶으면 `application.properties` 에 다음과 같이 설정하면 된다.
`mybatis.mapper-locations=classpath:mapper/**/*.xml`

이렇게 하면 `resources/mapper`를 포함한 그 하위 폴더에 있는 XML을 XML 매핑 파일로 인식한다.
이 경우 파일 이름은 자유롭게 설정해도 된다.  
참고로 테스트의 `application.properties` 파일도 함께 수정해야 테스트를 실행할 때 인식할 수 있다.

**insert - save**
```java
void save(Item item)

<insert id="save" useGeneratedKeys="true" keyProperty="id">
     insert into item (item_name, price, quantity)
     values (#{itemName}, #{price}, #{quantity})
</insert>
```
- Insert SQL은 `<insert>`를 사용하면 된다.  
- `id`에는 매퍼 인터페이스에 설정한 메서드 이름을 지정하면 된다. 여기서는 메서드 이름이 `save()`이므로 `save`로 지정하면 된다.
- 파라미터는 `#{}` 문법을 사용하면 된다. 그리고 매퍼에서 넘긴 객체의 프로퍼티 이름을 적어주면 된다.
- `#{}` 문법을 사용하면 `PreparedStatement`를 사용한다. JDBC의 `?`를 치환한다 생각하면 된다.
- `useGeneratedKeys`는 데이터베이스가 키를 생성해 주는 `IDENTITY` 전략일 때 사용한다.
- `keyProperty`는 생성되는 키의 속성 이름을 지정한다. Insert가 끝나면 `item` 객체의 `id` 속성에 생성된 값이 입력된다.

**update - update**
```java
import org.apache.ibatis.annotations.Param;
void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto updateParam);

<update id="update">
     update item
     set item_name=#{updateParam.itemName},
         price=#{updateParam.price},
         quantity=#{updateParam.quantity}
     where id = #{id}
</update>
```
- Update SQL은 `<update>`를 사용하면 된다.  
- 여기서는 파라미터가 `Long id`, `ItemUpdateDto updateParam`으로 2개이다.
- 파라미터가 1개만 있으면 `@Param`을 지정하지 않아도 되지만, 파라미터가 2개 이상이면 `@Param`으로 이름을 지정해서 파라미터를 구분해야 한다.

**select - findById**
```java
Optional<Item> findById(Long id);

<select id="findById" resultType="Item">  
  select id, item_name, price, quantity  
  from item  where id = #{id}
</select>
```
- Select SQL은 `<select>`를 사용하면 된다.
- `resultType`은 반환 타입을 명시하면 된다. 여기서는 결과를 `Item` 객체에 매핑한다.
	- 앞서 `application.properties`에 `mybatis.type-aliases-package=hello.itemservice.domain` 속성을 지정한 덕분에 모든 패키지 명을 다 적지는 않아도 된다.
	- 그렇지 않으면 모든 패키지 명을 다 적어야 한다.
	- JdbcTemplate의 `BeanPropertyRowMapper` 처럼 SELECT SQL의 결과를 편리하게 객체로 바로 변환해준다.
	- `mybatis.configuration.map-underscore-to-camel-case=true` 속성을 지정한 덕분에 언더스코어를 카멜 표기법으로 자동으로 처리해준다. (`item_name` -> `itemName`)
- 자바 코드에서 반환 객체가 하나이면 `Item`, `Optional<Item>`과 같이 사용하면 되고, 반환 객체가 하나 이상이면 컬렉션을 사용하면 된다. 주로 `List`를 사용한다. 다음을 참고하자.

**select - findAll**
```java
List<Item> findAll(ItemSearchCond itemSearch);

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
- Mybatis는 `<where>`, `<if>` 같은 동적 쿼리 문법을 통해 편리한 동적 쿼리를 지원한다.
- `<if>`는 해당 조건이 만족하면 구문을 추가한다.  
- `<where>`은 적절하게 `where` 문장을 만들어준다.
	- 예제에서 `<if>`가 모두 실패하게 되면 SQL `where`를 만들지 않는다.  
	- 예제에서 `<if>`가 하나라도 성공하면 처음 나타나는 `and`를 `where`로 변환해준다.

**XML 특수문자**
그런데 가격을 비교하는 조건을 보자
`and price &lt;= #{maxPrice}`
여기에 보면 `<=`를 사용하지 않고 `&lt;=`를 사용한 것을 확인할 수 있다. 그 이유는 XML 에서는 데이터 영역에 `<`, `>` 같은 특수문자를 사용할 수 없기 때문이다.
이유는 간단한데, XML 에서 TAG가 시작하거나 종료할 때 `<`, `>`와같은 특수문자를 사용하기 때문이다.
```
< : &lt;
> : &gt;
& : &amp;
```

다른 해결 방안으로는 XML에서 지원하는 `CDATA` 구문 문법을 사용하는 것이다.
이 구문 안에서는 특수문자를 사용할 수 있다. 대신 이 구문 안에서는 XML TAG가 단순 문자로 인식되기 때문에 `<if>`, `<where>` 등이 적용되지 않는다.

**XML CDATA 사용**
```xml
<select id="findAll" resultType="Item">  
  select id, item_name, price, quantity  
  from item
  <where>  
    <if test="itemName != null and itemName != ''">  
      and item_name like concat('%',#{itemName},'%')  
    </if>  
    <if test="maxPrice != null">  
      <![CDATA[
        and price <= #{maxPrice}
        ]]>
    </if>  
  </where>  
</select>
```
특수문자와 CDATA 각각 상황에 따른 장단점이 있으므로 원하는 방법을 그때그때 선택하면 된다.


__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__