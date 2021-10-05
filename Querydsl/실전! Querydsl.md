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
- 문자가 아닌 다른 타입들은 `stringValue()`로 문자로 변환할 수 있다. 
- `stringValue()`는 ENUM을 처리할 때 자주 사용한다.

# 중급 문법

## 프로젝션과 결과 반환 - 기본
select의 대상을 지정하는 것을 프로젝션이라고 한다.

### 프로젝션 대상이 하나
```java
List<String> result = queryFactory
 .select(member.username)
 .from(member)
 .fetch();
```
- 프로젝션 대상이 하나면 타입을 명확하게 지정할 수 있다.
- 프로젝션 대상이 둘 이상이면 튜플이나 DTO로 조회한다.
  
### 튜플 조회
프로젝션 대상이 둘 이상일 때 사용한다.

```java
List<Tuple> result = queryFactory
    .select(member.username, member.age)
    .from(member)
    .fetch();

for (Tuple tuple : result) {
    String username = tuple.get(member.username);
    Integer age = tuple.get(member.age);
    System.out.println("username=" + username);
    System.out.println("age=" + age);
}
```

**참고**

- `Tuple`은 Querydsl에 종속적인 타입이기 때문에 리포짓토리나 DAO 계층에서만 사용하는 것을 권장한다. 
- 하부 구현 기술(JPA, Querydsl)을 앞 단(Controller, Service)계층에서 알게 되면 하부 기술을 바꾸기 어렵다.
- Querydsl의 `Tuple`, JDBC의 `ResultSet` 같은 결과값을 앞 단에서 사용해야 할 경우에는 DTO로 변환해서 내보내자.

## 프로젝션과 결과 반환 - DTO 조회

**예제의 MemberDto 코드**

```java
@Data
@NoArgsConstructor
public class MemberDto {

    private String username;
    private int age;

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```
### 순수 JPA에서 DTO 조회 코드

```java
    @Test
    public void findDtoByJPQL() {
        List<MemberDto> result = em.createQuery("select new study.querydsl.dto.MemberDto(m.username, m.age)" +
                " from Member m", MemberDto.class)
                .getResultList();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
```
- 순수 JPA에서는 DTO를 조회할 때 new 명령어를 꼭 명시해야 한다.
- 단점
  - DTO의 패키지 이름을 다 적어줘야 해서 오타가 날 수 있고 지저분하다.
  - 생성자 방식만 지원한다.

### Querydsl 빈 생성(Bean population)
결과를 dto로 반환할 때 사용하는 방식이다. 프로퍼티 접근, 필드 직접 접근, 생성자 사용 접근 방법을 지원한다.

**프로퍼티(setter) 접근**
```java
    @Test
    public void findDtoBySetter() {
        List<MemberDto> result = queryFactory
                .select(Projections.bean(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
```
- `Projections`의 `bean()`에 dto 클래스를 적으면 된다.
- 필드에 프로퍼티로 접근하기 때문에 필드 이름이 다르면 안된다.


**필드 직접 접근**
```java
    @Test
    public void findDtoByField() {
        List<MemberDto> result = queryFactory
                .select(Projections.fields(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
```
- `Projections`의 `fields()`에 dto 클래스를 적으면 된다.
- 필드에 직접 접근하기 때문에 필드 이름이 다르면 안된다.


**필드 이름과 별칭이 다를 때**

```java
package study.querydsl.dto;
import lombok.Data;

@Data
public class UserDto {
    private String name;
    private int age;
}
```
```java
    @Test
    public void findUserDto() {
        QMember memberSub = new QMember("memberSub");

        List<MemberDto> result = queryFactory
                .select(Projections.fields(MemberDto.class,
                        member.username.as("name"),
                        ExpressionUtils.as(JPAExpressions
                        .select(memberSub.age.max())
                                .from(memberSub), "age")))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
```
- 프로퍼티 방식, 필드 접근 생성 방식에서 이름이 다를 때 `as()`를 사용한다.
- `ExpressionUtils.as(source,alias)` : 필드, 서브 쿼리에 별칭 적용
- `username.as("memberName")` : 필드에 별칭 적용


