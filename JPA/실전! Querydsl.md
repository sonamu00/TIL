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
  

## 결과 조회
- `fetch()` : 리스트 조회, 데이터 없으면 빈 리스트 반환
- `fetchOne()` : 단 건 조회
  - 결과가 없으면 : null
  - 결과가 둘 이상이면 : `com.querydsl.core.NonUniqueResultException`
- `fetchFirst()` : 첫번째 페이지 값 조회, `limit(1).fetchOne()`과 같음
- `fetchResults()` : 페이징 정보 가져옴, total count 쿼리 추가 실행
  - 복잡한 페이징에선 성능 최적화 때문에 사용하지 않음
- `fetchCount()` : count 쿼리로 변경해서 count 수 조회

```java
//List
List<Member> fetch = queryFactory
 .selectFrom(member)
 .fetch();

//단 건
Member findMember1 = queryFactory
 .selectFrom(member)
 .fetchOne();

//처음 한 건 조회
Member findMember2 = queryFactory
 .selectFrom(member)
 .fetchFirst();

//페이징에서 사용
QueryResults<Member> results = queryFactory
 .selectFrom(member)
 .fetchResults();

//count 쿼리로 변경
long count = queryFactory
 .selectFrom(member)
 .fetchCount();
```


## 정렬
```java
    @Test
    public void sort() {
        /**
         * 회원 정렬 순서
         * 1. 회원 나이 내림차순(desc)
         * 2. 회원 이름 올림차순(asc)
         * 단 2에서 회원 이름이 없으면 마지막에 출력(nulls last)
         */
        em.persist(new Member(null, 100));
        em.persist(new Member("member5", 100));
        em.persist(new Member("member6", 100));

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(100))
                .orderBy(member.age.desc(),
                member.username.asc().nullsLast())
                .fetch();

        Member member5 = result.get(0);
        Member member6 = result.get(1);
        Member memberNull = result.get(2);
        assertThat(member5.getUsername()).isEqualTo("member5");
        assertThat(member6.getUsername()).isEqualTo("member6");
        assertThat(memberNull.getUsername()).isNull();
    }
```
- `desc()`, `asc()`: 일반 정렬
- `nullsLast()`, `nullFirst()`: null 데이터 순서 부여

## 페이징
### 조회 건수 제한
```java
    @Test
    public void paging1() {
        List<Member> result = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc())
                .offset(1)
                .limit(2)
                .fetch();

        assertThat(result.size()).isEqualTo(2);
    }
```

### 전체 조회 수
```java
    @Test
    public void paging2() {
        QueryResults<Member> queryResults = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc())
                .offset(1)
                .limit(2)
                .fetchResults();

        assertThat(queryResults.getTotal()).isEqualTo(4);
        assertThat(queryResults.getLimit()).isEqualTo(2);
        assertThat(queryResults.getOffset()).isEqualTo(1);
        assertThat(queryResults.getResults().size()).isEqualTo(2);
    }
```
- `QueryResults`는 count 쿼리가 실행되니 성능상 주의를 요함

> 참고: 실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 하지만, count 쿼리는 조인이 필요 없는 경우도 있다. 그런데 이렇게 자동화된 count 쿼리는 원본 쿼리와 같이 모두 조인을 해버리기 때문에 성능이 안나올 수 있다. count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야 한다.

## 집합
### 집합 함수
```java
    /**
     * JPQL
     * select
     * COUNT(m), //회원수
     * SUM(m.age), //나이 합
     * AVG(m.age), //평균 나이
     * MAX(m.age), //최대 나이
     * MIN(m.age) //최소 나이
     * from Member m
     */
    @Test
    public void aggregation() {
        List<Tuple> result = queryFactory
                .select(
                        member.count(),
                        member.age.sum(),
                        member.age.avg(),
                        member.age.max(),
                        member.age.min()
                )
                .from(member)
                .fetch();

        Tuple tuple = result.get(0);
        assertThat(tuple.get(member.count())).isEqualTo(4);
        assertThat(tuple.get(member.age.sum())).isEqualTo(100);
        assertThat(tuple.get(member.age.avg())).isEqualTo(25);
        assertThat(tuple.get(member.age.max())).isEqualTo(40);
        assertThat(tuple.get(member.age.min())).isEqualTo(10);
    }
```
- JPQL이 제공하는 모든 집합 함수를 제공한다.
- Tuple은 프로젝션과 결과반환에서 설명한다.


