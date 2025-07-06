이번에는 애플리케이션에 `DataSource`를 적용해보자.

기존 코드 유지를 위해 기존 코드를 복사해서 새로 만든다.
`MemberRepositoryV0` -> `MemberRepositoryV1`
`MemberRepositoryV0Test` -> `MemberRepositoryV1Test`

__MemberRepositoryV1__
```java
package hello.jdbc.repository;  
  
import hello.jdbc.domain.Member;  
import java.sql.Connection;  
import java.sql.PreparedStatement;  
import java.sql.ResultSet;  
import java.sql.SQLException;  
import java.sql.Statement;  
import java.util.NoSuchElementException;  
import javax.sql.DataSource;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.jdbc.support.JdbcUtils;  
  
@Slf4j  
/*  
* JDBC - DataSource 사용, JdbcUtils 사용  
* */  
public class MemberRepositoryV1 {  
  
  private final DataSource dataSource;  
  
  public MemberRepositoryV1(DataSource dataSource) {  
    this.dataSource = dataSource;  
  }  
  
  public Member save(Member member) throws SQLException {  
    String sql = "insert into member(member_id, money) values(?, ?)";  
  
    Connection con = null;  
    PreparedStatement pstmt = null;  
  
    try {  
      con = getConnection();  
      pstmt = con.prepareStatement(sql);  
      pstmt.setString(1, member.getMemberId());  
      pstmt.setInt(2, member.getMoney());  
      pstmt.executeUpdate();  
      return member;  
    } catch (SQLException e) {  
      log.error("db error", e);  
      throw e;  
    } finally {  
      close(con, pstmt, null);  
    }  
  }  
  
  public Member findById(String memberId) throws SQLException {  
    String sql = "select * from member where member_id = ?";  
  
    Connection con = null;  
    PreparedStatement pstmt = null;  
    ResultSet rs = null;  
  
    try {  
      con = getConnection();  
      pstmt = con.prepareStatement(sql);  
      pstmt.setString(1, memberId);  
  
      rs = pstmt.executeQuery();  
  
      if (rs.next()) {  
        Member member = new Member();  
        member.setMemberId(rs.getString("member_id"));  
        member.setMoney(rs.getInt("money"));  
        return member;  
      } else {  
        throw new NoSuchElementException("member not found memberId = " + memberId);  
      }  
    } catch (SQLException e) {  
      log.error("db error", e);  
      throw e;  
    } finally {  
      close(con, pstmt, rs);  
    }  
  }  
  
  public void update(String memberId, int money) throws SQLException {  
    String sql = "update member set money = ? where member_id = ?";  
  
    Connection con = null;  
    PreparedStatement pstmt = null;  
  
    try {  
      con = getConnection();  
      pstmt = con.prepareStatement(sql);  
      pstmt.setInt(1, money);  
      pstmt.setString(2, memberId);  
      int resultSize = pstmt.executeUpdate();  
      log.info("resultSize = {}", resultSize);  
    } catch (SQLException e) {  
      log.error("db error", e);  
      throw e;  
    } finally {  
      close(con, pstmt, null);  
    }  
  }  
  
  public void delete(String memberId) throws SQLException {  
    String sql = "delete from member where member_id = ?";  
  
    Connection con = null;  
    PreparedStatement pstmt = null;  
  
    try {  
      con = getConnection();  
      pstmt = con.prepareStatement(sql);  
      pstmt.setString(1, memberId);  
  
      pstmt.executeUpdate();  
    } catch (SQLException e) {  
      log.error("db error", e);  
      throw e;  
    } finally {  
      close(con, pstmt, null);  
    }  
  }  
  
  private void close(Connection con, Statement stmt, ResultSet rs) {  
    JdbcUtils.closeResultSet(rs);  
    JdbcUtils.closeStatement(stmt);  
    JdbcUtils.closeConnection(con);  
  }  
  private Connection getConnection() throws SQLException {  
    Connection con = dataSource.getConnection();
    log.info("get connection = {}, class = {}", con, con.getClass());
    return con;  
  }  
  
}
```

- `DataSource`의존관계 주입  
	- 외부에서 `DataSource`를 주입 받아서 사용한다.
	- 이제 직접 만든 `DBConnectionUtil`을 사용하지 않아도 된다.
	- `DataSource`는 표준 인터페이스 이기 때문에 `DriverManagerDataSource`에서 `HikariDataSource`로 변경되어도 해당 코드를 변경하지 않아도 된다.
- `JdbcUtils` 편의 메서드  
	- 스프링은 JDBC를 편리하게 다룰 수 있는 `JdbcUtils`라는 편의 메서드를 제공한다.
	- `JdbcUtils`을 사용하면 커넥션을 좀 더 편리하게 닫을 수 있다.