**생성자 사용**
```java
    @Test
    public void findDtoByConstructor() {
        List<MemberDto> result = queryFactory
                .select(Projections.constructor(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
```
클래스로 접근하기 때문에 필드 이름과 별칭이 달라고 상관 없다.

## 프로젝션과 결과 반환 - @QueryProjection

```java
@Data
@NoArgsConstructor
public class MemberDto {

    private String username;
    private int age;

    @QueryProjection
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```
- 생성자에 `@QueryProjection`을 사용하면 QMemberDto가 생성된다.
- 주의: `./gradlew compileQuerydsl`을 실행해야 Q파일이 생성됨


### QueryProjection 활용
```java
    @Test
    public void findDtoByQueryProjection() {
        List<MemberDto> result = queryFactory
                .select(new QMemberDto(member.username, member.age))
                .from(member)
                .fetch();
        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }

    }
```
- 컴파일러로 타입을 체크할 수 있으므로 가장 안전한 방법이다.
- DTO에 QueryDSL 어노테이션을 유지해야 하는 점과 DTO까지 Q파일을 생성해야 하는 단점이 있다.
- Querydsl에 의존적이게 된다.
- DTO를 의존성없이 깔끔하게 가져가고 싶으면 빈 생성 방식을 사용하고, 하부 기술이 바뀔 일이 없을 경우 QueryProjection 방식을 사용하자.

## 동적 쿼리 - BooleanBuilder 사용
```java
    @Test
    public void dynamicQuery_BooleanBuilder() {
        String usernameParam = "member1";
        Integer ageParam = null;

        List<Member> result = searchMember1(usernameParam, ageParam);

    }

    private List<Member> searchMember1(String usernameCond, Integer ageCond) {

        BooleanBuilder builder = new BooleanBuilder();
        if (usernameCond != null) {
            builder.and(member.username.eq(usernameCond));
        }

        if (ageCond != null) {
            builder.and(member.age.eq(ageCond));
        }

        return queryFactory
                .selectFrom(member)
                .where(builder)
                .fetch();
    }
*
    /*select
        member0_.member_id as member_i1_1_,
        member0_.age as age2_1_,
        member0_.team_id as team_id4_1_,
        member0_.username as username3_1_ 
    from
        member member0_ 
    where
        member0_.username=?
    age값이 null이여서 age 조회 쿼리 실행안됨*/
```
- BooleanBuilder 객체를 이용해서 동적 쿼리를 만드는 방식
- 조건문을 사용해서 null이 아니면 조건문 안에 쿼리가 실행된다.


## 동적 쿼리 - Where 다중 파라미터 사용
```java
@Test
public void 동적쿼리_WhereParam() throws Exception {
    String usernameParam = "member1";
    Integer ageParam = 10;
    List<Member> result = searchMember2(usernameParam, ageParam);
    Assertions.assertThat(result.size()).isEqualTo(1);
}
private List<Member> searchMember2(String usernameCond, Integer ageCond) {
    return queryFactory
    .selectFrom(member)
    .where(usernameEq(usernameCond) , ageEq(ageCond)) // null이면 무시
    .fetch();
}
private BooleanExpression usernameEq(String usernameCond) {
    return usernameCond != null ? member.username.eq(usernameCond) : null;
}
private BooleanExpression ageEq(Integer ageCond) {
    return ageCond != null ? member.age.eq(ageCond) : null;
}
```
- where 조건에 null 값은 무시되어 쿼리가 실행되지 않는다.
- 메서드를 다른 쿼리에서도 재활용 할 수 있다.
- 메서드 이름으로 사용해서 쿼리 자체의 가독성이 높아진다. 

