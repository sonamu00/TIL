> [실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84/dashboard) 를 보고 정리한  글입니다.

# 목차
- [목차](#목차)
- [쿼리 메소드 기능](#쿼리-메소드-기능)
  - [메소드 이름으로 쿼리 생성](#메소드-이름으로-쿼리-생성)
    - [순수 JPA Repository와 테스트 코드](#순수-jpa-repository와-테스트-코드)
    - [스프링 데이터 JPA](#스프링-데이터-jpa)
    - [쿼리 메소드 기능](#쿼리-메소드-기능-1)
  - [JPA NamedQuery](#jpa-namedquery)
    - [@NamedQuery 어노테이션으로 Named 쿼리 정의](#namedquery-어노테이션으로-named-쿼리-정의)
    - [JPA를 직접 사용해서 Named 쿼리 호출](#jpa를-직접-사용해서-named-쿼리-호출)
    - [스프링 데이터 JPA로 NamedQuery 사용](#스프링-데이터-jpa로-namedquery-사용)
  - [@Query, Repository 메소드에 쿼리 정의하기](#query-repository-메소드에-쿼리-정의하기)
    - [메서드에 JPQL 쿼리 작성](#메서드에-jpql-쿼리-작성)
  - [@Query, 값, DTO 조회하기](#query-값-dto-조회하기)
    - [단순 값 하나를 조회](#단순-값-하나를-조회)
    - [DTO로 직접 조회](#dto로-직접-조회)
  - [반환 타입](#반환-타입)
    - [조회 결과가 많거나 없는 경우](#조회-결과가-많거나-없는-경우)
  - [순수 JPA 페이징과 정렬](#순수-jpa-페이징과-정렬)
  - [스프링 데이터 JPA 페이징과 정렬](#스프링-데이터-jpa-페이징과-정렬)
    - [페이징과 정렬 파라미터](#페이징과-정렬-파라미터)
    - [특별한 반환타입](#특별한-반환타입)
    - [페이징과 정렬 사용 예제](#페이징과-정렬-사용-예제)
  - [벌크성 수정 쿼리](#벌크성-수정-쿼리)
    - [순수 JPA를 사용한 벌크성 수정 쿼리](#순수-jpa를-사용한-벌크성-수정-쿼리)
    - [스프링 데이터 JPA를 사용한 벌크성 수정 쿼리](#스프링-데이터-jpa를-사용한-벌크성-수정-쿼리)
  - [@EntityGraph](#entitygraph)
    - [순수 JPA 페치 조인](#순수-jpa-페치-조인)
    - [스프링 데이터 JPA의 EntityGraph](#스프링-데이터-jpa의-entitygraph)
  - [JPA Hint & Lock](#jpa-hint--lock)
  - [Lock](#lock)
- [확장 기능](#확장-기능)
  - [사용자 정의 리포지토리 구현](#사용자-정의-리포지토리-구현)
  - [스프링 데이터 JPA 분석](#스프링-데이터-jpa-분석)
    - [스프링 데이터 JPA 구현체 분석](#스프링-데이터-jpa-구현체-분석)
  - [새로운 엔티티를 구별하는 방법](#새로운-엔티티를-구별하는-방법)


# 쿼리 메소드 기능
## 메소드 이름으로 쿼리 생성
메소드 이름을 분석해서 JPQL 쿼리를 실행하는 기능이다.
### 순수 JPA Repository와 테스트 코드
```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
 return em.createQuery("select m from Member m where m.username = :username
and m.age > :age")
 .setParameter("username", username)
 .setParameter("age", age)
 .getResultList();
}
```
```java
@Test
public void findByUsernameAndAgeGreaterThan() {
 Member m1 = new Member("AAA", 10);
 Member m2 = new Member("AAA", 20);
 memberJpaRepository.save(m1);
 memberJpaRepository.save(m2);
 
 List<Member> result =
memberJpaRepository.findByUsernameAndAgeGreaterThan("AAA", 15);
 assertThat(result.get(0).getUsername()).isEqualTo("AAA");
 assertThat(result.get(0).getAge()).isEqualTo(20);
 assertThat(result.size()).isEqualTo(1);
}
```
간단한 조회 기능도 쿼리를 작성해야 한다.
### 스프링 데이터 JPA
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```
스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행한다.
### 쿼리 메소드 기능
- 조회: find…By ,read…By ,query…By get…By
    - Reference: https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation
    - ex) findHelloBy 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
- COUNT: count…By 반환타입 long
- EXISTS: exists…By 반환타입 boolean
- 삭제: delete…By, remove…By 반환타입 long
- DISTINCT: findDistinct, findMemberDistinctBy
- LIMIT: findFirst3, findFirst, findTop, findTop3
    - Reference: https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result

**참고**
- 쿼리 메소드 기능은 엔티티의 필드명이 인터페이스에 정의한 메서드 이름과 다를 경우 애플리케이션 로딩 시점에서 오류가 발생한다.
- 메서드 기능이 3개 이상일 경우에는 메서드 이름이 지저분해지기 때문에 @Query 정의 방식을 사용하길 권장한다.

## JPA NamedQuery
- JPA의 NamedQuery를 호출할 수 있다.
- 실무에서 잘 사용하지 않는 방법이다.
### @NamedQuery 어노테이션으로 Named 쿼리 정의
```java
@Entity
@NamedQuery(
     name="Member.findByUsername",
     query="select m from Member m where m.username = :username")
    public class Member {
     ...
}
```
### JPA를 직접 사용해서 Named 쿼리 호출
```java
public class MemberRepository {
     public List<Member> findByUsername(String username) {
         ...
         List<Member> resultList =
         em.createNamedQuery("Member.findByUsername", Member.class)
         .setParameter("username", username)
         .getResultList();
     }
} 

```

### 스프링 데이터 JPA로 NamedQuery 사용
```java
@Query(name = "Member.findByUsername") //생략 가능
List<Member> findByUsername(@Param("username") String username);
```
`@Query`를 생략하고 메서드 이름만으로 Named 쿼리를 호출할 수 있다.
  - 조건: 선언한 '도메인 클래스' + '.(점)' + '메서드 이름'으로 Named 쿼리를 찾아서 실행한다. 
  - 만약 실행할 Named 쿼리가 없으면 메서드 이름으로 쿼리 생성 전략을 사용한다.
## @Query, Repository 메소드에 쿼리 정의하기
### 메서드에 JPQL 쿼리 작성
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  @Query("select m from Member m where m.username= :username and m.age = :age")
  List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```
- `@org.springframework.data.jpa.repository.Query`어노테이션을 사용한다.
- 실행할 메서드에 정적 쿼리를 직접 작성하므로 이름 없는 Named 쿼리이다.
- JPA Named 쿼리처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있다.

## @Query, 값, DTO 조회하기
### 단순 값 하나를 조회
```java
@Query("select m.username from Member m")
List<String> findUsernameList();
```
JPA 값 타입(`@Embedded`)도 이 방식으로 조회할 수 있다.
### DTO로 직접 조회
```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
 "from Member m join m.team t")
List<MemberDto> findMemberDto();
```
주의점
- DTO로 직접 조회 하려면 JPA의 `new` 명령어를 사용해야 한다.
- 생성자가 맞는 DTO가 필요하다.(JPA와 사용방식이 동일함)
  - ex)
    ```java
    package study.datajpa.repository;
    import lombok.Data;
    
    @Data
    public class MemberDto {
      private Long id;
      private String username;
      private String teamName;
    
      public MemberDto(Long id, String username, String teamName) { //new 명령어와 생성자 동일
      this.id = id;
      this.username = username;
      this.teamName = teamName;
      }
    }
    ```
## 반환 타입
스프링 데이터 JPA는 유연한 반환 타입 지원함
```java
List<Member> findListByUsername(String username); //컬렉션

Member findMemberByUsername(String username); //단건

Optional<Member> findOptionalByUsername(String username); //단건 Optional
```
### 조회 결과가 많거나 없는 경우
```java
 @Test
    public void returnType() {
        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);
        memberRepository.save(m1);
        memberRepository.save(m2);

        Member findMember = memberRepository.findOptionalByUsername("empty");
        System.out.println("findMember = " + findMember); //결과: null
    }
```
- 컬렉션
  - 결과 없음: 빈 컬렉션 반환
- 단건 조회
  - 결과 없음: null 반환
  - 결과가  2건 이상: `javax.persistence.NonUniqueResultException` 예외 발생

JPA는 결과 값이 없으면 `javax.persistence.NoResultException` 예외가 발생하지만, 스프링 데이터 JPA는 단건으로 조회할 때 이 예외가 발생하면 무시하고 대신에 `null`을 반환한다.

## 순수 JPA 페이징과 정렬
다음 조건으로 페이징과 정렬을 사용하는 예제 코드를 보자.
- 검색 조건: 나이가 10살
- 정렬 조건: 이름으로 내림차순
- 페이징 조건: 첫 번쨰 페이지, 페이지당 보여줄 데이터는 3건

*jpa 페이징 리포지토리*
```java
public List<Member> findByPage(int age, int offset, int limit) {
        return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
                .setParameter("age", age)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }

public long totalCount(int age) {
        return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
                .setParameter("age", age)
                .getSingleResult();
    }
```
*JPA 페이징 테스트 코드*
```java
@Test
public void paging() throws Exception {
    //given
    memberJpaRepository.save(new Member("member1", 10));
    memberJpaRepository.save(new Member("member2", 10));
    memberJpaRepository.save(new Member("member3", 10));
    memberJpaRepository.save(new Member("member4", 10));
    memberJpaRepository.save(new Member("member5", 10));

    int age = 10;
    int offset = 0;
    int limit = 3;

    //when
    List<Member> members = memberJpaRepository.findByPage(age, offset, limit);
    long totalCount = memberJpaRepository.totalCount(age);

    //페이지 계산 공식 적용...
    // totalPage = totalCount / size ...
    // 마지막 페이지 ...
    // 최초 페이지 ..
    
    //then
    assertThat(members.size()).isEqualTo(3);
    assertThat(totalCount).isEqualTo(5);
}
```
## 스프링 데이터 JPA 페이징과 정렬
### 페이징과 정렬 파라미터
- `org.springframework.data.domain.Sort`: 정렬 기능
- `org.springframework.data.domain.Pageable`: 페이징 기능(내부에 `Sort` 포함)

### 특별한 반환타입
- `org.springframework.data.domain.Page`: 추가 count 쿼리 결과를 포함하는 페이징
- `org.springframework.data.domain.Slice`: 추가 count 쿼리 없이 다음 페이지만 확인 가능(내부적으로 limit + 1 조회), 모바일 리스트에서 주로 사용함
- `List`(자바 컬렉션): 추가 count 쿼리 없이 결과만 반환
### 페이징과 정렬 사용 예제
```java
Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용

Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 X

List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 X

List<Member> findByUsername(String name, Sort sort);
```
*Page 사용 예제 정의 코드*
```java
public interface MemberRepository extends Repository<Member, Long> {
  Page<Member> findByAge(int age, Pageable pageable);
}
```
*Page 사용 예제 실행 코드*
```java
//페이징 조건과 정렬 조건 설정
@Test
public void page() throws Exception {
  //given
  memberRepository.save(new Member("member1", 10));
  memberRepository.save(new Member("member2", 10));
  memberRepository.save(new Member("member3", 10));
  memberRepository.save(new Member("member4", 10));
  memberRepository.save(new Member("member5", 10));

  //when
  PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC,
  "username"));
  Page<Member> page = memberRepository.findByAge(10, pageRequest);

  //then
  List<Member> content = page.getContent(); //조회된 데이터
  assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
  assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
  assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
  assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
  assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
  assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
}
```
`PageRequest` 생성자의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를 입력한다. 여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다. 참고로 페이지는 0부터 시작한다.

*count 쿼리 분리*
```java
//totalCount를 조회할 때 조인할 필요가 없을 경우 countQuery를 사용하여 성능 최적화
@Query(value = "select m from Member m left join m.team t", countQuery = "select count(m.username) from Member m")
Page<Member> findByAge(int age, Pageable pageable);

```
조인이 많은 쿼리일 때 카운트 쿼리를 분리하지 않으면 count 쿼리가 매우 무거워 진다. 이럴 경우 꼭 `countQuery`를 사용해서 분리하자.
## 벌크성 수정 쿼리
- 특정 하나가 아닌 다수의 데이터를 수정하는 쿼리를 말한다.
- 특정 데이터를 모두 변경할 때 변경 감지를 해서 하나씩 수정하는 것 보다 모두 변경하는 쿼리(벌크성 수정 쿼리)를 한 번에 보내는 것이 효율적이다.
### 순수 JPA를 사용한 벌크성 수정 쿼리
```java
public int bulkAgePlus(int age) {
  int resultCount = em.createQuery(
      "update Member m set m.age = m.age + 1" +
      "where m.age >= :age")
      .setParameter("age", age)
      .executeUpdate();

  return resultCount;
}
```
```java
@Test
public void bulkUpdate() throws Exception {
  //given
  memberJpaRepository.save(new Member("member1", 10));
  memberJpaRepository.save(new Member("member2", 19));
  memberJpaRepository.save(new Member("member3", 20));
  memberJpaRepository.save(new Member("member4", 21));
  memberJpaRepository.save(new Member("member5", 40));
  
  //when
  int resultCount = memberJpaRepository.bulkAgePlus(20);
  
  //then
  assertThat(resultCount).isEqualTo(3);
}
```
### 스프링 데이터 JPA를 사용한 벌크성 수정 쿼리
```java
@Modifying(clearAutomatically = true) //executeUpdate()를 실행함
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```
```java
    @Test
    public void bulkUpdate() throws Exception {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 19));
        memberRepository.save(new Member("member3", 20));
        memberRepository.save(new Member("member4", 21));
        memberRepository.save(new Member("member5", 40));

        //when
        int resultCount = memberRepository.bulkAgePlus(20);

        List<Member> result = memberRepository.findByUsername("member5");
        Member member5 = result.get(0);
        System.out.println("member5 = " + member5);

        //then
        assertThat(resultCount).isEqualTo(3);
    }
```
- 벌크성 수정, 삭제 쿼리는 `@Modifying` 어노테이션을 사용한다.
  - 사용하지 않으면 `.getResultList()`, `.getSingleResult()`를 반환해서 `org.hibernate.hql.internal.QueryExecutionRequestException: Not supported for DML operations` 가 발생한다.
- 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화 `@Modifying(clearAutomatically = true)`를 해야 한다.
  - 벌크성 쿼리는 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 엔티티 상태가 달라질 수 있다.
  - 벌크 연산 후에 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화하자.
  
## @EntityGraph
  연관된 엔티티를 한 번에 조회하는 방법(페치 조인)이다.
### 순수 JPA 페치 조인
  ```java
  @Query("select m from Member m left join fetch m.team")
  List<Member> findMemberFetchJoin();
  ```
  - 복잡한 쿼리에서 적합한 방식히다.
  - 간단한 조회문도 쿼리를 작성해야 하는 단점이 있다.
### 스프링 데이터 JPA의 EntityGraph
```java
  //공통 메서드 오버라이드
  @Override
  @EntityGraph(attributePaths = {"team"})
  List<Member> findAll();
  
  //JPQL + 엔티티 그래프
  @EntityGraph(attributePaths = {"team"})
  @Query("select m from Member m")
  List<Member> findMemberEntityGraph();
  
  //메서드 이름으로 쿼리에서 특히 편리하다.
  @EntityGraph(attributePaths = {"team"})
  List<Member> findByUsername(String username)
```
- 엔티티 그래프 기능을 편리하게 사용하게 도와준다.
- JPQL 없이 페치 조인을 사용할 수 있다. (JPQL + 엔티티 그래프도 가능)
  
*참고*  
fetch Join은 innerJoin, outerJoin 선택이 가능하지만, EntityGraph는 outerJoin만 가능하다.

## JPA Hint & Lock
- JPA 구현체에 쿼리에 대한 힌트를 주는 것이다.
- 데이터 조회 시 1차 캐시에 저장하고, 변경 감지를 위한 스냅샷 저장 등 객체 상태의 정보 유지를 위한 과정을 거친다.
- 조회가 목적인 경우 객체 상태의 정보 유지를 위한 과정은 불필요 하며, 이런 과정을 쿼리 힌트로 최적화한다.
- 성능 최적화가 필요할 때 다양한 방법과 비교해서 선택하자.
  - 조회 쿼리에 모두 추가해도 생각보다 최적화 효과가 미비할 수 있으므로.
  
*쿼리 힌트 사용*
```java
  @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value =
  "true"))
  Member findReadOnlyByUsername(String username);
```
*쿼리 힌트 사용 확인*
```java
  @Test
  public void queryHint() throws Exception {
    //given
    memberRepository.save(new Member("member1", 10));
    
    em.flush();
    em.clear();
    
    //when
    Member member = memberRepository.findReadOnlyByUsername("member1");
    member.setUsername("member2");
    em.flush(); //Update Query 실행X
  }
```
*쿼리 힌트 Page 추가*
```java
  @QueryHints(value = { @QueryHint(name = "org.hibernate.readOnly", value = "true")}, forCounting = true)
  Page<Member> findByUsername(String name, Pagable pageable);
```
forCounting : 반환 타입으로 `Page` 인터페이스를 적용하면 추가로 호출하는 페이징을 위한 count 쿼리도 쿼리 힌트 적용(기본값 true)
## Lock
```java
  @Lock(LockModeType.PESSIMISTIC_WRITE)
  List<Member> findByUsername(String name);
```
- `org.springframework.data.jpa.repository.Lock` 어노테이션을 사용
- 개념적으로 깊은 내용이라서 따로 개념 이해를 위한 공부를 추천
- JPA가 제공하는 락은 JPA 책 16.1 트랜잭션과 락 절을 참고

# 확장 기능
## 사용자 정의 리포지토리 구현
- 스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동 생성
- 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하면 구현해야 하는 기능이 너무 많음
- 여러 이유로 인터페이스의 메서드를 직접 구현하고 싶은 경우
  - JPA 직접 사용( EntityManager )
  - 스프링 JDBC Template 사용
  - MyBatis 사용
  - 데이터베이스 커넥션 직접 사용
  - Querydsl 사용

*사용자 정의 인터페이스*
```java
public interface MemberRepositoryCustom {
 List<Member> findMemberCustom();
}
```

*사용자 정의 인터페이스 구현 클래스*
```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
  
  private final EntityManager em;
  
  @Override
  public List<Member> findMemberCustom() {
    return em.createQuery("select m from Member m")
    .getResultList();
  }
}
```
*사용자 정의 인터페이스 상속*
```java
public interface MemberRepository
  extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```
*사용자 정의 메서드 호출 코드*
```java
List<Member> result = memberRepository.findMemberCustom();
```
*사용자 정의 구현 클래스*
- 규칙: 리포지토리 인터페이스 이름 + Impl
- 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록

*참고*
- 실무에서는 주로 QueryDSL이나 SpringJdbcTemplate을 함께 사용할 때 사용자 정의 리포지토리 기능 자주 사용
- 임의의 리포지토리 클래스를 만들어서 스프링 빈으로 등록하고 직접 사용해도 된다. 물론 이 경우 스프링 데이터 JPA와는 별도로 동작한다.
- CQRS 패턴을 적용해서 사용자 정의 리포지토리를 활용하는 것이 중요하다.

## 스프링 데이터 JPA 분석
### 스프링 데이터 JPA 구현체 분석
- 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체
- 패키지: `org.springframework.data.jpa.repository.support.SimpleJpaRepository`
```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> ...{
  
  @Transactional
  public <S extends T> S save(S entity) {
    if (entityInformation.isNew(entity)) {
      em.persist(entity);
      return entity;
    } else {
      return em.merge(entity);
    }
  }
 ...
}
```

- `@Repository` 적용: JPA나 JDBC의 예외를 스프링이 추상화한 예외로 변환한다.
  - 하부 기술을 바꿔도 예외처리를 변경하지 않아도 된다.
  - ex) JDBC에서 JPA로 바꿔도 Repository를 사용하는 Controller나 Service의 예외처리를 변경하지 않아도 된다.
- `@Transactional` 트랜잭션 적용
  - JPA의 모든 변경은 트랜잭션 안에서 동작
  - 스프링 데이터 JPA는 변경(등록, 수정, 삭제) 메서드를 트랜잭션 처리
  - 서비스 계층에서 트랜잭션을 시작하지 않으면 리파지토리에서 트랜잭션 시작
  - 서비스 계층에서 트랜잭션을 시작하면 리파지토리는 해당 트랜잭션을 이어 받아서 사용
  - 그래서 스프링 데이터 JPA를 사용할 때 트랜잭션이 없어도 데이터 등록, 변경이 가능함(사실은 트랜잭션이 리포지토리 계층에 걸려있는 것)
- `@Transactional(readOnly = true)`
  - 데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서 readOnly = true 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음
  - 자세한 내용은 JPA 책 15.4.2 읽기 전용 쿼리의 성능 최적화 참고

## 새로운 엔티티를 구별하는 방법
*매우 중요!!!*
- `save()` 메서드
  - 새로운 엔티티면 저장(persist)
  - 새로운 엔티티가 아니면 병합(merge)  
  
- 새로운 엔티티를 판단하는 기본 전략
  - 식별자가 객체일 때 null로 판단
  - 식별자가 자바 기본 타입일 때 0 으로 판단
  - `Persistable` 인터페이스를 구현해서 판단 로직 변경 가능
  ```java
  package org.springframework.data.domain;
  
  public interface Persistable<ID> {
    ID getId();
    boolean isNew();
  }
  ```
JPA 식별자 생성 전략이 `@GenerateValue` 면 `save()` 호출 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 정상 동작한다. 그런데 JPA 식별자 생성 전략이 `@Id` 만 사용해서 직접 할당이면 이미 식별자 값이 있는 상태로 `save()` 를 호출한다. 따라서 이 경우 `merge()` 가 호출된다. `merge()` 는 우선
DB를 호출해서 값을 확인하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율 적이다. 따라서 `Persistable` 를 사용해서 새로운 엔티티 확인 여부를 직접 구현하게는 효과적이다.

> 참고: 등록시간( @CreatedDate )을 조합해서 사용하면 이 필드로 새로운 엔티티 여부를 편리하게 확인할 수 있다. (@CreatedDate에 값이 없으면 새로운 엔티티로 판단)

*Persistable 구현*
```java
@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {
  @Id
  private String id;
  
  @CreatedDate
  private LocalDateTime createdDate;
  
  public Item(String id) {
    this.id = id;
  }
  
  @Override
  public String getId() {
    return id;
  }
  
  @Override
  public boolean isNew() {
    return createdDate == null;
  }
}
```