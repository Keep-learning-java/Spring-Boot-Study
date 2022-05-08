 # 스프링 통합 테스트
 - **@SpringBootTest** : 단위테스트와 같이 기능 검증이 아닌, 스프링에서 실제 운영 환경과 같이 전체 플로우가 제대로 동작하는지 보기 위한 통합테스트이다.
 - **@Transactional** : Test case에 이 어노테이션이 있으면 테스트 시작 전에 transaction을 먼저 시작한다. DB에 데이터를 넣은 후 테스트가 끝나면 롤백을 해서 DB 안에 데이터가 남지 않는다. 즉, 다음 테스트에 영향을 주지 않는다.
    > Transaction이란 데이터베이스의 상태를 변화시키기 위해 수행하는 작업의 단위를 의미한다. 상태를 변화시킨다는 것은 SQL 질의어를 통해 DB에 접근하는 것을 의미하며 대표적으로 `SELECT`, `INSERT`, `DELETE`, `UPDATE`가 있다. 
    ```text
    예시) 사용자 A가 사용자 B에게 만원을 송금한다.

    * 이때 DB 작업
    - 1. 사용자 A의 계좌에서 만원을 차감한다 : UPDATE 문을 사용해 사용자 A의 잔고를 변경
    - 2. 사용자 B의 계좌에 만원을 추가한다 : UPDATE 문을 사용해 사용자 B의 잔고를 변경

    현재 작업 단위 : 출금 UPDATE문 + 입금 UPDATE문
    → 이를 통틀어 하나의 트랜잭션이라고 한다.
    - 위 두 쿼리문 모두 성공적으로 완료되어야만 "하나의 작업(트랜잭션)"이 완료되는 것이다. `Commit`
    - 작업 단위에 속하는 쿼리 중 하나라도 실패하면 모든 쿼리문을 취소하고 이전 상태로 돌려놓아야한다. `Rollback`
    ```

# JDBC
- 데이터베이스 종류에 상관 없이 자바를 이용하여 데이터베이스에 접속, SQL 질의문의 실행과 그 결과를 데이터로 핸들링하는 프로그래밍을 하기 위해 사용되는 API이다. 즉, JDBC를 이용하면 데이터베이스에 비종속적인 DB 연동 로직을 구현할 수 있다.
- 대략 다음의 과정으로 작업을 수행하게 된다.
  ```text
  1. 쿼리 설정
  2. connection 설정
  3. 쿼리 실행 준비
  4. 쿼리 실행
  5. connection 닫기
  ```

## Spring JdbcTemplate
JdbcTemplate은 GoF 디자인 패턴 중 **템플릿 메소드 패턴**이 적용된 클래스이다. 템플릿 메소드 패턴은 복잡하고 반복되는 알고리즘을 캡슐화해서 재사용하는 패턴이다. 따라서 반복되는 알고리즘을 템플릿 메소드로 캡슐화할 수 있어서 JDBC처럼 코딩 순서가 정형화된 기술에 유용하게 사용된다.

따라서 개발자들은 JdbcTemplate 클래스가 어떻게 JDBC API를 이용하는지 몰라도 DB에 대한 접근과 데이터 핸들링이 가능하다.

cf) 템플릿 메소드 패턴 : https://steady-coding.tistory.com/384

### JDBC Template 사용방법
- Query for object
  - Query for an Integer
    ```java
        // 모든 학생의 수
        String SQL = "select count(*) from Student";
        int rowCount = jdbcTemplateObject.qeuryForObject(SQL, Integer.class);
    ```
  - Query for an String
    ```java
        // 해당 학번(10)에 해당하는 학생의 이름
        String SQL = "select name from Student where id = ?"; 
        String name = jdbcTemplateObject.queryForObject(SQL, new Object[]{10}, String.class);
    ``` 
- Update
  - Inserting a row int the table
    ```java
        /* 이름이 Zara, 학번이 11인 학생을 삽입 */
        String SQL = "insert into Student (name, age) values (?, ?)"; 
        jdbcTemplateObject.update(SQL, new Object[]{"Zara", 11});
    ```
  - Updating a row into the table
    ```java
        /* 학번이 10인 학생의 이름은 Zara로 수정 */
        String SQL = "update Student set name = ? where id = ?"; 
        jdbcTemplateObject.update(SQL, new Object[]{"Zara", 10});
        https://gmlwjd9405.github.io/2018/12/19/jdbctemplate-usage.html
    ```
  - Deleteing a row into the table
    ```java
        /* 학번이 20인 학생을 삭제 */
        String SQL = "delete from Student where id = ?"; 
        jdbcTemplateObject.update(SQL, new Object[]{20});
        https://gmlwjd9405.github.io/2018/12/19/jdbctemplate-usage.html
    ```

### 아쉬운 점
JDBC -> JdbcTemplate로 바꾸면 반복적인 코드는 줄게 되지만, SQL은 여전히 개발자가 직접 작성해야 한다.
# JPA
- JPA는 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.
- JPA는 자바의 표준 인터페이스이고, JPA의 구현체인 hibernate를 쓰는 것이다.
- JPA는 객체와 ORM(Object Relational Mapping) 기술로, 어노테이션을 통해 매핑한다.
- JPA를 사용하면, SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환을 할 수 있다.
- JPA를 사용하면 개발 생산성을 높일 수 있다.

## JPA 기반으로 코드 작성하기
```java
// ...\domain\Member
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

// ...\repository\JpaMemberRepository
public class JpaMemberRepository implements MemberRepository {
    private final EntityManager em;

    public JpaMemberRepository(EntityManger em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
        .setParameter("name", name)
        .getResultList();

        return result.stream().findAny();
    }
}
```
- Entity : DB에서 영속적으로 저장된 데이터를 자바 객체로 매핑하여 **인스턴스의 형태로** 존재하는 데이터를 말한다. 데이터베이스의 테이블과 자바 클래스가 매핑이 된다고 생각하면 된다.
- Id : 식별키(PK, Primary Key)를 의미한다. 주로 `@GeneratedValue`와 함께 식별키를 어떤 전략으로 생성하는지 명시한다.
- IDENTITY : 기본키 생성 방식 자체를 데이터베이스에 위임하는 방식이다. 위 코드에서 데이터베이스에 이름만 넣으면 id는 데이터베이스에서 알아서 자동으로 생성된다.