__MemberRepositoryV1Test__
```java
package hello.jdbc.repository;  
  
import static hello.jdbc.connection.ConnectionConst.PASSWORD;  
import static hello.jdbc.connection.ConnectionConst.URL;  
import static hello.jdbc.connection.ConnectionConst.USER;  
import static org.assertj.core.api.Assertions.assertThat;  
import static org.assertj.core.api.Assertions.assertThatThrownBy;  
  
import com.zaxxer.hikari.HikariDataSource;  
import hello.jdbc.domain.Member;  
import java.sql.SQLException;  
import java.util.NoSuchElementException;  
import lombok.extern.slf4j.Slf4j;  
import org.junit.jupiter.api.BeforeEach;  
import org.junit.jupiter.api.Test;  
import org.springframework.jdbc.datasource.DriverManagerDataSource;
  
@Slf4j  
class MemberRepositoryV1Test {  
  
  MemberRepositoryV1 repository;  
  
  @BeforeEach  
  void beforeEach() {  
    // 기본 DriverManager - 항상 새로운 커넥션 획득  
    // DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USER, PASSWORD);  
    // 커넥션 풀링: HikariProxyConnection -> JdbcConnection    HikariDataSource dataSource = new HikariDataSource();  
    dataSource.setJdbcUrl(URL);  
    dataSource.setUsername(USER);  
    dataSource.setPassword(PASSWORD);  
  
    repository = new MemberRepositoryV1(dataSource);  
  }  
  
  @Test  
  void crud() throws SQLException {  
    // save  
    String memberId = "memberV5";  
    Member member = new Member(memberId, 10000);  
    repository.save(member);  
  
    // findById  
    Member findMember = repository.findById(memberId);  
    log.info("findMember = {}", findMember);  
    assertThat(findMember).isEqualTo(member);  
  
    // update: money 10000 -> 20000  
    repository.update(member.getMemberId(), 20000);  
    Member updatedMember = repository.findById(member.getMemberId());  
    assertThat(updatedMember.getMoney()).isEqualTo(20000);  
  
    // delete  
    repository.delete(updatedMember.getMemberId());  
    assertThatThrownBy(() -> repository.findById(member.getMemberId())).isInstanceOf(  
        NoSuchElementException.class);  
  }  
}
```

__DriverManagerDataSource 사용__
```
get connection = conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
get connection = conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
...
get connection = conn5: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
```
- `DriverManagerDataSource`를 사용하면 `conn0 ~ 5` 번호를 통해서 항상 새로운 커넥션이 생성되어서 사용 되는 것을 확인할 수 있다.

__HikariDataSource 사용__
```
get connection = HikariProxyConnection@1171434979 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class com.zaxxer.hikari.pool.HikariProxyConnection
get connection = HikariProxyConnection@916835004 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class com.zaxxer.hikari.pool.HikariProxyConnection
get connection = HikariProxyConnection@1756573246 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class com.zaxxer.hikari.pool.HikariProxyConnection
get connection = HikariProxyConnection@198112003 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class com.zaxxer.hikari.pool.HikariProxyConnection
get connection = HikariProxyConnection@1265508963 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class com.zaxxer.hikari.pool.HikariProxyConnection
get connection = HikariProxyConnection@1398241764 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class com.zaxxer.hikari.pool.HikariProxyConnection
```
- 커넥션 풀 사용시 `conn0`커넥션이 재사용 된 것을 확인할 수 있다.
- 테스트는 순서대로 실행되기 때문에 커넥션을 사용하고 다시 돌려주는 것을 반복한다. 따라서 `conn0`만 사용된다.
- 웹 애플리케이션에 동시에 여러 요청이 들어오면 여러 쓰레드에서 커넥션 풀의 커넥션을 다양하게 가져가는 상황을 확인할 수 있다.
- 객체의 인스턴스 주소는 다르지만, 위 예제에서는 사실 같은 커넥션을 사용한다.
	- Hikari에서 생성된 커넥션은 커넥션 풀에 담겨 있다가, 커넥션 제공(반환) 시에는 `HikariProxyConnection`에 담아서 반환된다.
	- 그래서 객체의 인스턴스 주소는 다르게 표현된다.

__DI__
- `DriverManagerDataSource` -> `HikariDataSource`로 변경해도 `MemberRepositoryV1`의 코드는 전혀 변경하지 않아도 된다.
- `MemberRepositoryV1`는 `DataSource`인터페이스에만 의존하기 때문이다. 이것이 `DataSource`를 사용하는 장점이다. (DI + OCP)
__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 1편__