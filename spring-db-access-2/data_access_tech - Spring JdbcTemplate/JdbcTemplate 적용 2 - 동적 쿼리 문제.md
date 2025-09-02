결과를 검색하는 `findAll()`에서 어려운 부분은 사용자가 검색하는 값에 따라서 실행하는 SQL이 동적으로 달라져야 한다는 점이다.
예를 들어 다음과 같다.

검색 조건이 없음
```sql
select id, item_name, price quantity from item
```

상품명(`itemName`)으로 검색
```sql
select id, item_name, price, quantity from item where item_name like concat('%',?,'%')
```

최대 가격(`maxPrice`)으로 검색 
```sql
select id, item_name, price, quantity from item where price <= ?
```

상품명(`itemName`), 최대 가격(`maxPrice`) 둘다 검색
```sql
select id, item_name, price, quantity from item
where item_name like concat('%',?,'%')
and price <= ?
```

결과적으로 4가지 상황에 따른 SQL을 동적으로 생성해야 한다. 
동적 쿼리가 언뜻 보면 쉬워 보이지만, 막상 개발해보면 생각보다 다양한 상황을 고민해야 한다. 예를 들어서 어떤 경우에는 `where`를 앞에 넣고 어떤 경우에는 `and`를 넣어야 하는지 등을 모두 계산해야 한다.  
그리고 각 상황에 맞추어 파라미터도 생성해야 한다.  
물론 실무에서는 이보다 훨씬 더 복잡한 동적 쿼리들이 사용된다.

참고로 이후에 설명할 MyBatis의 가장 큰 장점은 SQL을 직접 사용할 때 동적 쿼리를 쉽게 작성할 수 있다는 점이다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__