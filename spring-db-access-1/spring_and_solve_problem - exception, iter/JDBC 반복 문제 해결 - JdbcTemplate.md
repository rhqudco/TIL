지금까지 서비스 계층의 순수함을 유지하기 위해 수 많은 노력을 했고, 덕분에 서비스 계층의 순수함을 유지하게 되었다.
이번에는 리포지토리에서 JDBC를 사용하기 때문에 발생하는 반복 문제를 해결해보자.

**JDBC 반복 문제**
- 커넥션 조회, 커넥션 동기화
- `PreparedStatement` 생성 및 파라미터 바인딩
- 쿼리 실행
- 결과 바인딩
- 예외 발생 시 스프링 예외 변환기 실행
- 리소스 종료

리포지토리의 각각의 메서드를 살펴보면 많은 부분이 반복된다. 이런 반복을 효과적으로 처리하는 방법이 바로 `템플릿 콜백 패턴`이다.
스프링은 JDBC의 반복 문제를 해결하기 위해 `JdbcTemplate`이라는 템플릿을 제공한다.
지금은 전체 구조와, 이 기능을 사용해서 반복 코드를 제거하는 것에 초점을 맞추자.

**MemberRepositoryV5**
```java
package hello.jdbc.repository;  
  
import hello.jdbc.domain.Member;  
import javax.sql.DataSource;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.jdbc.core.RowMapper;  
  
/*  
* JdbcTemplate 사용  
* */  
@Slf4j  
public class MemberRepositoryV5 implements MemberRepository {  
  private final JdbcTemplate template;  
  
  public MemberRepositoryV5(DataSource dataSource) {  
    this.template = new JdbcTemplate(dataSource);  
  }  
  
  @Override  
  public Member save(Member member) {  
    String sql = "insert into member(member_id, money) values(?, ?)";  
    template.update(sql, member.getMemberId(), member.getMoney());  
    return member;  
  }  
  
  @Override  
  public Member findById(String memberId) {  
    String sql = "select * from member where member_id = ?";  
    return template.queryForObject(sql, memberRowMapper(), memberId);  
  }  
  
  @Override  
  public void update(String memberId, int money) {  
    String sql = "update member set money=? where member_id=?";  
    template.update(sql, money, memberId);  
  }  
  
  @Override  
  public void delete(String memberId) {  
    String sql = "delete from member where member_id=?";  
    template.update(sql, memberId);  
  }  
  
  private RowMapper<Member> memberRowMapper() {  
    return (rs, rowNum) -> { // rs = resultSet, rowNum = 몇 번째 row 인지  
      Member member = new Member();  
      member.setMemberId(rs.getString("member_id"));  
      member.setMoney(rs.getInt("money"));  
      return member;  
    };  
  }  
}
```

**MemberServiceV4Test - 수정**
```java
@Bean  
MemberRepository memberRepository() {  
  // return new MemberRepositoryV4_1(dataSource); // 단순 예외 반환  
  // return new MemberRepositoryV4_2(dataSource); // 스프링 예외 반환  
  return new MemberRepositoryV5(dataSource); // JdbcTemplate  
}
```

`MemberRepository` 인터페이스가 제공되므로 등록하는 빈만 `MemberRepositoryV5`로 변경해서 등록하면 된다.
`JdbcTemplate`은 JDBC로 개발할 때 발생하는 반복을 대부분 해결해준다.
그 뿐만 아니라 지금까지 학습했던, **트랜잭션을 위한 커넥션 동기화**는 물론이고, 예외 발생시 **스프링 예외 변환기**도 자동으로 실행해준다.

__출처: 인프런 김영한 지식공유자님의 강의 - 스프링 DB 1편__