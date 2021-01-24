# 김영한님의 실전! Querydsl
> 아래 내용은 [실전! Querydsl](https://www.inflearn.com/course/Querydsl-실전# "실전! Querydsl") 강좌를 정리한 내용 입니다.

## 1. 프로젝트 환경설정

* 프로젝트 생성

    * `스프링 부트 스타터`에서 프로젝트를 생성한다.
    
    * 의존성 추가 : `Spring Web`, `JPA`, `H2`, `LOMBOK`
    
        ![image 1](images/img1.png)
        
* QueryDsl 설정 및 검증

    * `Gradle`의 경우, `build.gradle`에 주석을 참고해서 QueryDsl 설정을 추가한다.
    
        ```
        plugins {
        	...
      
        	//querydsl용 플러그인 추가
        	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
      
        	...
        }
        
        ...
        
        dependencies {
        	...
      
        	//querydsl 라이브러리 추가
        	implementation 'com.querydsl:querydsl-jpa'
      
        	...
        }
        
        ...
      
        // querydsl 빌드 관련 설정 START
        def querydslDir = "$buildDir/generated/querydsl"
        
        querydsl {
        	jpa = true
        	querydslSourcesDir = querydslDir
        }
        sourceSets {
        	main.java.srcDir querydslDir
        }
        configurations {
        	querydsl.extendsFrom compileClasspath
        }
        compileQuerydsl {
        	options.annotationProcessorPath = configurations.querydsl
        }
        // querydsl 빌드 관련 설정 END
        ```
      
    * Querydsl 환경설정 검증
    
        * 검증용 엔티티 생성
        
            ```java
            @Entity
            @Getter @Setter
            public class Hello {
          
                @Id @GeneratedValue
                private Long id;
          
            }
            ```
          
        * 검증용 Q 타입 생성
        
            * 인텔리제이(IntelliJ)에서 Gradle인 경우, 사용법
        
                * `Gradle` → `Tasks` → `build` → `clean`
                
                * `Gradle` → `Tasks` → `other` → `compileQuerydsl`
                
            * 콘솔에서 Gradle인 경우, 사용법
        
                * `./gradlew clean compileQuerydsl`
                
        * Q 타입 생성 확인
        
            * `build` → `generated` → `querydsl`
            
            * `study.querydsl.entity.QHello.java` 파일이 생성되어 있어야 함
                
        * [참고] Q 타입은 컴파일 시점에 자동 생성되므로 버전 관리를 하지 않는 것이 좋다. 
        
            * 앞서 설정에서 생성 위치를 `gradle build` 폴더 아래 생성 되도록 했기 때문에 이 부분도 자연스럽게 해결된다. 
            
            * 대부분 `gradle build` 폴더를 git에 포함하지 않는다.

        * 테스트 케이스로 실행 검증
        
            ```java
            @SpringBootTest
            @Transactional
            //@Commit
            class QuerydslApplicationTests {
            
                @Autowired EntityManager em;
            
                @Test
                void contextLoads() {
                    // Hello 엔티티 저장
                    Hello hello = new Hello();
                    em.persist(hello);
            
                    // JPAQueryFactory 생성
                    JPAQueryFactory query = new JPAQueryFactory(em);
                    QHello qHello = QHello.hello;
                    
                    Hello result = query
                            .selectFrom(qHello)
                            .fetchOne();
            
                    assertThat(result).isEqualTo(hello);
                    assertThat(result.getId()).isEqualTo(hello.getId()); // lombok 동작 확인 (hello.getId())
            
                }
            
            }
            ```

* H2 데이터베이스 설치

    * Mac을 사용하는 경우, `chmod 755 h2.sh`로 권한을 부여해야 된다.
    
* 스프링 부트 설정 - JPA, DB

    * `org.hibernate.SQL` 옵션은 `로거(logger)`를 통해 하이버네이트 실행 SQL을 남긴다.
    
        ```
        spring:
          datasource:
            url: jdbc:h2:tcp://localhost/~/querydsl
            username: sa
            password:
            driver-class-name: org.h2.Driver
        
          jpa:
            hibernate:
              ddl-auto: create
            properties:
              hibernate:
                #show_sql: true
                format_sql: true
        
        logging.level:
          org.hibernate.SQL: debug
        #  org.hibernate.type: trace
        ```

    * 쿼리 파라미터 로그 남기기
    
        * `implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.8'`

## 2. 예제 도메인 모델

* `스프링 데이터 JPA` 강좌와 동일한 예제 도메인 모델을 사용하므로 생략한다.

## 3. 기본 문법

#### 1) JPQL vs Querydsl

* JPQL vs Querydsl

    * `Querydsl`은 JPQL 빌더이다.
    
    * JPQL은 문자이므로 실행 시점에 오류를 발견 할 수 있다. 하지만 Querydsl은 코드이므로 컴파일 시점에 오류를 발견 할 수 있다. 
    
    * 그리고 JPQL은 파라미터 바인딩을 직접 처리해야 되지만 Querydsl은 파라미터 바인딩을 자동으로 처리해준다.

* 실습

    * 테스트 기본 코드를 작성한다.
    
        ```java
        @SpringBootTest
        @Transactional
        public class QuerydslBasicTest {
        
            @Autowired
            EntityManager em;
        
            @BeforeEach
            public void before(){
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
            }
        
        }
        ```
    
    * JPQL
    
        ```java
        @Test
        public void startJPQL(){
            // member1을 찾아라.
            String qlString =
                    "select m from Member m " +
                    "where m.username = :username";
        
            Member findMember = em.createQuery(qlString, Member.class)
                    .setParameter("username", "member1")
                    .getSingleResult();
        
            assertThat(findMember.getUsername()).isEqualTo("member1");
        }
        ```
      
    * Querydsl
    
        ```java
        @Test
        public void startQuerydsl(){
            // EntityManager로 JPAQueryFactory를 생성한다.
            JPAQueryFactory queryFactory = new JPAQueryFactory(em);
        
            // 쿼리 타입(Q)을 생성한다. 생성자에는 별칭을 지정한다.
            QMember m = new QMember("m");
        
            Member findMember = queryFactory
                    .select(m)
                    .from(m)
                    .where(m.username.eq("member1"))
                    .fetchOne();
        
            assertThat(findMember.getUsername()).isEqualTo("member1");
        }
        ```

#### 2) 기본 Q-Type 활용

* Q 클래스 인스턴스를 사용하는 2가지 방법

    * Q 클래스 인스턴스를 직접 생성해서 사용하기

        ```java
        QMember qMember = new QMember("m"); //별칭 직접 지정 
        ```

    * 이미 생성된 Q 클래스의 기본 인스턴스를 사용하기

        ```java
        QMember qMember = QMember.member; //기본 인스턴스 사용
        ```
* Tip

    * Q 클래스의 기본 인스턴스를 `static import`와 함께 사용 할 수 있다.
    
        * `QMember.member`라고 작성한 다음, 퀵 픽스 기능을 사용하면 `member`로 작성 할 수 있다.
      
            ```java
            @Test
            public void startQuerydsl(){
                Member findMember = queryFactory
                        .select(member)
                        .from(member)
                        .where(member.username.eq("member1"))
                        .fetchOne();
            
                assertThat(findMember.getUsername()).isEqualTo("member1");
            }
            ```
    
    * `Querydsl`로 작성한 코드가 실행하는 JPQL를 보고 싶다면 다음 설정을 추가하자.
       
        * `spring.jpa.properties.hibernate.use_sql_comments: true`
        
    * 같은 테이블을 조인해야 하는 경우가 있다면 다음과 같이 직접 생성하고 그 외의 경우에는 기본 인스턴스를 사용하자.
    
        * `QMember m1 = new QMember("m1");`
        
#### 3) 검색 조건 쿼리

* JPQL이 제공하는 모든 검색 조건 제공

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

* AND 조건 처리

    * `and()`를 사용하기
            
            ```java
            @Test
            public void search(){
                Member findMember = queryFactory
                        .selectFrom(member)
                        .where(member.username.eq("member1")
                                .and(member.age.eq(10)))
                        .fetchOne();
            
                assertThat(findMember.getUsername()).isEqualTo("member1");
            }
            ```

    * AND 조건을 파라미터로 처리하기
        
        * `where()`에 파라미터로 검색 조건을 `,`로 구분하여 전달하면 AND 조건이 추가된다.
    
        * 그리고 이 방식은 null 값을 무시하기 때문에 동적 쿼리를 만들 때, 편리하다.
    
            ```java
            @Test
            public void searchAndParam(){
                Member findMember = queryFactory
                        .selectFrom(member)
                        .where(member.username.eq("member1"),
                                (member.age.eq(10)))
                        .fetchOne();
            
                assertThat(findMember.getUsername()).isEqualTo("member1");
            }
            ```

#### 4) 결과 조회

* 문법

    * `fetch()` : 리스트로 조회한다. 데이터가 없으면 비어있는 리스트를 반환한다. 
    
    * `fetchOne()` : 단 건을 조회할 때, 사용한다.
    
        * 결과가 없으면 `null`을 반환하고
        
        * 결과가 둘 이상이면 `com.querydsl.core.NonUniqueResultException` 예외가 발생한다.
        
    * `fetchFirst()` : limit을 붙여 fetchOne()을 실행한다.
    
        * `limit(1).fetchOne()`
        
    * `fetchResults()` : 페이징 정보를 포함하는 결과를 반환한다. total count 쿼리를 추가 실행한다. 
    
    * `fetchCount()` : count 쿼리로 변경해서 count를 조회한다.

* 예시

    ```java
    @Test
    public void resultFetch(){
        //List
        List<Member> fetch = queryFactory
                .selectFrom(member)
                .fetch();
    
        //단 건 - 결과가 둘 이상이므로 예외가 발생함. 주석 처리하고 실행하자.
        //Member findMember1 = queryFactory
        //        .selectFrom(member)
        //        .fetchOne();
    
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
    
    }
    ```
  
#### 5) 정렬

* 정렬은 `orderBy()`를 사용하며 파라미터로 정렬 조건을 `,`로 구분하여 전달하면 된다.

    ```java
    /*
    * 회원 정렬 순서
    * (1) 회원 나이 내림차순(DESC)
    * (2) 회원 이름 오름차순(ASC)
    *     단, (2)에서 회원 이름이 없으면 마지막에 출력 (nulls last)
    * */
  
    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.eq(100))
            .orderBy(member.age.desc(), member.username.asc().nullsLast())
            .fetch();
    ```

    * `asc()` : 오름차순 정렬
      
    * `desc()` : 내림차순 정렬 
    
    * `nullsFirst()` : null 데이터가 먼저 나오도록 한다.
    
    * `nullsLast()` : null 데이터가 마지막에 나오도록 한다.

#### 6) 페이징

* 페이징은 `offset()`과 `limit()`을 사용한다.

    * `offset()`은 시작 페이지를 지정하며 페이지는 0 부터 시작한다. 
    
    * `limit()`은 조회 할 데이터 건수를 지정한다.
    
* 예시

    * 기본적인 사용 방법

        ```java
        @Test
        public void paging1(){
            List<Member> result = queryFactory.selectFrom(member)
                    .orderBy(member.username.desc())
                    .offset(1)
                    .limit(2)
                    .fetch();
        
            assertThat(result.size()).isEqualTo(2);
        }
        ```
      
    * 전체 데이터 수가 필요하면 다음과 같이 작성한다.

        ```java
        @Test
        public void paging2(){
            // count 쿼리를 실행한 다음, 컨텐츠를 조회하기 위한 쿼리가 실행된다.
            QueryResults<Member> queryResults = queryFactory.selectFrom(member)
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
      
        * count 쿼리가 실행되니 성능 상 주의가 필요하다.
        
        * count 쿼리에 조인이 필요 없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야 한다.

#### 7) 집합

* 
  



        
