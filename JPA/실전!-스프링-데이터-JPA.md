> [실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84/dashboard) 를 보고 정리한  글입니다.

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

