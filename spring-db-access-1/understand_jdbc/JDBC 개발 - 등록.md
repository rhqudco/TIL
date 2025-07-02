본격적으로 JDBC를 사용해서 애플리케이션 개발을 해보자.
여기서는 JDBC를 사용해 회원(Member) 데이터를 데이터베이스에 관리하는 기능을 개발하자.

__Member__
```java
package hello.jdbc.domain;  
  
import lombok.Data;  
  
@Data  
public class Member {  
    
  private String memberId;  
  private int money;  
  
  public Member() {  
  }  
  
  public Member(String memberId, int money) {  
    this.memberId = memberId;  
    this.money = money;  
  }  
}
```

회원의 ID와 해당 회원이 소지한 금액을 표현하는 단순한 클래스이다. 앞서 만들어둔 `member`테이블에 데이터를 저장하고 조회할 때 사용한다.

가장 먼저 JDBC를 사용해서 이렇게 만든 회원 객체를 데이터베이스에 저장해보자.

__MemberRepositoryV0 - 회원 등록__
```java
package hello.jdbc.repository;  
  
import hello.jdbc.connection.DBConnectionUtil;  
import hello.jdbc.domain.Member;  
import java.sql.Connection;  
import java.sql.PreparedStatement;  
import java.sql.ResultSet;  
import java.sql.SQLException;  
import java.sql.Statement;  
import lombok.extern.slf4j.Slf4j;  
  
@Slf4j  
public class MemberRepositoryV0 {  
  
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
  
  private void close(Connection con, Statement stmt, ResultSet rs) {  
    if (rs != null) {  
      try {  
        rs.close();  
      } catch (SQLException e) {  
        log.info("error", e);  
      }  
    }  
  
    if (stmt != null) {  
      try {  
        stmt.close();  
      } catch (SQLException e) {  
        log.info("error", e);  
      }  
    }  
  
    if (con != null) {  
      try {  
        con.close();  
      } catch (SQLException e) {  
        log.info("error", e);  
      }  
    }  
  }  
  
  private Connection getConnection() {  
    return DBConnectionUtil.getConnection();  
  }  
  
}
```
- 커넥션 획득
	- `getConnection()`: 이전에 만들어준 `DBConnectionUtil`을 통해서 데이터베이스 커넥션을 획득한다.
- save() - SQL 전달
	- `sql`: 데이터베이스에 전달할 SQL을 정의한다.
		- 여기서는 데이터를 등록해야 하므로 `insert sql`을 준비했다.
	- `con.prepareStatement(sql)`: 데이터베이스에 전달할 SQL과 파라미터로 전달할 데이터들을 준비한다.
		- `sql`: `insert into member(member_id, money) values(?, ?)`
		- `pstmt.setString(1, member.getMemberId()`: SQL의 첫 번째 `?`에 값을 지정한다. 문자이므로 `setString`을 사용한다.
		- `pstmt.setInt(2, member.getMoney())`: SQL의 두 번째 `?`에 값을 지정한다. `Int`형 숫자이므로 `setInt`를 지정한다.
	- `pstmt.executeUpdate()`: `Statement`를 통해 준비된 SQL을 커넥션 통해 실제 데이터베이스에 전달한다. 참고로 `executeUpdate()`은 `int`를 반환하는데 영향받은 `DB row`수를 반환한다.
		- 여기서는 하나의 row를 등록했기에 1을 반환한다.

```java
// executeUpdate()
int executeUpdate() throws SQLException;
```

#### 리소스 정리
쿼리를 실행하고 나면 리소스를 정리해야 한다. 여기서는 `Connection`, `PreparedStatement`를 사용했다. 리소스를 정리할 때는 항상 역순으로 해야한다.
`Connection`을 먼저 획득하고 `Connection`을 통해 `PreparedStatement`를 만들었기 때문에, `PreparedStatement`를 먼저 종료하고, 그 다음에 `Connection`을 종료하면 된다.
참고로 여기서 사용하지 않은 `ResultSet`은 결과 조회 시 사용한다.

- 리소스 획득 순서
	- `Connection` -> `PreparedStatement`
- 리소스 반환 순서
	- `PreparedStatement` -> `Connection`

> 주의
> 리소스 정리는 꼭!!!!!!!!!!!! 해야한다. 따라서 예외가 발생하든, 하지 않든 항상 수행되어야 하므로, `finally` 구문에 주의해서 작성해야 한다.
> 만약 이 부분을 놓치게 되면 커넥션이 끊어지지 않고 계속 유지되는 문제가 발생할 수 있다.
> 이런 것을 리소스 누수라고 하는데, 결과적으로 커넥션 부족으로 인한 장애 발생 가능성이 생긴다.

> 참고
> `PreparedStatement`는 `Statement`의 자식 타입인데, `?`를 통한 파라미터 바인딩을 가능하게 해준다.
> 참고로 SQL Injection 공격을 예방하려면 `PreparedStatement`를 통한 파라미터 바인딩 방식을 사용해야 한다.

__MemberRepositoryV0Test - 회원 등록__
```java
package hello.jdbc.repository;  
  
import static org.junit.jupiter.api.Assertions.*;  
  
import hello.jdbc.domain.Member;  
import java.sql.SQLException;  
import org.junit.jupiter.api.Test;  
  
class MemberRepositoryV0Test {  
  
  MemberRepositoryV0 repository = new MemberRepositoryV0();  
  
  @Test  
  void crud() throws SQLException {  
    // save  
    Member member = new Member("memberV0", 10000);  
    repository.save(member);  
  }  
}
```

__실행 결과__
데이터베이스에서 `select * from member;` 쿼리를 실행하면 데이터가 저장된 것을 확인할 수 있다.
참고로 이 테스트는 두 번 실행하면 PK 중복 오류가 발생한다. 이 경우 `delete from member` 쿼리로 데이터를 삭제한 후 다시 실행하자.

__PK 중복 오류__
```
org.h2.jdbc.JdbcSQLIntegrityConstraintViolationException: Unique index or primary key violation: "PUBLIC.PRIMARY_KEY_8 ON PUBLIC.MEMBER(MEMBER_ID) VALUES ( /* 3 */ 'memberV0' )"; SQL statement:
insert into member(member_id, money) values(?, ?) [23505-232]
```


__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 1편__
