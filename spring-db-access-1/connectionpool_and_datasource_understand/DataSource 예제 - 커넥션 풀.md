이번에는 `DataSource`를 통해 커넥션 풀을 사용하는 예제를 알아보자.

__ConnectionTest - 데이터 소스 커넥션 풀 추가__
```java
import com.zaxxer.hikari.HikariDataSource;

@Test  
void dataSourceConnectionPool() throws SQLException, InterruptedException {  
  // 커넥션 풀링: HikariProxyConnection(Proxy) -> JdbcConnection(Target)  
  HikariDataSource dataSource = new HikariDataSource();  
  dataSource.setJdbcUrl(URL);  
  dataSource.setUsername(USER);  
  dataSource.setPassword(PASSWORD);  
  dataSource.setMaximumPoolSize(10);  
  dataSource.setPoolName("MyPool");  
    
  useDataSource(dataSource);  
  Thread.sleep(1000);  
}
```

- HikariCP 커넥션 풀을 사용한다.
- `HikariDataSource`는 `DataSource`인터페이스를 구현하고 있다.
- 커넥션 풀 최대 사이즈를 10으로 지정하고, 풀의 이름을 `MyPool`이라고 지정했다.
- 커넥션 풀에서 커넥션을 생성하는 작업은 애플리케이션 실행 속도에 영향을 주지 않기 위해 별도의 쓰레드에서 작동한다.
- 별도의 쓰레드에서 동작하기 때문에 테스트가 먼저 종료되어 버린다.
- 예제처럼 `Thread.sleep`을 통해 대기 시간을 주어야 커넥션 풀에 커넥션이 생성되는 로그를 확인할 수 있다.

__실행 결과__
```
#커넥션 풀 초기화 정보 출력  
HikariConfig - MyPool - configuration:  
HikariConfig - maximumPoolSize................................10
HikariConfig - poolName................................"MyPool"

# 커넥션 풀 전용 쓰레드가 커넥션 풀에 커넥션을 10개 채움
MyPool - Added connection conn0: url=jdbc:h2:tcp://localhost/~/test user=SA
MyPool - Added connection conn1: url=jdbc:h2:tcp://localhost/~/test user=SA
...
MyPool - Added connection conn9: url=jdbc:h2:tcp://localhost/~/test user=SA

#커넥션 풀에서 커넥션 획득1  
ConnectionTest - connection = HikariProxyConnection@261748192 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class com.zaxxer.hikari.pool.HikariProxyConnection  
#커넥션 풀에서 커넥션 획득2  
ConnectionTest - connection = HikariProxyConnection@970535245 wrapping conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class com.zaxxer.hikari.pool.HikariProxyConnection

MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)
```

실행 결과를 분석하면 아래와 같다

__HikariConfig__
`HikariCP`관련 설정을 확인할 수 있다. 풀의 이름(`MyPool`)과 최대 풀 수(`10`)을 확인할 수 있다.

__MyPool connection adder__
별도의 쓰레드를 사용해서 커넥션 풀에 커넥션을 채우고 있는 것을 확인할 수 있다. 이 쓰레드는 커넥션 풀에 커넥션을 최 대 풀 수(`10`)까지 채운다.
그렇다면 왜 별도의 쓰레드를 사용해서 커넥션 풀에 커넥션을 채우는 것일까?
커넥션 풀에 커넥션을 채우는 것은 상대적으로 오래 걸리는 일이다. 애플리케이션을 실행할 때 커넥션 풀을 채울 때 까지 마냥 대기하고 있다면 애플리케이션 실행 시간이 늦어진다.
따라서 이렇게 별도의 쓰레드를 사용해서 커넥션 풀을 채워야 애플리케이션 실행 시간에 영향을 주지 않는다.

__커넥션 풀에서 커넥션 획득__
커넥션 풀에서 커넥션을 획득하고 그 결과를 출력했다.
여기서는 커넥션 풀에서 커넥션을 2개 획득하고 반환하지는 않았다. 따라서 풀에 있는 10개의 커넥션 중에 2개를 가지고 있는 상태이다. 그래서 마지막 로그를 보면 사용중인 커넥션 `active=2`, 풀에서 대기 상태인 커넥션 `idle=8`을 확인할 수 있다.  
`MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)`

> 참고
> HikariCP 커넥션 풀에 대한 더 자세한 내용은 다음 공식 사이트를 참고하자.
> https://github.com/brettwooldridge/HikariCP


__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 1편__