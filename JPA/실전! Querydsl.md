> [실전! Querydsl](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/dashboard)을 보고 정리한 글 입니다.
<br/>

# 기본 문법 

## JPQL vs Querydsl
*JPQL*
```java
    @Test
    public void startJPQL() {
        //member1 찾기
        Member findMember = em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```
- 쿼리문에 오타가 있을 경우 실행 시점에 오류가 난다.
- 파라미터 바인딩을 직접 처리해야 한다.
  
*Querydsl*
```java
    @Test
    public void startQuerydsl() {
        JPAQueryFactory queryFactory = new JPAQueryFactory(em);
        QMember m = new QMember("m");

        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1")) //파라미터 바인딩
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```
- 쿼리문에 오타가 있을 경우 컴파일 시점에서 오류가 난다.
- 파라미터 바인딩을 자동으로 처리한다.

*인터페이스 설명*
- `JPAQueryFactory`: QueryDSL을 사용하여 쿼리를 Build 할 때 사용함
- `EntityManager` 로 `JPAQueryFactory` 생성
  - `JPAQueryFactory`는 `EntityManager`를 통해 질의를 처리함
- Querydsl은 JPQL 빌더

### JPAQueryFactory를 필드로
```java
@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired
    EntityManager em;

    JPAQueryFactory queryFactory;

    @Test
    public void startQuerydsl() {
        // member1 찾기
        QMember m = new QMember("m");

        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1")) //파라미터 바인딩 처리
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```
JPAQueryFactory를 필드로 제공해도 동시성 문제는 일어나지 않는다. `JPAQueryFactory`를 생성할 때 제공하는 `EntityManager(em)`에 여러 쓰레드가 동시에 접근해도, 트랜잭션 마다 별도의 영속성 컨텍스트를 제공하기 때문이다.

## 기본 Q-Type 활용
### Q클래스 인스턴스를 사용하는 2가지 방법
```java
QMember qMember = new QMember("m"); //별칭 직접 지정
QMember qMember = QMember.member; //기본 인스턴스 사용
```

### 기본 인스턴스를 static import와 함께 사용
```java
import static study.querydsl.entity.QMember.*;

@Test
public void startQuerydsl3() {
    //member1을 찾아라.
    Member findMember = queryFactory.select(member)
    .from(member)
    .where(member.username.eq("member1"))
    .fetchOne();

    assertThat(findMember.getUsername()).isEqualTo("member1");
}
```
같은 테이블을 조인할 경우 직접 정의해서 별칭을 만들고, 일반적인 경우에는 기본 인스턴스를 사용하자

## 검색 조건 쿼리
### 기본 검색 쿼리
```java
    @Test
    public void search() {
        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1")
                    .and(member.age.eq(10)))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```
- 검색 조건은 `.and`, `.or`으로 연결할 수 있다.
- `select`, `from`을 `selectFrom` 으로 합칠 수 있다.
### JPQL이 제공하는 검색 조건 메서드 모음
```java
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'
member.username.isNotNull() //이름이 is not null
member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30
member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30
member.username.like("member%") //like 검색
member.username.contains("member") // like ‘%member%’ 검색
member.username.startsWith("member") //like ‘member%’ 검색
```
### AND 조건을 파라미터로 처리
```java
    @Test
    public void searchAndParam() {
        Member findMember = queryFactory
                .selectFrom(member)
                .where(
                        member.username.eq("member1"),
                        member.age.eq(10)
                )
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```
- `where()`에 파라미터로 검색조건을 추가하면 AND 조건이 추가된다.
- null 값을 무시한다.
- 메서드 추출을 활용해서 동적 쿼리를 깔끔하게 만들 수 있다.





