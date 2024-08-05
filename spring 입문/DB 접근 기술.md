## 순수 JDBC
### JDBC 설정
- h2 설치
- gradle에 아래와 같이 연결 설정 추가
- `implementation 'org.springframework.boot:spring-boot-starter-jdbc'
runtimeOnly 'com.h2database:h2'`
- `application.properties` 파일에 다음 추가
- `spring.datasource.url=jdbc:h2:tcp://localhost/~/test
 spring.datasource.driver-class-name=org.h2.Driver
 spring.datasource.username=sa`
 
 > 스프링부트 2.4부터 `spring.datasource.username=sa` 를 꼭 추가해주어야한다. 그렇지 않으면 `Wrong user name or password` 오류가 발생한다. 
 모든 설정 뒤에 공백이 있으면 안된다. **공백 주의!**

### JDBC 리포지토리 구현
```
import hello.hellospring.domain.Member;
import org.springframework.jdbc.datasource.DataSourceUtils;
import javax.sql.DataSource;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
public class JdbcMemberRepository implements MemberRepository {
   private final DataSource dataSource;
   public JdbcMemberRepository(DataSource dataSource) {
   		this.dataSource = dataSource;
   }
   
   @Override
   public Member save(Member member) {
       String sql = "insert into member(name) values(?)";
       Connection conn = null;
       PreparedStatement pstmt = null;
       ResultSet rs = null;
       try {
           conn = getConnection();
           pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
           pstmt.setString(1, member.getName());
           pstmt.executeUpdate();
           rs = pstmt.getGeneratedKeys();
           if (rs.next()) {
           		member.setId(rs.getLong(1));
           } else {
           		throw new SQLException("id 조회 실패");
           }
           return member;
       } catch (Exception e) {
       		throw new IllegalStateException(e);
       } finally {
       		close(conn, pstmt, rs); // 자원은 꼭 닫아주기
       }
   }
   
   @Override
   public Optional<Member> findById(Long id) {
       String sql = "select * from member where id = ?";
       Connection conn = null;
       PreparedStatement pstmt = null;
       ResultSet rs = null;
       try {
           conn = getConnection();
           pstmt = conn.prepareStatement(sql);
           pstmt.setLong(1, id);
           rs = pstmt.executeQuery();
           if(rs.next()) {
               Member member = new Member();
               member.setId(rs.getLong("id"));
               member.setName(rs.getString("name"));
               return Optional.of(member);
           } else {
           		return Optional.empty();
           }
       } catch (Exception e) {
       		throw new IllegalStateException(e);
       } finally {
       		close(conn, pstmt, rs);
       }
   }
   
   @Override
   public List<Member> findAll() {
       String sql = "select * from member";
       Connection conn = null;
       PreparedStatement pstmt = null;
       ResultSet rs = null;
       try {
           conn = getConnection();
           pstmt = conn.prepareStatement(sql);
           rs = pstmt.executeQuery();
           List<Member> members = new ArrayList<>();
           while(rs.next()) {
               Member member = new Member();
               member.setId(rs.getLong("id"));
               member.setName(rs.getString("name"));
               members.add(member);
           }
           return members;
       } catch (Exception e) {
       		throw new IllegalStateException(e);
       } finally {
       		close(conn, pstmt, rs);
       }
   }
   
   @Override
   public Optional<Member> findByName(String name) {
       String sql = "select * from member where name = ?";
       Connection conn = null;
       PreparedStatement pstmt = null;
       ResultSet rs = null;
       try {
           conn = getConnection();
           pstmt = conn.prepareStatement(sql);
           pstmt.setString(1, name);
           rs = pstmt.executeQuery();
           if(rs.next()) {
               Member member = new Member();
               member.setId(rs.getLong("id"));
               member.setName(rs.getString("name"));
               return Optional.of(member);
           }
       		return Optional.empty();
       } catch (Exception e) {
            throw new IllegalStateException(e);
       } finally {
            close(conn, pstmt, rs);
       }
   }
   private Connection getConnection() {
       return DataSourceUtils.getConnection(dataSource);
       }
       private void close(Connection conn, PreparedStatement pstmt, ResultSet rs) {
           try {
               if (rs != null) {
                    rs.close();
               }
           } catch (SQLException e) {
                e.printStackTrace();
           }
           try {
               if (pstmt != null) {
               pstmt.close();
           }
           } catch (SQLException e) {
                e.printStackTrace();
           }
           try {
               if (conn != null) {
                    close(conn);
                }
           } catch (SQLException e) {
                e.printStackTrace();
           }
       }
       private void close(Connection conn) throws SQLException {
       DataSourceUtils.releaseConnection(conn, dataSource);
    }
}
```