### GroupBy 사용
```java
    /**
     * 팀의 이름과 각 팀의 평균 연령을 구해라.
     */
    @Test
    public void group() throws Exception {
        List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team)
                .groupBy(team.name)
                .fetch();
        Tuple teamA = result.get(0);
        Tuple teamB = result.get(1);

        assertThat(teamA.get(team.name)).isEqualTo("teamA");
        assertThat(teamA.get(member.age.avg())).isEqualTo(15);

        assertThat(teamB.get(team.name)).isEqualTo("teamB");
        assertThat(teamB.get(member.age.avg())).isEqualTo(35);
    }
```
*groupBy(), having() 예시*
```java
…
.groupBy(item.price)
.having(item.price.gt(1000))
…
```

## 조인 - 기본조인
### 기본 조인
조인의 기본 문법은 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭(alias)으로 사용할 Q 타입을 지정하면 된다.
```java
join(조인 대상, 별칭으로 사용할 Q타입)
```

*기본 조인*
```java
    /**
     * 팀 A에 소속된 모든 회원
     */
    @Test
    public void join() {
        List<Member> result = queryFactory
                .selectFrom(member)
                .join(member.team, team)
                .where(team.name.eq("teamA"))
                .fetch();

        assertThat(result)
                .extracting("username")
                .containsExactly("member1", "member2");
    }
```
- `join()`, `innerJoin()`: 내부 조인(inner join)
- `leftJoin()`: left 외부 조인(left outer join)
- `rightJoin()`: right 외부 조인(right outer join)
- JPQL의 `on`과 성능 최적화를 위한 `fetch` 조인 제공

### 세타 조인
연관관계가 없는 필드로 조인

```java
    /**
     * 세타 조인(연관관계가 없는 필드로 조인)
     * 회원의 이름이 팀 이름과 같은 회원 조회
     */
    @Test
    public void theta_join() {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));
        em.persist(new Member("teamC"));

        List<Member> result = queryFactory
                .select(member)
                .from(member, team)
                .where(member.username.eq(team.name))
                .fetch();

        assertThat(result)
                .extracting("username")
                .containsExactly("teamA", "teamB");
    }
```
- from절에 여러 엔티티를 선택해서 cross join
- 외부 조인이 불가능하지만 조인 on을 사용하면 외부 조인 가능

## 조인 - on절
- On절을 활용한 조인(JPA 2.1부터 지원)
    1. 조인 대상 필터링
    2. 연관관계 없는 엔티티 외부 조인

### 조인 대상 필터링
```java
    /**
     * 예) 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회
     * JPQL: SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'teamA'
     * SQL: SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and
     t.name='teamA'
     */
    @Test
    public void Join_on_filtering() {
        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(member.team, team).on(team.name.eq("teamA"))
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
```
*결과*
```java
t=[Member(id=3, username=member1, age=10), Team(id=1, name=teamA)]
t=[Member(id=4, username=member2, age=20), Team(id=1, name=teamA)]
t=[Member(id=5, username=member3, age=30), null]
t=[Member(id=6, username=member4, age=40), null]
```
> 참고: on 절을 활용해 조인 대상을 필터링 할 때, 외부조인이 아니라 내부조인(inner join)을 사용하면,
where 절에서 필터링 하는 것과 기능이 동일하다. 따라서 on 절을 활용한 조인 대상 필터링을 사용할 때,
내부조인 이면 익숙한 where 절로 해결하고, 정말 외부조인이 필요한 경우에만 이 기능을 사용하자.