### 조합 가능
```java
private BooleanExpression allEq(String usernameCond, Integer ageCond) {
    return usernameEq(usernameCond).and(ageEq(ageCond));
}
```
- 여러 다양한 조건들을 조합해서 메서드로 사용할 수 있다.
- 다양한 조건을 조합하기 때문에 null 체크는 주의해서 처리해야 한다.


## 수정, 삭제 벌크 연산
### 쿼리 한번으로 대량 데이터 수정
```java
    @Test
    public void bulkUpdate() {
        long count = queryFactory
                .update(member)
                .set(member.username, "비회원")
                .where(member.age.lt(28))
                .execute();

        List<Member> result = queryFactory
                .selectFrom(member)
                .fetch();

        for (Member member1 : result) {
            System.out.println("member1 = " + member1);
        }
    }
```
- JPQL 배치와 같이 영속성 컨텍스트에 있는 엔티티를 무시하고 쿼리를 실행한다.
- 조회할 때 영속성 컨텍스트 우선으로 조회해서 DB와 값이 다르게 된다.
- 벌크 연산을 실행하고 나면 `flush()`와 `clear()`를 실행하여 영속성 컨텍스트를 초기화 하자.
  
### 기존 숫자에 1 더하기
```java
long count = queryFactory
    .update(member)
    .set(member.age, member.age.add(1))
    .execute();

//쿼리 결과: update member set age = age + 1
 ```
- 곱하기: `multiply(x)`

### 쿼리 한번으로 데이터 삭제
```java
long count = queryFactory
    .delete(member)
    .where(member.age.gt(18))
    .execute();
 ```

## SQL function 호출하기
SQL function은 JPA와 같이 Dialect에 등록된 내용만 호출할 수 있다.

### member -> M으로 변경하는 replace 함수 사용
```java
String result = queryFactory
    .select(Expressions.stringTemplate("function('replace', {0}, {1}, {2})", member.username, "member", "M")) // member 문자열을 M으로 바꾸고 나머지 문자열은 0~2 인덱스까지 남겨놔라
    .from(member)
    .fetchFirst();
 ```

**소문자 변경**
```java
.select(member.username)
.from(member)
.where(member.username.eq(Expressions.stringTemplate("function('lower', {0})",
member.username)))
```
ower 같은 ansi 표준 함수들은 querydsl이 내장하고 있다. 따라서 다음과 같이 처리해도 결과는 같다.

```java
.where(member.username.eq(member.username.lower()))
```


# 실무 활용 - 순수 JPA와 Querydsl

## 순수 JPA 리포지토리와 Querydsl
**순수 JPA Repository**
```java
@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findAll_Querydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
}
```

### 순수 JPA와 Querydsl의 차이
```java
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findAll_Querydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
```
- 순수 JPA
  -  문자열로 쿼리문을 작성하기 때문에 런타임에 오류를 알 수 있다.
  -  런타임 시점에 오류가 발생하기 때문에 기능을 호출한 사용자에게 오류가 발생할 수 있다.
  - 파라미터 바인딩을 `.setParameter()`를 통해 해야한다.
- Querydsl
  - 파라미터 바인딩을 자동으로 해준다.
  - 자바 코드로 쿼리문을 작성하기 때문에 컴파일 시점에서 오류를 알 수 있다.
  
### JPAQueryFactory 스프링 빈 등록
다음과 같이 스프링 빈으로 등록해도 된다.
```java
@Bean
JPAQueryFactory jpaQueryFactory(EntityManager em) {
    return new JPAQueryFactory(em);
}
```
**참고**: 동시성 문제는 걱정하지 않아도 된다. 왜냐하면 여기서 스프링이 주입해주는 엔티티 매니저는 실제
동작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티 매니저이다. 이 가짜 엔티티 매니저는
실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.
더 자세한 내용은 자바 ORM 표준 JPA 책 13.1 트랜잭션 범위의 영속성 컨텍스트를 참고하자.  