### SpringConfig 수정

```
@Configuration
public class SpringConfig {
   private final DataSource dataSource;
   public SpringConfig(DataSource dataSource) {
   this.dataSource = dataSource;
}
@Bean
public MemberService memberService() {
 	return new MemberService(memberRepository());
}
@Bean
public MemberRepository memberRepository() {
	// return new MemoryMemberRepository();
	return new JdbcMemberRepository(dataSource);
}
```
**스프링 DI를 사용하여서 기존 코드를 손대지 않고 설정파일만으로 구현 클래스를 변경할 수 있다.**


~~JDBC는 예전방식이고 효율적이지 않으므로 참고만 하자~~


<br>
<br>

## 스프링 통합 테스트
지난번 단위테스트와 달리 DB를 연결한 통합 테스트 예제이다.
```
@SpringBootTest
@Transactional
class MemberServiceIntegrationTest {
	@Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;
    
    @Test
void 회원가입() {
	//given
    Member member = new Member();
    member.setName("hello");
   
    //when
    Long saveId = memnerService.join(member);
    
    //then
    Member findMember = memberService.findOne(saveId).get();
    assertThat(memebr,getName()).isEqualTo(findMember.getName());
}

@Test
public void 중복_회원_예외() {
	//given
    Member member1 = new Member();
    member1.setName("spring");
    
    Member member2 = new Member();
    member2.setName("spring");
    
    //when
    memberService.join(member1);
    
    /* 아래와 같이 예외를 던져 같은 예외 메시지가 발생하는지 확인할 수 있지만 더 편리한 메서드를 스프링에서 제공함
    try {
    	memberService.join(member2);
        fail();
    } catch (IllegalStateException e) {
    	assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
    }
}
```

- `@SpringBootTest` : 스프링 컨테이너와 테스트를 함께 실행한다.
- `@Transactional` : 테스트 케이스에 한해서 테스트 시작 전 트랜잭션을 시작하고, 완료 후에 항상 롤백하여 다음 테스트에 영향을 주지 않게 한다.
- `@Autowired` : 테스트 케이스에서는 코드를 중간에서 변경할 필요가 없으므로 생성자가 아닌 `field` 수준에서 선언해도 무방하다.

<br>
<br>

## 스프링 JDBCTemplate
### 순수 JDBC와 공통점, 차이점
- 환경설정은 순수 JDBC와 동일하다.
- 순수 JDBC에서 중복되는 코드들을 최소화 한 것이다.

### 스프링 JDBCTemplate 회원 리포지토리
```
public class JdbcTemplateMemberRepository implements MemberRepository {
   private final JdbcTemplate jdbcTemplate;
   public JdbcTemplateMemberRepository(DataSource dataSource) {
		jdbcTemplate = new JdbcTemplate(dataSource);
   }
   
   @Override
   public Member save(Member member) {
       SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
       jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
       Map<String, Object> parameters = new HashMap<>();
       parameters.put("name", member.getName());
       Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
       member.setId(key.longValue());
       return member;
   }
   
   @Override
   public Optional<Member> findById(Long id) {
       List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
       return result.stream().findAny();
   }
   
   @Override
   public List<Member> findAll() {
       return jdbcTemplate.query("select * from member", memberRowMapper());
   }
   
   @Override
   public Optional<Member> findByName(String name) {
       List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
       return result.stream().findAny();
   }
   private RowMapper<Member> memberRowMapper() {
       return (rs, rowNum) -> { // 람다식
           Member member = new Member();
           member.setId(rs.getLong("id"));
           member.setName(rs.getString("name"));
           return member;
       };
   }
}
```
`SpringConfig` 수정 : 기존 리포지토리에서 -> 생성한 `JDBCTemplate` 리포지토리로
```
public MemberRepository memberRepository() {
   // return new MemoryMemberRepository();
   // return new JdbcMemberRepository(dataSource);
   return new JdbcTemplateMemberRepository(dataSource);
}
```

