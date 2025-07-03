수정과 삭제는 등록과 비슷하다. 등록, 수정, 삭제처럼 데이터를 변경하는 쿼리는 `executeUpdate()`를 사용하면 된다.

__MemberRepositoryV0 - 회원 수정 추가__
```java
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
```

`executeUpdate()`는 쿼리를 실행하고 영향받은 row 수를 반환한다.
여기서는 하나의 데이터만 변경하기 때문에 결과로 1이 반환된다. 만약 회원이 100명이고, 모든 회원의 데이터를 한번에 수정하는 update sql을 실행하면 결과는 100이 된다.

__MemberRepositoryV0Test - 회원 수정 추가__
```java
package hello.jdbc.repository;  
  
import static org.assertj.core.api.Assertions.*;  
  
import hello.jdbc.domain.Member;  
import java.sql.SQLException;  
import lombok.extern.slf4j.Slf4j;  
import org.junit.jupiter.api.Test;  
  
@Slf4j  
class MemberRepositoryV0Test {  
  
  MemberRepositoryV0 repository = new MemberRepositoryV0();  
  
  @Test  
  void crud() throws SQLException {  
    // save  
    String memberId = "memberV3";  
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
  }  
}
```
회원 데이터의 `money`를 10000 -> 20000으로 수정하고, DB에서 데이터를 다시 조회해서 20000으로 변경 되었는지 검증

__실행 로그__
```
get connection = conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection # -> 첫 번째 커넥션(save)
get connection = conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection # -> 두 번째 커넥션(findById)
findMember = Member(memberId=memberV3, money=10000)
get connection = conn2: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection # -> 세 번째 커넥션(update)
resultSize = 1
get connection = conn3: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection # -> 네 번째 커넥션(findById)
```

이번에는 회원 삭제를 만들어보자

__MemberRepositoryV0 - 회원 삭제 추가__
```java
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
```

__MemberRepositoryV0Test - 회원 삭제 추가__
```java
package hello.jdbc.repository;  
  
import static org.assertj.core.api.Assertions.*;  
  
import hello.jdbc.domain.Member;  
import java.sql.SQLException;  
import java.util.NoSuchElementException;  
import lombok.extern.slf4j.Slf4j;  
import org.junit.jupiter.api.Test;  
  
@Slf4j  
class MemberRepositoryV0Test {  
  
  MemberRepositoryV0 repository = new MemberRepositoryV0();  
  
  @Test  
  void crud() throws SQLException {  
    // save  
    String memberId = "memberV4";  
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

회원을 삭제한 다음 `findById()`를 통해서 조회한다. 회원이 없기 때문에 `NoSuchElementException`이 발생한다. `assertThatThrownBy`는 해당 예외가 발생해야 검증에 성공한다.


__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 1편__