### 연관관계 없는 엔티티 외부 조인
```java
    /**
     * 2. 연관관계 없는 엔티티 외부 조인
     * 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
     * JPQL: SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name
     * SQL: SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.username = t.name
     */
    @Test
    public void Join_on_no_filtering() {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));
        em.persist(new Member("teamC"));

        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(team).on(member.username.eq(team.name))
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }

```
- 하이버네이트 5.1부터 `on`을 사용해서 서로 관계가 없는 필드로 외부 조인하는 기능이 추가됨, 내부 조인도 가능함
- 주의: `leftJoin()`에 일반 조인과 다르게 엔티티 하나만 들어간다.
  - 일반조인: `leftJoin(member.team, team)`
  - on조인: `from(member).leftJoin(team).on(xxx)`


*결과*
```java
t=[Member(id=3, username=member1, age=10), null]
t=[Member(id=4, username=member2, age=20), null]
t=[Member(id=5, username=member3, age=30), null]
t=[Member(id=6, username=member4, age=40), null]
t=[Member(id=7, username=teamA, age=0), Team(id=1, name=teamA)]
t=[Member(id=8, username=teamB, age=0), Team(id=2, name=teamB)]
```

## 조인 - 페치 조인
페치 조인은 SQL 조인을 활용해서 연관된 엔티티를 SQL 한번에 조회하는 기능이다. 자세한 내용은 JPA 기본편 페치 조인 참고.

### 페치 조인 미적용
```java
    @PersistenceUnit
    EntityManagerFactory emf;

    @Test
    public void fetchJoinNo() {
        em.flush();
        em.clear();

        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"))
                .fetchOne();

        boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        assertThat(loaded).as("패치 조인 미적용").isFalse();
    }
```
지연로딩으로 Member, Team SQL 쿼리 각각 실행한다.

### 페치 조인 적용
```java
    @Test
    public void fetchJoinUse() {
        em.flush();
        em.clear();

        Member findMember = queryFactory
                .selectFrom(member)
                .join(member.team, team).fetchJoin()
                .where(member.username.eq("member1"))
                .fetchOne();

        boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        assertThat(loaded).as("패치 조인 적용").isTrue();
    }
```
- 즉시로딩으로 Member, Team SQL 쿼리 조인으로 한번에 조회한다.
- `join()` 뒤에 `fetchJoin()` 이라고 추가해서 사용하면 된다.

## 서브쿼리
`com.querydsl.jpa.JPAExpressions`를 사용한다.

### 서브쿼리 eq 사용
```java
    /**
     * 나이가 가장 많은 회원 조회
     */
    @Test
    public void subQuery() {

        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(
                        JPAExpressions.select(memberSub.age.max())
                        .from(memberSub)
                )).fetch();

        assertThat(result).extracting("age")
                .containsExactly(40);

    }
```

### 서브쿼리 goe 사용
```java
    /**
     * 나이가 평균 나이 이상인 회원
     */
    @Test
    public void subQueryGoe() {

        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.goe(
                        JPAExpressions.select(memberSub.age.avg())
                                .from(memberSub)
                )).fetch();

        assertThat(result).extracting("age")
                .containsExactly(30, 40);

    }
```

### 서브쿼리 여러 건 처리 in 사용
```java
/**
 * 서브쿼리 여러 건 처리, in 사용
 */
@Test
public void subQueryIn() throws Exception {
    
    QMember memberSub = new QMember("memberSub");
    
    List<Member> result = queryFactory
        .selectFrom(member)
        .where(member.age.in(
            JPAExpressions
                .select(memberSub.age)
                .from(memberSub)
                .where(memberSub.age.gt(10))
                )).fetch();
    
    assertThat(result).extracting("age")
            .containsExactly(20, 30, 40);
}
```

### select 절에 subquery
```java
    @Test
    public void selectSubQuery() {

        QMember memberSub = new QMember("memberSub");

        List<Tuple> result = queryFactory
                .select(member.username,
                        .select(memberSub.age.avg())
                        .from(memberSub))
                .from(member)
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
```