<br>
<br>


## JPA
### 장점 
- 기존의 반복 코드와 기본적인 `SQL`을 `JPA`가 만들어 실행해준다.
- `SQL`과 데이터 중심 설계에서 객체 중심의 설계로 패러다임을 전환할 수 있다.
- `JPA`를 사용하면 개발 생산성을 크게 높이 수 있다.

### 설정
`build.gradle` 파일에 `JPA, h2` 데이터 베이스 관련 라이브러리를 추가한다.
```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

- `show-sql` : `JPA`가 생성하는 `SQL` 을 출력한다.
- `ddl-auto` : `create`를 사용하면 테이블을 자동으로 생성해주고, `none`을 사용하면 해당 기능을 끈다.

### 기존 엔티티에 매핑을 위해 @Entity를 추가한다.

### JPA 회원 리포지토리
```
public class JpaMemberRepository implements MemberRepository {

	private final EntityManager em;
    
    public jpaMemberRepository(EntityManager em) {
    	this.em = em;
    }
    
    public Member save(Member member) {
    	em.persist(member);
        return member;
    }
    
    public Optional<Member> findById(Long id) {
    	Member member = em.find(Member.class, id);
        return Optional.ofNullable(member); //null 값 반환 가능하도록
    }
    
    public List<Member> findAll() {
    	return em.createQuery("select m from Member m", Member.class)
        		.getResultList();
    }
    
    public Optional<Member> findByName(String name) {
    	List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
        		.setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }
}
```
- 주입 받을 것이 하나라면 생성자에서 `@Autowired` 생략 가능하다.

### 서비스 계층에 트랜잭션 추가
```
@Transactional
public class MemberService {}
```

- 이는 메서드가 정상 종료되면 트랜잭션을 `commit`하고, 런타임 예외가 발생하면 `rollback` 한다.
- JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행되어야 한다.
- 트랜젝션을 각 메서드 별로 필요한 곳에만 주어도 무방하다.

### 스프링 빈 설정 변경
`SpringConfig`
```
@Bean
 public MemberRepository memberRepository() {
     // return new MemoryMemberRepository();
     // return new JdbcMemberRepository(dataSource);
     // return new JdbcTemplateMemberRepository(dataSource);
     return new JpaMemberRepository(em);
 }
```

<br>
<br>


## 스프링 데이터 JPA
### 설정
- 앞서 했던 JPA 설정을 그대로 사용
### 스프링 데이터 JPA 회원 리포지토리

```
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {
	Optional<Member> findByName(String name);
}
```

### 스프링 빈 설정 변경
```
@Configuration
public class SpringConfig {
   private final MemberRepository memberRepository;
   public SpringConfig(MemberRepository memberRepository) {
   		this.memberRepository = memberRepository;
   }
   @Bean
   public MemberService memberService() {
   		return new MemberService(memberRepository);
   }
}
```

- 스프링 데이터 `JPA`가 `SpringDataJpaMemberRepository` 를 스프링 빈으로 자동 등록해준다.

스프링 데이터 JPA 제공 클래스

| JpaRepository |
|:-------------:|
| findAll() : List<T> |
| findAll(Sort) : List<T> |
| findAll(Iterable<ID>) : List<T> |
| save(Iterable<ID>) : List<S> |
| flush() |
| saveAndFlush(T) : T |
| deleteInBatch(Iterable<T>) |
| deleteAllInBatch() |
| getOne(ID) : T |
  
> 실무에서는 `JPA`와 스프링 데이터 `JPA`를 기본으로 사용하고 복잡한 동적 쿼리는 `Querydsl`이라는 라이브러리를 사용한다. `Querydsl`을 사용하면 쿼리도 자바 코드로 안전하게 작성할 수 있고 동적 쿼리도 편리하게 작성할 수 있다. 이렇게 해도 해결하기 어려운 쿼리는 `JPA`가 제공하는 네이티브 쿼리를 사용하거나, 앞서 학습한 `JDBC Template`을 사용하면 된다.
