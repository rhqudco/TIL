MyBatis에서 자주 사용하는 주요 기능을 공식 메뉴얼이 제공하는 예제를 통해 간단히 정리해보자.
- MyBatis 공식 메뉴얼: https://mybatis.org/mybatis-3/ko/index.html
- MyBatis 스프링 공식 메뉴얼: https://mybatis.org/spring/ko/index.html

## 동적 SQL
마이바티스가 제공하는 최고의 기능이자 마이바티스를 사용하는 이유는 바로 동적 SQL 기능 때문이다.
동적 쿼리를 위해 제공되는 기능은 다음과 같다.

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

공식 매뉴얼에서 제공하는 예제를 통해 동적 SQL을 알아보자.

**if**
```xml
<select id="findActiveBlogWithTitleLike"
      resultType="Blog">
   SELECT * FROM BLOG
   WHERE state = ‘ACTIVE’
   <if test="title != null">
     AND title like #{title}
   </if>
</select>
```

- 해당 조건에 따라 값을 추가할지 말지 판단한다.  
- 내부의 문법은 OGNL을 사용한다. 자세한 내용은 OGNL을 검색해보자.

**choose, when, otherwise**
```xml
<select id="findActiveBlogLike"
      resultType="Blog">
   SELECT * FROM BLOG WHERE state = ‘ACTIVE’
   <choose>
     <when test="title != null">
       AND title like #{title}
     </when>
     <when test="author != null and author.name != null">
       AND author_name like #{author.name}
     </when>
     <otherwise>
       AND featured = 1
     </otherwise>
   </choose>
</select>
```
- 자바의 switch 구문과 유사한 구문도 사용할 수 있다.

**trim, where, set**
```xml
<select id="findActiveBlogLike"
      resultType="Blog">
   SELECT * FROM BLOG
   WHERE
   <if test="state != null">
     state = #{state}
   </if>
   <if test="title != null">
     AND title like #{title}
   </if>
   <if test="author != null and author.name != null">
     AND author_name like #{author.name}
   </if>
</select>
```

이 예제의 문제점은 문장을 모두 만족하지 않을 때 발생한다.

```SQL
SELECT * FROM BLOG
WHERE
```

`title`만 만족할 때도 문제가 발생한다.
```SQL
SELECT * FROM BLOG
WHERE
AND title like ‘someTitle’
```
결국 `WHERE` 문을 언제 넣어야 할지 상황에 따라서 동적으로 달라지는 문제가 있다.
`<where>`를 사용하면 이런 문제를 해결할 수 있다.

where 사용
```xml
<select id="findActiveBlogLike"
      resultType="Blog">
   SELECT * FROM BLOG
   <where>
     <if test="state != null">
          state = #{state}
     </if>
     <if test="title != null">
         AND title like #{title}
     </if>
     <if test="author != null and author.name != null">
         AND author_name like #{author.name}
     </if>
   </where>
</select>
```
`<where>` 는 문장이 없으면 `where`를 추가하지 않는다. 문장이 있으면 `where`를 추가한다. 만약 `and`가 먼저 시작된다면 `and`를 지운다.

참고로 다음과 같이 `trim`이라는 기능으로 사용해도 된다. 이렇게 정의하면 `<where>`와 같은 기능을 수행한다.
```xml
<trim prefix="WHERE" prefixOverrides="AND | OR ">
   ...
</trim>
```

**foreach**
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
   SELECT *
   FROM POST P
   <where>
     <foreach item="item" index="index" collection="list"
         open="ID in (" separator="," close=")" nullable="true">
           #{item}
     </foreach>
  </where>
</select>
```
- 컬렉션을 반복 처리할 때 사용한다. `where in (1,2,3,4,5,6)`와 같은 문장을 쉽게 완성할 수 있다.
- 파라미터로 `List`를 전달하면 된다.

> **참고**  
> 동적 쿼리에 대한 자세한 내용은 다음을 참고하자.
> https://mybatis.org/mybatis-3/ko/dynamic-sql.html


__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__