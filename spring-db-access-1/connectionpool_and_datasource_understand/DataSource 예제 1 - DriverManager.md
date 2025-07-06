예제를 통해 `DataSource`를 알아보자.

먼저 기존에 개발했던 `DriverManager`를 통해 커넥션 획득 방법을 확인해보자.

__ConnectionTest - 드라이버 매니저__
```java
package hello.jdbc.connection;  
  
import static hello.jdbc.connection.ConnectionConst.PASSWORD;  
import static hello.jdbc.connection.ConnectionConst.URL;  
import static hello.jdbc.connection.ConnectionConst.USER;  
  
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.SQLException;  
import lombok.extern.slf4j.Slf4j;  
import org.junit.jupiter.api.Test;  
  
@Slf4j  
public class ConnectionTest {  
  
  @Test  
  void driverManager() throws SQLException {  
    Connection con1 = DriverManager.getConnection(URL, USER, PASSWORD);  
    Connection con2 = DriverManager.getConnection(URL, USER, PASSWORD);  
  
    log.info("connection = {}, class = {}", con1, con1.getClass());  
    log.info("connection = {}, class = {}", con2, con2.getClass());  
  }  
  
}
```

__실행 결과__
```
connection = conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
connection = conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
```

이번에는 스프링이 제공하는 `DataSource`가 적용된 `DriverManager`인 `DriverManagerDataSource`를 사용해보자

__ConnectionTest - 데이터소스 드라이버 매니저 추가__
```java
package hello.jdbc.connection;  
  
import static hello.jdbc.connection.ConnectionConst.PASSWORD;  
import static hello.jdbc.connection.ConnectionConst.URL;  
import static hello.jdbc.connection.ConnectionConst.USER;  
  
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.SQLException;  
import javax.sql.DataSource;  
import lombok.extern.slf4j.Slf4j;  
import org.junit.jupiter.api.Test;  
import org.springframework.jdbc.datasource.DriverManagerDataSource;  
  
@Slf4j  
public class ConnectionTest {  
  
  @Test  
  void driverManager() throws SQLException {  
    Connection con1 = DriverManager.getConnection(URL, USER, PASSWORD);  
    Connection con2 = DriverManager.getConnection(URL, USER, PASSWORD);  
  
    log.info("connection = {}, class = {}", con1, con1.getClass());  
    log.info("connection = {}, class = {}", con2, con2.getClass());  
  }  
  
  @Test  
  void dataSourceDriverManager() throws SQLException {  
    // DriverManagerDataSource - 항상 새로운 커넥션 획득  
    DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USER, PASSWORD);  
    useDataSource(dataSource);  
  }  
  
  private void useDataSource(DataSource dataSource) throws SQLException {  
    Connection con1 = dataSource.getConnection();  
    Connection con2 = dataSource.getConnection();  
  
    log.info("connection = {}, class = {}", con1, con1.getClass());  
    log.info("connection = {}, class = {}", con2, con2.getClass());  
  }  
  
}
```

__logback.xml__
```xml
%% resources/logback.xml %%

<configuration>  
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
    <encoder>  
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} -%kvp- %msg%n</pattern>  
    </encoder>  
  </appender>  
  <root level="DEBUG">  
    <appender-ref ref="STDOUT" />  
  </root>  
</configuration>
```

__실행 결과__
```
Creating new JDBC DriverManager Connection to [jdbc:h2:tcp://localhost/~/test]
Creating new JDBC DriverManager Connection to [jdbc:h2:tcp://localhost/~/test]
connection = conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
connection = conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
```

기존 코드와 비슷하지만 `DriverManagerDataSource`는 `DataSource`를 통해 커넥션을 획득할 수 있다. (참고로 `DriverManagerDataSource`는 스프링이 제공하는 코드이다.)

__파라미터 차이__
기존 `DriverManager`를 통해 커넥션 획득하는 방법과 `DataSource`를 통해 커넥션을 획득하는 방법에는 큰 차이가 있다.

__DriverManager__
```java
DriverManager.getConnection(URL, USER, PASSWORD)
DriverManager.getConnection(URL, USER, PASSWORD)
```

__DataSource__
```java
void dataSourceDriverManager() throws SQLException {  
  DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USER, PASSWORD);  
  useDataSource(dataSource);  
}

private void useDataSource(DataSource dataSource) throws SQLException {  
  Connection con1 = dataSource.getConnection();  
  Connection con2 = dataSource.getConnection();  
  
  log.info("connection = {}, class = {}", con1, con1.getClass());  
  log.info("connection = {}, class = {}", con2, con2.getClass());  
}
```

- `DriverManager`는 커넥션 획득할 때마다 `URL`, `USER`, `PASSWORD` 같은 파라미터를 계속 전달해야 한다.
- 반면 `DataSource`를 사용하는 방식은 처음 객체 생성시에 필요한 파라미터를 넘겨두고, 커넥션 획득할 때는 단순하게  `dataSource.getConnection()`만 호출하면 된다.

__설정과 사용의 분리__
- 설정: `DataSource`를 만들고 필요한 속성들을 사용해서 `URL`, `USER`, `PASSWORD`같은 부분을 입력하는 것을 말한다. 이렇게 설정과 관련된 속성들은 한 곳에 있는 것이 향후 변경에 더 유연하게 대처할 수 있다.
- 사용: 설정은 신경쓰지 않고 `DataSource`의 `getConnection()`만 호출해서 사용하면 된다.

__설정과 사용의 분리 설명__
- 이 부분이 작아보이지만 큰 차이를 만들어낸다.
- 필요한 데이터를 `DataSource`가 만들어지는 시점에 미리 다 넣어두게 되면, `DataSource`를 사용하는 곳에서는 `dataSource.getConnection()`만 호출하면 된다.
	- 즉 -> `URL`, `USER`, `PASSWORD`같은 속성들에 의존하지 않아도 된다. 그냥 `DataSource`만 주입받아서 `getConnection()`만 호출하면 된다.
- 쉽게 이야기해서 리포지토리(Repository)는 `DataSource`만 의존하고, 이런 속성을 몰라도 된다.
- 애플리케이션을 개발해보면 보통 설정은 한 곳에서 하지만, 사용은 수 많은 곳에서 하게 된다.
- 덕분에 객체를 설정하는 부분과, 사용하는 부분을 좀 더 명확하게 분리할 수 있다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 1편__