## 동적 쿼리와 성능 최적화 조회 - Builder 사용
**MemberTeamDto - 조회 최적화용 DTO 추가**
```java
@Data
public class MemberTeamDto {

    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}
```
- `@QueryProjection`을 사용하여 `@QMemberDto`를 생성했다.
- `./gradlew compileQuerydsl`를 실행해서 `@QMemberDto`를 생성하자.
- `@QueryProjection`을 사용하면 해당 DTO가 Querydsl을 의존하게 된다. 이런 의존이 싫으면, `Projection.bean()`, `fields()`, `constructor()` 을 사용하면 된다.

**MemberSearchCondition - 회원 검색 조건 DTO**
```java
@Data
public class MemberSearchCondition {

    private String username;
    private String teamName;
    private Integer ageGoe;
    private Integer ageLoe;
}
```

### 동적 쿼리 - Builder 사용
```java
//Builder 사용
//회원명, 팀명, 나이(ageGoe, ageLoe)
public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condition) {
    
    BooleanBuilder builder = new BooleanBuilder();

    if (hasText(condition.getUsername())) {
        builder.and(member.username.eq(condition.getUsername()));
    }
    if (hasText(condition.getTeamName())) {
        builder.and(team.name.eq(condition.getTeamName()));
    }
    if (condition.getAgeGoe() != null) {
        builder.and(member.age.goe(condition.getAgeGoe()));
    }
    if (condition.getAgeLoe() != null) {
        builder.and(member.age.loe(condition.getAgeLoe()));
    }
    return queryFactory
        .select(new QMemberTeamDto(
            member.id,
            member.username,
            member.age,
            team.id,
            team.name))
        .from(member)
        .leftJoin(member.team, team)
        .where(builder)
        .fetch();
}
```
- `QMemberTeamDto`는 생성자 방식이라 `as()`를 사용하여 필드 이름을 안 맞쳐도 된다.  

**조회 예제 테스트**
```java
    @Test
    public void searchTest() {
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");

        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);
        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);

        MemberSearchCondition condition = new MemberSearchCondition();
        condition.setAgeGoe(35);
        condition.setAgeLoe(40);
        condition.setTeamName("teamB");

        List<MemberTeamDto> result = memberJpaRepository.searchByBuilder(condition);

        assertThat(result).extracting("username").containsExactly("member4");
    }
```
**주의**: 만약 동적쿼리에 해당하는 조건이 하나도 없으면 모든 데이터를 조회하게 된다. 이럴 경우 기본 조건을 두거나 리미트를 둬서 방지하도록 하자.  
  
## 동적 쿼리와 성능 최적화 조회 - Where절 파라미터 사용
**Where절에 파라미터를 사용한 예제**
```java
    public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        return hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }
```
- where절 방식은 파라미터 재사용이 가능하고, 쿼리문을 읽기 편한 장점이 있기 때문에 where절 방식을 사용하길 권장한다.
- `Predicate`보다 `BooleanBuilder`을 사용하자.
  - `BooleanBuilder`는 `and`와 `or`와 같은 메소드를 사용할 수 있다.
  - `BooleanBuilder`의 메소드를 활용해서 메서드를 재사용할 수 있다.
  - `BooleanBuilder`은 null을 반환하게 되면 where절에서 조건을


## 조회 API 컨트롤러 개발
편리한 데이터 확인을 위해 샘플 데이터를 추가하자.
샘플 데이터 추가가 테스트 케이스 실행에 영향을 주지 않으려면 프로파일을 분리해야 한다.

### 프로파일 설정

**서버용 프로파일**

`src/main/resources/application.yml`
```yml
spring:
    profiles:
        active: local
```

**테스트용 프로파일**