static import 사용 가능: `import static com.querydsl.jpa.JPAExpressions.select`


### from절에 서브쿼리 사용
JPA JPQL 서브쿼리의 한계점으로 from 절의 서브쿼리(인라인 뷰)는 지원하지 않는다. 하이버네이트 구현체를 사용하면 select절의 서브쿼리는 지원한다. Querydsl도 하이버네이트 구현체를 사용해서 select절의 서브쿼리를 지원한다.

### from 절의 서브쿼리 해결방안
1. 서브쿼리를 join으로 변경한다. (가능한 상황도 있고, 불가능한 상황도 있다.)
2. 애플리케이션에서 쿼리를 2번 분리해서 실행한다.
3. nativeSQL을 사용한다.

*참고*

- 쿼리로 로직들을 해결할 때 from절 서브쿼리가 발생할 경우가 많다.
  - ex) 뷰에 보여줄 데이터(날짜 포맷)를 애플리케이션 내에서 해결하지 않고 쿼리문을 만들어 데이터를 꺼내온다.
- 데이터베이스는 데이터를 필터링하여 가져오는 역할에만 집중하고, 로직은 애플리케이션 레벨에서 해결하자.

## Case문
select, 조건절(where), order by에서 사용 가능하다.

### 단순한 조건
```java
    @Test
    public void basicCase() {
        List<String> result = queryFactory
                .select(member.age
                        .when(10).then("열살")
                        .when(20).then("스무살")
                        .otherwise("기타"))
                .from(member)
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }
```

### 복잡한 조건
```java
    @Test
    public void complexCase() {
        List<String> result = queryFactory
                .select(new CaseBuilder()
                    .when(member.age.between(0, 20)).then("0~20")
                    .when(member.age.between(21, 30)).then("21~30")
                    .otherwise("기타"))
                .from(member)
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }
```

### orderBy에서 Case문 사용하여 정렬하기
예제로 다음과 같은 임의의 순서로 회원을 출력한다.
1. 0 ~ 30살이 아닌 회원을 가장 먼저 출력
2. 0 ~ 20살 회원 출력
3. 21 ~ 30살 회원 출력

```java
NumberExpression<Integer> rankPath = new CaseBuilder()
    .when(member.age.between(0, 20)).then(2)
    .when(member.age.between(21, 30)).then(1)
    .otherwise(3);
List<Tuple> result = queryFactory
    .select(member.username, member.age, rankPath)
    .from(member)
    .orderBy(rankPath.desc())
    .fetch();
```
Querydsl은 자바 코드로 작성하기 때문에 rankPath처럼 복잡한 조건을 변수로 선언해서 select 절, orderBy 절에서 함께 사용할 수 있다.

*참고*
데이터베이스는 최소한의 그루핑과 필터링을 통해서 데이터를 줄이는 일을 하는 것 좋다. Case 예제처럼 데이터를 전환해서 보여주는 것은 애플리케이션 로직에서 하는 것을 권장한다.

## 상수, 문자 더하기

### 상수 더하기
```java
    @Test
    public void constant() {
        List<Tuple> result = queryFactory
                .select(member.username, Expressions.constant("A"))
                .from(member)
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
    /** 결과
    tuple = [member1, A]
    tuple = [member2, A]
    tuple = [member3, A]
    tuple = [member4, A]
    **/
```
상수가 필요할 경우 `Expressions.constant()`을 사용한다.

### 문자 더하기
```java
    @Test
    public void concat() {

        //{username}_{age}
        List<String> result = queryFactory
                .select(member.username.concat("_").concat(member.age.stringValue()))
                .from(member)
                .where(member.username.eq("member1"))
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }
    // 결과: member1_10
```
문자가 아닌 다른 타입들은 `stringValue()`로 문자로 변환할 수 있다. `stringValue()`는 ENUM을 처리할 때 자주 사용한다.