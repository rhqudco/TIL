`mybatis-spring-boot-starter` 라이브러리를 사용하면 MyBatis를 스프링과 통합하고, 설정도 아주 간단히 할 수 있다.  
`mybatis-spring-boot-starter` 라이브러리를 사용해서 간단히 설정하는 방법을 알아보자.

`build.gradle` 에 다음 의존 관계를 추가한다.
```groovy
//MyBatis 추가  
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0'
```

참고로 뒤에 버전 정보가 붙는 이유는 스프링 부트가 버전을 관리해주는 공식 라이브러리가 아니기 때문이다. 스프링 부트가 버전을 관리해주는 경우 버전 정보를 붙이지 않아도 최적의 버전을 자동으로 찾아준다.

다음과 같은 라이브러리가 추가된다.  
- `mybatis-spring-boot-starter`: MyBatis를 스프링 부트에서 편리하게 사용할 수 있게 시작하는 라이브러리  
- `mybatis-spring-boot-autoconfigure`: MyBatis와 스프링 부트 설정 라이브러리
- `mybatis-spring`: MyBatis와 스프링을 연동하는 라이브러리  
- `mybatis`: MyBatis 라이브러리

라이브러리 추가는 완료되었다 다음으로 설정을 해보자.

**설정**
`application.properties`에 다음 설정을 추가하자.

**주의!**
웹 애플리케이션을 실행하는 `main`, 테스트를 실행하는 `test` 각 위치의 `application.properties`를 모두 수정해주어야 한다.
설정을 변경해도 반영이 안된다면 이 부분을 꼭! 확인하자.

**application.properties - main**
```properties
#MyBatis
mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```

**application.properties - test**
```properties
#MyBatis
mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```

- `mybatis.type-aliases-package`
	- 마이바티스에서 타입 정보를 사용할 때는 패키지 이름을 적어주어야 하는데, 여기에 명시하면 패키지 이름을 생략할 수 있다.
	- 지정한 패키지와 그 하위 패키지가 자동으로 인식된다.
	- 여러 위치를 지정하려면 `,`, `;`로 구분하면 된다.
- `mybatis.configuration.map-underscore-to-camel-case`  
	- JdbcTemplate의 `BeanPropertyRowMapper`에서 처럼 언더바를 카멜로 자동 변경해주는 기능을 활성화 한다.
	- 바로 다음에 설명하는 관례의 불일치 내용을 참고하자.
- `logging.level.hello.itemservice.repository.mybatis=trace`
	- MyBatis에서 실행되는 쿼리 로그를 확인할 수 있다.

**관례의 불일치**
자바 객체에는 주로 카멜(`camelCase`) 표기법을 사용한다. `itemName` 처럼 중간에 낙타 봉이 올라와 있는 표기법이다.
반면에 관계형 데이터베이스에서는 주로 언더스코어를 사용하는 `snake_case` 표기법을 사용한다.
`item_name` 처럼 중간에 언더스코어를 사용하는 표기법이다.  
이렇게 관례로 많이 사용하다 보니 `map-underscore-to-camel-case` 기능을 활성화 하면 언더스코어 표기법을 카멜로 자동 변환해준다.
따라서 DB에서 `select item_name`으로 조회해도 객체의 `itemName` (`setItemName()`) 속성에 값이 정상 입력된다.
정리하면 해당 옵션을 켜면 `snake_case`는 자동으로 해결되니 그냥 두면 되고, 컬럼 이름과 객체 이름이 완전히 다른 경우에는 조회 SQL에서 별칭을 사용하면 된다.

예)
- DB `select item_name`
- 객체 `name`

별칭을 통한 해결방안
`select item_name as name`

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 2편__