`src/test/resources/application.yml`
```yml
spring:
    profiles:
        active: test
```
이렇게 프로파일을 분리하면 main 소스코드와 테스트 소스 코드 실행시 서로 다른 설정값을 가지게 된다.
실무에서는 서버용, 테스트용, 운영용 프로파일로 나누는 경우가 많다.  
  
  
**샘플 데이터 추가**
```java
@Profile("local")
@Component
@RequiredArgsConstructor
public class InitMember {

    private final InitMemberService initMemberService;

    @PostConstruct //빈 생성 후 생성하기
    public void init() {
        initMemberService.init();
    }

    @Component //빈 등록
    static class InitMemberService {
        @PersistenceContext
        private EntityManager em;

        @Transactional
        public void init() {
            Team teamA = new Team("teamA");
            Team teamB = new Team("teamB");
            em.persist(teamA);
            em.persist(teamB);

            for (int i = 0; i < 100; i++) {
                Team selectedTeam = i % 2 == 0 ? teamA : teamB;
                em.persist(new Member("member"+i, i, selectedTeam));
            }
        }
    }
}
```
`@Profile`에 사용할 프로파일의 active 이름을 작성하면 원하는 프로파일을 적용할 수 있다.
  
**조회 컨트롤러**
```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberJpaRepository memberJpaRepository;

    @GetMapping("/v1/members")
    public List<MemberTeamDto> searchMemberV1(MemberSearchCondition condition) {
        return memberJpaRepository.search(condition);
    }
}
```

# 실무 활용 - 스프링 데이터 JPA와 Querydsl
## 사용자 정의 리포지토리
스프링 데이터 JPA를 사용할 경우 Querydsl를 사용한 커스텀 기능을 사용자 정의 인터페이스에 작성해야 한다.

### 사용자 정의 리포지토리 사용법

1. 사용자 정의 인터페이스 작성
```java
public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);
}
```
사용하고자 하는 커스텀 기능을 사용자 정의 인터페이스에 작성한다.

  
2. 사용자 정의 인터페이스 구현
```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        return hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }
}
```
사용자 정의 인터페이스를 상속 받아서 구현한다.

3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속
```java
public interface MemberRepository extends JpaRepository<Member, Long>,
MemberRepositoryCustom {
    List<Member> findByUsername(String username);
}
```
`JpaRepository`가 있는 곳(스프링 데이터 리포지토리)에 사용자 정의 인터페이스를 상속한다.

**참고**  
특정 API나 화면에 종속되어 있으면 별도로 조회용 리포지토리를 만들어 사용하는 것이 아키텍쳐 설계 상으로 좋다. 엔티티를 많이 검색하거나 재사용성이 좋은 경우는 사용자 정의 인터페이스에 정의해서 재사용성을 높이도록 하자.

## 스프링 데이터 페이징 활용1 - Querydsl 페이징 연동
- 스프링 데이터의 Page, Pageable을 활용하는 방법
  - 전체 카운트를 한번에 조회하는 단순한 방법
  - 데이터 내용과 전체 카운트를 별도로 조회하는 방법
  
**사용자 정의 인터페이스에 페이징 2가지 추가**
```java
public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);

    Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition,Pageable pageable);

    Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable);
}
```

### 전체 카운트를 한번에 조회하는 단순한 방법
```java
@Override
    public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
        QueryResults<MemberTeamDto> results = queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .offset(pageable.getOffset()) //페이지 시작 번호 지정
                .limit(pageable.getPageSize()) //조회할 때 가져올 데이터 수
                .fetchResults(); //Content 쿼리와 Count 쿼리도 같이 보냄

        List<MemberTeamDto> content = results.getResults(); //데이터 리스트에 담음
        long total = results.getTotal(); //가져온 데이터 개수

        return new PageImpl<>(content, pageable, total); //PageImpl: 스프링 데이터의 Page 구현체
    }
```
- Querydsl이 제공하는 `fetchResults()`를 사용하면 내용과 전체 카운트를 한번에 조회할 수 있다.(실제 쿼리는 2번 호출)
- `fetchResult()`는 카운트 쿼리 실행시 필요없는 `order by` 는 제거한다.

### 데이터 내용과 전체 카운트를 별도로 조회하는 방법
```java
    @Override
    public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable) {
        List<MemberTeamDto> content = queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        long total = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .fetchCount();

        return new PageImpl<>(content, pageable, total);
    }
```
- 전체 카운트를 조회 하는 방법을 최적화 할 수 있으면 카운트 쿼리를 분리하면 된다.
  - ex) 전체 카운트를 조회할 때 카운트 쿼리를 분리하면 조인 쿼리를 줄일 수 있어 성능 향상에 상당한 효과가 있다.
- 코드를 리팩토링해서 내용 쿼리와 전체 카운트 쿼리를 따로 메소드로 분리하여 읽기 좋게 하는 것을 권장함

## 스프링 데이터 페이징 활용2 - CountQuery 최적화
```java
        JPAQuery<Member> countQuery = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                );
        // return new PageImpl<>(content, pageable, total);
        return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchCount);
    }
```
- `PageableExecutionUtils.getPage()`로 최적화 할 수 있다.
- count 쿼리가 생략 가능한 경우에 생략해서 처리해준다.
  - 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때(첫 번째 페이지이자 마지막 페이지)
  - 마지막 페이지 일 때 
    - 컨텐츠 사이즈가 페이지 사이즈보다 작을 때 마지막 페이지가 된다. 이때 offset + 컨텐츠 사이즈를 더하면 totalCount를 구할 수 있어서 카운트 쿼리가 필요없다.
  - 위에 조건이 만족하지 않을 때에만 `countQuery::fetchCount`를 실행한다.

# 스프링 데이터 JPA가 제공하는 Querydsl 기능
여기서 소개하는 기능은 제약이 커서 복잡한 실무 환경에서 사용하기에는 많이 부족하다. 그래도 스프링 데이터에서 제공하는 기능이므로 간단히 소개하고, 왜 부족한지 설명하겠다.

## 인터페이스 지원 - QuerydslPredicateExecutor
공식 URL: https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.extensions.querydsl

**QuerydslPredicateExecutor 인터페이스**
```java
public interface QuerydslPredicateExecutor<T> {
    Optional<T> findById(Predicate predicate);
    Iterable<T> findAll(Predicate predicate);
    long count(Predicate predicate);
    boolean exists(Predicate predicate);
    // … more functionality omitted.
}
```

**리포지토리에 적용**
```java
interface MemberRepository extends JpaRepository<User, Long>,
QuerydslPredicateExecutor<User> {
}
```
```java
Iterable result = memberRepository.findAll(
    member.age.between(10, 40).and(member.username.eq("member1"))
);
```
### 한계점
- 조인X (묵시적 조인은 가능하지만 left join이 불가능하다.)
- 클라이언트가 Querydsl에 의존해야 한다. 
  - 서비스 클래스가 Querydsl이라는 구현 기술에 의존해야 한다.
- 복잡한 실무환경에서 사용하기에는 한계가 명확하다.

## Querydsl Web 지원
공식 URL: https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.web.type-safe  

### 한계점
- 단순한 조건만 가능
- 조건을 커스텀하는 기능이 복잡하고 명시적이지 않음
- 컨트롤러가 Querydsl에 의존
- 복잡한 실무환경에서 사용하기에는 한계가 명확

## 리포지토리 지원 - QuerydslRepositorySupport
### 장점
- `getQuerydsl().applyPagination()` 스프링 데이터가 제공하는 페이징을 Querydsl로 편리하게 변환 가능(Sort는 오류발생)
- `from()` 으로 시작 가능(최근에는 QueryFactory를 사용해서 `select()`로 시작하는 것이 더 명시적)
- `EntityManager` 제공

### 한계
- Querydsl 3.x 버전을 대상으로 만듬
- Querydsl 4.x에 나온 JPAQueryFactory로 시작할 수 없음
- select로 시작할 수 없음 (from으로 시작해야함)
- QueryFactory 를 제공하지 않음
- 스프링 데이터 Sort 기능이 정상 동작하지 않음
