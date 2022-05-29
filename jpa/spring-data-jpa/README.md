# 김영한님의 실전! 스프링 데이터 JPA
> 아래 내용은 [실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84 "실전! 스프링 데이터 JPA") 강좌를 정리한 내용 입니다.

## 1. 프로젝트 환경설정

* `JUnit Vintage` : JUnit 4로 작성한 테스트 코드를 실행할 때, 사용하는 모듈이다.

* H2 데이터베이스 설치

    * Mac을 사용하는 경우, `chmod 755 h2.sh`로 권한을 부여해야 된다.
    
* 스프링 데이터 JPA 동작 확인

    * `@PersistenceContext`: EntityManager를 주입 받을 때 사용한다.
    
    * JPA 기본편 내용 복습
    
        * JPA의 모든 데이터 변경은 트랜잭션 안에서 이루어져야 한다.
    
        * JPA는 같은 트랜잭션 안에서 영속성 컨텍스트의 동일성을 보장한다.
    
            * `member == findMember`도 true가 되어야 한다는 의미다.
    
    * `@Transactional`
    
        * 트랜잭션을 처리 할 때 사용한다.
        
        * 테스트 코드에서 `@Transactional`를 사용하면 기본적으로 테스트가 종료된 다음, 바로 롤백을 한다.
        
        * 그리고 `@Rollback(false)`를 추가하면 롤백을 하지 않고 커밋을 한다.
        
            * 스프링 데이터 JPA를 공부할 때, 사용하면 편리하다.
    
* 자주 사용되는 설정

    * 데이터 소스 설정하기

        * `spring.datasource.url` : 데이터베이스의 JDBC URL를 의미한다.

        * `spring.datasource.username` : 데이터베이스에 접속할 때 사용하는 username를 의미한다.
    
        * `spring.datasource.password` : 데이터베이스에 접속할 때 사용하는 password를 의미한다.
    
        * `spring.datasource.driver-class-name` : 패키지 명을 포함한 JDBC 드라이버 이름을 의미한다.
    
    * `spring.jpa.hibernate.ddl-auto` : DDL 자동 생성 기능을 지정한다.
    
        ```
        spring.jpa.hibernate.ddl-auto = create
        ```
        
    * `spring.jpa.properties.hibernate.show_sql` : 하이버네이트가 실행한 SQL을 System.out에 출력한다.

        ```
        spring.jpa.properties.hibernate.show_sql=true
        ```
    
    * `logging.level.org.hibernate.SQL` : 하이버네이트가 실행한 SQL을 로거를 통해 출력한다.

        ```
        logging.level.org.hibernate.SQL=debug
        ```

        * 모든 로그 출력은 로거를 통해 남겨야 한다.

    * `spring.jpa.properties.hibernate.format_sql` : 하이버네이트가 실행한 SQL을 가독성 있게 표현한다.
    
        ```
        spring.jpa.properties.hibernate.format_sql=true
        ```
    
    * 바인드 파라미터(Bind Parameter)를 로그로 출력하기
    
        * ① `logging.level.org.hibernate.type.descriptor.sql` : 바인드 파라미터를 로그로 출력한다.
    
            ```
            logging.level.org.hibernate.type.descriptor.sql=trace
            ```

            * 하이버네이트가 실행한 SQL에서 물음표(`?`)로 표기된 부분을 바인드 파라미터라고 한다.
    
            * 바인드 파라미터에 어떤 값이 전달되는지 확인할 때 사용하는 옵션이다.

        * ② 외부 라이브러리 사용하기 (`p6spy`)
    
            * 스프링 부트를 사용하면 이 라이브러리만 추가하면 된다.
            
                ```
                implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'
                ```

                * 바인드 파라미터를 로그로 남기는 외부 라이브러리는 시스템 자원을 사용하므로, 개발 단계에서는 편하게 사용해도 된다. 
                  
                * 하지만 운영 시스템에 적용하려면 꼭 성능 테스트를 하고 사용하는 것이 좋다.

            * [참고] https://github.com/gavlyukovskiy/spring-boot-data-source-decorator

    * `spring.jpa.properties.hibernate.use_sql_comments` : SQL문 이외에 추가적인 정보를 출력한다.
    
        ```
        spring.jpa.properties.hibernate.use_sql_comments=true
        ```

    * `spring.jpa.properties.hibernate.dialect` : 데이터베이스 방언을 지정한다.

        ```
        spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
        ```

## 2. 예제 도메인 모델

* 실무에서는 가급적 `@Setter`를 사용하지 않고 비즈니스 메소드를 따로 만드는 것이 좋다.

* JPA는 `기본 생성자`가 필요하다. 

    * 다음과 같은 코드를 작성 하였다.
    
        ```java
        protected Member() {
        }
        ```
      
    * 위의 코드는 아래 코드로 대체 할 수 있다.
    
        * `@NoArgsConstructor(access = AccessLevel.PROTECTED)`

* `@ToString`는 연관관계가 없는 필드만 지정한다. (무한루프 방지 하기 위함)

    * `@ToString(of = {"id", "username", "age"})` (O)

    * `@ToString(of = {"id", "username", "age", "team"})` (X)

* `양방향 연관관계`는 연관관계 편의 메소드(`changeTeam()`)를 만들어 한번에 처리하는 것이 좋다. 

* JPA의 모든 연관 관계는 지연 로딩(LAZY)으로 설정해야 한다. (EAGER는 성능 최적화 하기가 어렵다.)

    ```java
    @Entity
    @Getter @Setter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    @ToString(of = {"id", "username", "age"})
    public class Member {
    
        @Id @GeneratedValue
        @Column(name = "member_id")
        private Long id;
    
        private String username;
    
        private int age;
    
        @ManyToOne(fetch = LAZY)
        @JoinColumn(name = "team_id")
        private Team team;
    
        public Member(String username) {
            this.username = username;
        }
    
        public void changeTeam(Team team) {
            this.team = team;
            team.getMembers().add(this);
        }
    
    }
    ```

    ```java
    @Entity
    @Getter @Setter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    @ToString(of = {"id", "name"})
    public class Team {
    
        @Id @GeneratedValue
        @Column(name = "team_id")
        private Long id;
    
        private String name;
    
        @OneToMany(mappedBy = "team")
        private List<Member> members = new ArrayList<>();
    
        public Team(String name) {
            this.name = name;
        }
    
    }
    ```

## 3. 공통 인터페이스 기능

#### 1) 공통 인터페이스 설정

* 스프링 설정에 자바 설정 파일을 사용한다면 `@EnableJpaRepositories` 애노테이션을 추가하고 `basePackages`에 리포지토리를 검색할 패키지 위치를 지정한다. 

     ```java
    @Configuration
    @EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
    public class AppConfig {}
     ```

    * **스프링 부트를 사용하면 해당 애노테이션(`@EnableJpaRepositories`)을 생략할 수 있다.** 
    
    * 그 이유는 `@SpringBootApplication`이 위치한 패키지 이하 부터는 자동으로 컴포넌트를 스캔하기 때문이다.

* 스프링 데이터 JPA를 사용할 때, 인터페이스만 작성해도 동작한 이유

    * `스프링 데이터 JPA`는 애플리케이션을 실행할 때, `JpaRepository`를 상속 받은 리포지토리 인터페이스들을 찾아서 해당 인터페이스의 구현체(Proxy)를 생성해서 주입해준다.

    * 따라서 개발자가 직접 구현 클래스를 만들지 않아도 된다.

    * 그리고 `@Repository`를 생략할 수 있다.

        * `JpaRepository` 인터페이스를 상속 받으면 컴포넌트 스캔의 대상이 된다.

        * JPA 예외를 스프링 예외로 변환하는 과정도 자동으로 처리한다.
    
#### 2) 공통 인터페이스 적용

* `JpaRepository`의 제네릭을 보면 `T`는 엔티티 타입, `ID`는 식별자 타입(PK)을 의미한다.

#### 3) 공통 인터페이스 분석

* 공통 인터페이스(`JpaRepository`)는 CRUD 메소드를 제공한다.

* 제네릭은 `<엔티티 타입, 식별자 타입>`를 설정한다.

     ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
    }
     ```

* 공통 인터페이스 구성

    * JpaRepository 인터페이스의 계층 구조
    
        * `Repository`, `CrudRepository`, `PagingAndSortingRepository`는 스프링 데이터 프로젝트가 공통으로 사용하는 인터페이스다.
        
        * 스프링 데이터 JPA가 제공하는 `JpaRepository` 인터페이스는 여기에 추가로 JPA에 특화된 기능을 제공한다.
    
    * 제네릭 타입
    
        * `T` : 엔티티 타입 , `ID` : 엔티티의 식별자 타입 , `S` : 엔티티와 그 자식 타입 
    
    * JpaRepository의 주요 메소드
    
        * `save(S)` : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합(merge)한다.
        
        * `delete(T)` : 엔티티 하나를 삭제한다. 
        
            * 내부에서 `EntityManager.remove()`를 호출한다.
        
        * `findById(ID)` : 엔티티 하나를 조회한다. 
        
            * 내부에서 `EntityManager.find()`를 호출한다.
        
        * `getOne(ID)` : 엔티티를 프록시로 조회한다. 
        
            * 내부에서 `EntityManager.getReference()`를 호출한다. 
        
        * `findAll(...)` : 모든 엔티티를 조회한다. 
        
            * 정렬(`Sort`)이나 페이징(`Pageable`) 조건을 파라미터로 제공할 수 있다.

## 4. 쿼리 메소드 기능

#### 스프링 데이터 JPA가 제공하는 `쿼리 메소드 기능` 3 가지

* ① 메소드 이름으로 쿼리 생성

* ② 메소드 이름으로 JPA NamedQuery 호출

* ③ `@Query`를 사용해서 리포지토리 메소드에 직접 쿼리를 정의

#### 1) 메소드 이름으로 쿼리 생성

* **(1) 실습**

    * ① 스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행한다.
    
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> {
        
          List<Member> findByUsernameAndAge(String name, int age);
      
          List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
          
        }
        ```
    
        * `Username` : username이 ~ 와 같으면
        
        * `AgeGreaterThan` : age가 ~ 보다 크면
        
    * ② 위의 메소드는 다음과 같은 JPQL을 생성한다.
      
        * `findByUsernameAndAge()`
    
            ```java
            select m 
            from Member m 
            where m.username = :username and m.age = :age
            ```
          
        * `findByUsernameAndAgeGreaterThan()`
    
            ```java
            select m 
            from Member m 
            where m.username = :username and m.age > :age
            ```

    * ③ 테스트 코드를 작성한다.

        ```java
        @SpringBootTest
        @Transactional
        @Rollback(false)
        class MemberRepositoryTest {
        
            @Autowired MemberRepository memberRepository;
            @Autowired TeamRepository teamRepository;
            
            @Test
            public void findByUsernameAndGraterThan() {
                Member m1 = new Member("AAA", 10);
                Member m2 = new Member("BBB", 20);
                memberRepository.save(m1);
                memberRepository.save(m2);
        
                List<Member> result = memberRepository.findByUsernameAndAgeGreaterThan("AAA", 15);
        
                assertThat(result.get(0).getUsername()).isEqualTo("AAA");
                assertThat(result.get(0).getAge()).isEqualTo(20);
                assertThat(result.size()).isEqualTo(1);
            }
            
        }
        ```

* **(2) 메소드 이름에서 지원하는 키워드** 
  
    * 메소드 이름으로 쿼리를 생성할 때, 자주 사용되는 키워드는 다음과 같다.
    
        * 조회 : `find...By` , `read...By` , `query...By`, `get...By`
        
            * `...`는 아무 내용이나 입력해도 된다. 
              
                * Ex) `findHelloBy`
    
            * `By` 키워드 다음에는 where 조건을 지정한다.
        
                * `By` 키워드 다음에 where 조건을 지정하지 않으면 전체 조회가 된다.
                
                    * Ex) `findHelloBy()`    
    
        * AND : `And`
        
            * where 절에 and가 추가된다.
            
        * OR : `Or`
        
            * where 절에 or가 추가된다.
    
        * COUNT : `count...By`
        
            * 반환 타입 : `long`
            
        * EXISTS : `exists...By` 
        
            * 반환 타입 : `boolean`   
    
        * 삭제 : `delete...By`, `remove...By`
        
            * 반환 타입 : `void` 또는 `long` (삭제 횟수)
            
        * DISTINCT : `findDistinct`, `findMemberDistinctBy`
        
        * LIMIT : `findFirst3`, `findFirst`, `findTop`, `findTop3` 
    
    * 메소드 이름으로 쿼리를 생성할 때 사용되는 키워드를 좀 더 살펴보고 싶다면 다음 링크를 확인하자.
    
        * [스프링 데이터 JPA 공식 문서 - Supported keywords inside method names](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation "")
    
* **(3) 참고 사항**

    * `메소드 이름으로 쿼리 생성 방식`은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다. 
    
    * 그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
    
        * 애플리케이션 로딩 시점에 오류를 확인할 수 있는 것은 스프링 데이터 JPA의 큰 장점이다.
    
    * 그리고 **해당 기능은 검색 조건으로 사용되는 컬럼이 2개 이하일 때 사용하기 좋다.**

#### 2) JPA NamedQuery - 사용 X

* **(1) JPA NamedQuery**

    * `NamedQuery`는 쿼리에 이름을 부여해서 호출하는 방법이다.
    
        * **실무에서 거의 사용되지 않는다.**

* **(2) 실습**

    * ① `@NamedQuery` 애노테이션으로 Named 쿼리를 정의한다.
    
         ```java
        @Entity
        @NamedQuery(
                name = "Member.findByUsername",
                query = "select m from Member m where m.username = :username")
        public class Member {
            //...
        }
         ```

    * ② 스프링 데이터 JPA로 Named 쿼리 호출한다.
    
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> { 
            @Query(name = "Member.findByUsername") // 생략 가능
            List<Member> findByUsername(@Param("username") String username);
        }
        ```
    
        * `Named 쿼리`는 애플리케이션을 시작하는 시점에 JPQL 문법 오류를 발견할 수 있다.
        
        * `@Param` : 이름 기반 파라미터를 바인딩 할 때 사용한다.
    
            * JPQL에서 이름 기반 파라미터를 명확하게 작성했을 때 `@Param`를 사용한다.
    
                * Ex) `:파라미터명`

    * ③ 테스트 코드를 작성한다.
    
        ```java
        @SpringBootTest
        @Transactional
        @Rollback(false)
        class MemberRepositoryTest {
    
            // ...
            
            @Test
            public void testNamedQuery() {
                Member m1 = new Member("AAA", 10);
                Member m2 = new Member("BBB", 20);
                memberRepository.save(m1);
                memberRepository.save(m2);
        
                List<Member> result = memberRepository.findByUsername("AAA");
                Member findMember = result.get(0);
                assertThat(findMember).isEqualTo(m1);
            }
        
        }
        ```

#### 3) @Query, 리포지토리 메소드에 직접 쿼리 정의하기

* **(1) 실습**

    * ① `@Query`는 리포지토리 메소드에 직접 쿼리를 정의할 때 사용한다.
    
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> {
        
            @Query("select m from Member m where m.username = :username and m.age = :age")
            List<Member> findUser(@Param("username") String username, @Param("age") int age);
        
        }
        ```
    
        * 실행할 메서드에 정적 쿼리를 직접 작성하므로 `이름 없는 Named 쿼리`라 할 수 있다.
        
        * `@Query`에 직접 쿼리를 정의하는 것은 애플리케이션을 시작하는 시점에 JPQL 문법 오류를 발견할 수 있다는 장점이 있다. 
        
    * ② 테스트 코드를 작성한다.
    
        ```java
        @SpringBootTest
        @Transactional
        @Rollback(false)
        class MemberRepositoryTest {
    
            // ...
            
            @Test
            public void testQuery() {
                Member m1 = new Member("AAA", 10);
                Member m2 = new Member("BBB", 20);
                memberRepository.save(m1);
                memberRepository.save(m2);
        
                List<Member> result = memberRepository.findUser("AAA", 10);
                assertThat(result.get(0)).isEqualTo(m1);
            }
        }
        ```

* **(2) 쿼리 메소드 기능의 실행 순서**

    * ① 스프링 데이터 JPA는 `@Query`에 정의한 쿼리를 먼저 실행한다.

        * ⓐ `@Query`에 직접 쿼리를 정의 했다면 해당 쿼리를 먼저 실행한다.

        * ⓑ 또는 `@Query`에 name 속성으로 선언한 Named 쿼리를 찾아서 먼저 실행한다.

        * ⓒ 또는 `@Query`를 생략 했다면 `엔티티 이름 + .(점) + 메서드 이름`으로 Named 쿼리를 찾아서 먼저 실행한다.

    * ② 만약 실행할 쿼리가 없으면 `메소드 이름으로 쿼리 생성` 방식을 사용한다.

        * 필요하면 전략을 변경할 수 있지만 권장하지 않는다.

#### 4) @Query, 값과 DTO 조회하기

* **(1) 단순히 값 하나를 조회하기**

    * ① 엔티티(`m`)가 아닌 값 하나(`m.username`)를 조회하는 메소드를 추가한다.
      
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> {
            @Query("select m.username from Member m")
            List<String> findUsernameList();
        }
        ```

    * ② 테스트 코드를 작성한다.

        ```java
        @SpringBootTest
        @Transactional
        @Rollback(false)
        class MemberRepositoryTest {
    
            // ...
            
            @Test
            public void findUsernameList() {
                Member m1 = new Member("AAA", 10);
                Member m2 = new Member("BBB", 20);
                memberRepository.save(m1);
                memberRepository.save(m2);
        
                List<String> usernameList = memberRepository.findUsernameList();
        
                for (String s : usernameList) {
                    System.out.println("s = " + s);
                }
            }
        }
        ```

* **(2) DTO로 직접 조회하기**

    * `DTO (Data Transfer Object)` : 계층 간 데이터 교환을 위한 객체다.

        * ① DTO를 작성한다.
        
            ```java
            @Data
            public class MemberDto {
            
                private Long id;
                
                private String username;
                
                private String teamName;
                
                public MemberDto(Long id, String username, String teamName) {
                    this.id = id;
                    this.username = username;
                    this.teamName = teamName;
                }
            }
            ```
          
        * ② DTO로 직접 조회하려면 `new` 명령어를 사용한다. 그리고 DTO에 일치하는 생성자가 있어야 한다.
        
            ```java
            public interface MemberRepository extends JpaRepository<Member, Long> {           
                @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
                List<MemberDTO> findMemberDto();
            }
            ```
          
            * new 키워드 다음에 패키지 명을 포함한 전체 클래스명을 입력해야 한다.
            
                * 현재 `MemberDto`는 아래 경로에 존재한다고 가정한다.
                
                    * `src/main/java/study/datajpa/dto/MemberDto.java`
                
                * 소스코드 루트 (`src/main/java/`) 아래에서 부터 패키지 명을 작성하면 된다.
                
                    * `study.datajpa.dto.MemberDto(m.id, m.username, t.name)`          

        * ③ 테스트 코드를 작성한다.
    
            ```java
            @SpringBootTest
            @Transactional
            @Rollback(false)
            class MemberRepositoryTest {
            
                //...
                
                @Test
                public void findMemberDto() {
                    Team team = new Team("teamA");
                    teamRepository.save(team);
            
                    Member m1 = new Member("AAA", 10);
                    m1.changeTeam(team);
                    memberRepository.save(m1);
            
                    List<MemberDto> memberDto = memberRepository.findMemberDto();
            
                    for (MemberDto dto : memberDto) {
                        System.out.println("dto = " + dto);
                    }
                }
            }
            ``` 

#### 5) 파라미터 바인딩

* 파라미터 바인딩

    * 스프링 데이터 JPA는 `위치 기반 파라미터 바인딩`과 `이름 기반 파라미터 바인딩`을 모두 지원한다.
    
        ```java
        select m from Member m where m.username = ?0 // 위치 기반 
        select m from Member m where m.username = :name // 이름 기반
        ```
        
    * 코드 가독성과 유지 보수를 위해 `이름 기반 파라미터 바인딩`을 사용하자.
    
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> {
            
            // 이름 기반 파라미터 바인딩
            @Query("select m from Member m where m.username = :username")
            List<Member> findMembers(@Param("username") String username);
        }
        ```
      
        * 위치 기반은 순서가 바뀌면 문제가 될 수 있다.

* 컬렉션 파라미터 바인딩

    * `Collection` 타입으로 in 절을 지원한다.

        * ① IN 절로 조회하는 메소드를 추가한다.

            ```java
            // 쿼리 메소드 정의
            @Query("select m from Member m where m.username in :names")
            List<Member> findByNames(@Param("names") Collection<String> names); // "List<String> names"도 가능하다. 
            ```
          
            * `List<String> names`도 가능하다.

        * ② 테스트 코드를 작성한다.

            ```java
            @Test
            public void findByNames() {
                Member m1 = new Member("AAA", 10);
                Member m2 = new Member("BBB", 20);
                memberRepository.save(m1);
                memberRepository.save(m2);
                
                List<Member> result = memberRepository.findByNames(Arrays.asList("AAA", "BBB"));
                
                for (Member member : result) {
                    System.out.println("member = " + member);
                }
            }
            ```

#### 6) 반환 타입

* **(1) 유연한 반환 타입**
  
    * 스프링 데이터 JPA는 `유연한 반환 타입`을 지원한다.
       
        * ① 결과가 한 건 이상이면 반환 타입으로 `컬렉션`을 사용하고, 단건이면 `엔티티 타입(또는 Optional)`을 사용한다.
       
            ```java
            List<Member> findListByUsername(String username);           // 컬렉션
            Member findMemberByUsername(String username);               // 단건
            Optional<Member> findOptionalByUsername(String username);   // 단건 Optional
            ```
    
            * 쿼리 메소드를 작성할 때 사용할 수 있는 리턴 타입은 다음 링크를 확인하자.
        
                * [스프링 데이터 JPA 공식 문서 - Supported Query Return Types](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#appendix.query.return.types "")

        * ② 테스트 코드를 작성한다.
        
            ```java
            @SpringBootTest
            @Transactional
            @Rollback(false)
            class MemberRepositoryTest {
            
                // ...
                
                @Test
                public void returnType() {
                    Member m1 = new Member("AAA", 10);
                    Member m2 = new Member("BBB", 20);
                    memberRepository.save(m1);
                    memberRepository.save(m2);
            
                    Optional<Member> findMember = memberRepository.findOptionalByUsername("AAA");
                    System.out.println("findMember = " + findMember);
                }
            
            }
            ```

* **(2) 조회 결과가 많거나 없으면 어떻게 될까?**

    * ① 컬렉션 조회인 경우 (반환 타입 : 컬렉션)
    
        * 조회 결과가 없으면 비어있는 컬렉션을 반환한다.

            ```java
            @Test
            public void returnType() {
                // ...
          
                List<Member> result = memberRepository.findListByUsername("asdasdas"); // 조회 결과가 없으면, 비어 있는 컬렉션을 반환한다.
                System.out.println("result = " + result.size()); // 결과 : 0
            }
            ```

            * `null` 체크를 하지 않아도 된다.
    
    * ② 단건 조회인 경우 (반환 타입 : 엔티티 타입) 
    
        * 조회 결과가 없으면 `null`을 반환한다. (순수 JPA에서는 단건 조회 결과가 없으면 예외가 발생한다.)

            ```java
            @Test
            public void returnType() {
                // ...
          
                Member findMember = memberRepository.findMemberByUsername("asdasdas"); // 조회 결과가 없으면 null을 반환한다.
                System.out.println("findMember = " + findMember); // 결과 : null
            }
            ```
    
            * 조회 결과가 `null` 일 수도 있다면 반환 타입으로 단건 Optional(`Optional<Member>`)를 사용하자.
        
        * 조회 결과가 2건 이상이면 예외(`javax.persistence.NonUniqueResultException`)가 발생한다.

#### 7) 순수 JPA 페이징과 정렬

* 순수 JPA에서 페이징을 어떻게 할 것인가?

    * ① 페이징과 정렬을 하는 조건은 다음과 같다.
    
        * **검색 조건** : 나이가 10살
        
        * **정렬 조건** : 이름으로 내림차순
        
        * **페이징 조건** : 현재 페이지가 첫 번째 페이지며 페이지 당 보여줄 데이터는 3건

    * ② 페이징과 정렬을 사용하는 코드를 작성한다.
    
        ```java
        @Repository
        public class MemberJpaRepository {
            // 몇 번째(offset) 부터 시작해서 몇 개(limit)를 조회한다.
            public List<Member> findByPage(int age, int offset, int limit) { 
                return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
                        .setParameter("age", age)
                        .setFirstResult(offset)
                        .setMaxResults(limit)
                        .getResultList();
            }
        
            // 전체 개수 (전체 개수를 가져올 때는 정렬이 필요 없으므로 정렬하는 order by는 제거했다) 
            public long totalCount(int age) {
                return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
                        .setParameter("age", age)
                        .getSingleResult();
        
            }
        }
        ```
        
        * 페이징 예시
          
            * [`page 1`] offset = 0 , limit = 10  
              
            * [`page 2`] offset = 10, limit = 10 

            * ... 

    * ③ 테스트 코드를 작성한다.

        ```java
        @SpringBootTest
        @Transactional
        @Rollback(false)
        class MemberJpaRepositoryTest {
        
            // ...
      
            @Test
            public void paging() throws Exception {
                // given
                memberJpaRepository.save(new Member("member1", 10));
                memberJpaRepository.save(new Member("member2", 10));
                memberJpaRepository.save(new Member("member3", 10));
                memberJpaRepository.save(new Member("member4", 10));
                memberJpaRepository.save(new Member("member5", 10));
        
                int age = 10;
                int offset = 0;
                int limit = 3;
        
                // when
                List<Member> members = memberJpaRepository.findByPage(age, offset, limit);
                long totalCount = memberJpaRepository.totalCount(age);
        
                // then
                assertThat(members.size()).isEqualTo(3);
                assertThat(totalCount).isEqualTo(5);
            }
        }
        ```
      
        * JPA는 방언 (Dialect)이라는 개념이 존재하기 때문에 현재 데이터베이스에 맞게 페이징 쿼리가 실행된다.

#### 8) 스프링 데이터 JPA 페이징과 정렬

* **(1) 페이징, 정렬과 관련된 파라미터와 반환 타입** 

    * `스프링 데이터 JPA`는 쿼리 메소드에 `페이징`과 `정렬` 기능을 사용할 수 있도록 **2 가지 파라미터**를 제공한다.
    
        * ① `org.springframework.data.domain.Sort` : 정렬 기능을 지원한다.
    
            ```java
            Sort sort = Sort.by(Sort.Direction.ASC, "id");
          
            String sortProperty = sort.toString().contains("id") ? "id" : "updatedDate";
            ```
    
        * ② `org.springframework.data.domain.Pageable` : 페이징과 정렬 기능을 지원한다.
        
            * 내부에 `Sort`를 포함하고 있다.
    
    * 파라미터에 `Pageable`을 사용하면 다음과 같은 특별한 **반환 타입**을 사용할 수 있다. 
    
        * ① `org.springframework.data.domain.Page` : 추가 count 쿼리를 호출하는 페이징 결과를 반환한다.
        
            * `Page`의 인덱스는 0 부터 시작한다. 
        
        * ② `org.springframework.data.domain.Slice` : 추가 count 쿼리를 호출하지 않는 페이징 결과를 반환한다.
            
            * 다음 페이지 존재 여부를 확인 할 수 있다. (더보기 버튼) 
              
                * 내부적으로 limit + 1로 조회한다.
        
                * 즉, `limit + 1`는 한 페이지에 조회할 데이터 수 (limit) + 1로 조회한다는 의미다.
        
            * 더보기 기능을 구현할 때 사용한다.
        
        * ③ `자바 컬렉션` : 추가 count 쿼리 없이 결과만 반환한다.

            * Ex) `List`         

            * 데이터를 10개만 끊어서 가져올 때, 사용할 수 있다.
        
* **(2) 예시**
  
    * ① 쿼리 메소드를 정의한다.

        ```java
        public interface MemberRepository extends Repository<Member, Long> {
            Page<Member> findByAge(int age, Pageable pageable); // count 쿼리 사용
            
            Slice<Member> findByAge(int age, Pageable pageable); // count 쿼리 사용 안 함
            
            List<Member> findByAge(int age, Pageable pageable); // count 쿼리 사용 안 함
            
            List<Member> findByAge(int age, Sort sort);
        }
        ```
  
    * ② 쿼리 메소드를 사용하는 테스트 코드를 작성한다.
    
        ```java
        @Test
        public void paging() throws Exception {
            // given
            memberRepository.save(new Member("member1", 10)); 
            memberRepository.save(new Member("member2", 10)); 
            memberRepository.save(new Member("member3", 10)); 
            memberRepository.save(new Member("member4", 10)); 
            memberRepository.save(new Member("member5", 10));
    
            // when
            // 0 페이지에서 3개를 가져온다. 그리고 username으로 내림차순 정렬한다.
            PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username")); // 0 페이지를 조회하며 한 페이지에 조회할 데이터 수는 3개이며 정렬 기준도 설정해서 PageRequest를 만든다. 
            Page<Member> page = memberRepository.findByAge(10, pageRequest);        // 0 페이지에서 3개를 가져온다.
            // Slice<Member> page = memberRepository.findByAge(age, pageRequest);   // 0 페이지에서 3 + 1개를 가져온다.
      
            // then
            List<Member> content = page.getContent();
            assertThat(content.size()).isEqualTo(3);
            assertThat(page.getTotalElements()).isEqualTo(5);
            assertThat(page.getNumber()).isEqualTo(0);
            assertThat(page.getTotalPages()).isEqualTo(2);
            assertThat(page.isFirst()).isTrue();
            assertThat(page.hasNext()).isTrue();
        }
        ```
      
        * `findByAge()`
        
            * 두 번째 파라미터 `Pageable`은 인터페이스이며 페이징 처리에 필요한 정보를 제공할 때 사용한다.
        
            * 그리고 파라미터 타입이 `Pageable`이면 해당 인터페이스를 구현한 `PageRequest` 객체를 전달한다.
        
        * `PageRequest` : Pageable 인터페이스의 구현체다.
        
        * `PageRequest.of()`
        
            * **첫 번째 파라미터**(`page`)에는 **현재 페이지**를, **두 번째 파라미터**(`size`)에는 **조회할 데이터 수**를 전달한다.
         
            * 여기에 추가로 **정렬 정보**(`sort`)도 파라미터로 전달할 수 있다. 참고로 페이지는 0 부터 시작한다.
    
                * `PageRequest.of(int page, int size, Sort sort)`

* **(3) Page와 Slice 인터페이스**

    * `Page` 인터페이스
    
        * `Page`를 사용하면 전체 페이지 수를 알아내기 위해서 추가적인 카운트 쿼리가 실행된다. 
        
            * 기본적으로 카운트 쿼리는 실제로 실행되는 쿼리에서 파생된다.
        
                ```java
                public interface Page<T> extends Slice<T> {
                  int getTotalPages();      // 전체 페이지 수를 반환한다.
              
                  long getTotalElements();  // 전체 데이터 수를 반환한다.
              
                  <U> Page<U> map(Function<? super T, ? extends U> converter); // Page 내의 요소를 다른 요소로 변환한다.
                }
                ```
    
    * `Slice` 인터페이스
    
        * `Slice`를 사용하면 추가 카운트 쿼리를 실행하지 않고 다음 페이지 존재 여부를 확인할 수 있다. (내부적으로 `limit + 1`로 조회한다.)
    
            ```java
            public interface Slice<T> extends Streamable<T> {
                int getNumber();                  // 현재 페이지 번호를 반환한다.
                int getSize();                    // 페이지 크기 (한 페이지에 표시할 데이터 수)를 반환한다.
                int getNumberOfElements();        // 현재 페이지에 조회된 데이터 개수를 반환한다.
                List<T> getContent();             // 현재 페이지에 조회된 데이터를 반환한다.
                boolean hasContent();             // 현재 페이지에 조회된 데이터가 존재하는지를 반환한다.
                Sort getSort();                   // 정렬 정보를 반환한다.
                boolean isFirst();                // 현재 페이지가 첫 페이지 인지를 반환한다.
                boolean isLast();                 // 현재 페이지가 마지막 페이지 인지를 반환한다.
                boolean hasNext();                // 다음 페이지가 존재하는지를 반환한다.
                boolean hasPrevious();            // 이전 페이지가 존재하는지를 반환한다.
                Pageable getPageable();           // 페이지 요청 정보를 반환한다.
                Pageable nextPageable();          // 다음 페이지 객체를 반환한다.
                Pageable previousPageable();      // 이전 페이지 객체를 반환한다.
                <U> Slice<U> map(Function<? super T, ? extends U> converter); // Slice 내의 요소를 다른 요소로 변환한다.
            }
            ```
    
        * 더보기 기능을 구현할 때, 사용할 수 있다.

* **(4) 더 나아가기**

    * count 쿼리를 분리하기
    
        * count 쿼리에는 `left join` 조인을 하는 것이 의미가 없기 때문에 이럴 때 사용할 수 있다.
    
            ```java
            @Query(value = "select m from Member m left join m.team t",
                   countQuery = "select count(m) from Member m")
            Page<Member> findByAge(int age, Pageable pageable);
            ```

            * value 속성은 컨텐츠를 가져오는 쿼리를 작성하고 countQuery 속성은 카운트 쿼리를 작성한다.

            * 정렬 조건이 복잡해지면 `Sort`로 해결하기 어렵다. 
            
                * 이러한 경우에는 `@Query`의 JPQL로 작성된 쿼리에 정렬 기준을 직접 지정하자.
    
    * 페이지를 유지하면서 엔티티를 DTO로 변환하기
    
        * API를 작성할 때, 컨트롤러에서 엔티티를 바로 반환하면 안된다. 
        
        * 그 이유는 엔티티를 변경하면 API의 스펙이 변경되기 때문이다. (API 장애가 발생할 수 있음)
         
        * 그래서 엔티티는 절대 외부에 노출 시키지 않고 DTO로 변환해서 반환해야 한다.  
    
            ```java
            Page<Member> page = memberRepository.findByAge(age, pageRequest);
            Page<MemberDto> toMap = page.map(m -> new MemberDto(m.getId(), m.getUsername(), null));
            ```
          
            * `page.map()` : Page 내의 요소를 다른 요소로 변환한다.  

#### 9) 벌크성 수정 쿼리

* **(1) 순수 JPA에서 벌크 연산**

    * ① 벌크 연산을 하는 메소드를 추가한다.
    
        ```java
        public int bulkAgePlus(int age) {
            return em.createQuery(
                    "update Member m set m.age = m.age + 1" +
                            " where m.age >= :age")
                    .setParameter("age", age)
                    .executeUpdate();
        }
        ```

    * ② 테스트 코드를 작성한다.
    
        ```java
        @Test
        public void bulkUpdate() {
            // given
            memberJpaRepository.save(new Member("member1", 10));
            memberJpaRepository.save(new Member("member2", 19));
            memberJpaRepository.save(new Member("member3", 20));
            memberJpaRepository.save(new Member("member4", 21));
            memberJpaRepository.save(new Member("member5", 40));
        
            /*
            * 벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에,
            * 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 엔티티 상태가 달라질 수 있다.
            * 예를 들면, member5는 DB에는 41살 이지만, 영속성 컨텍스트에는 40살이다.
            * */
      
            // when
            int resultCount = memberJpaRepository.bulkAgePlus(20); // 해당 JPQL를 실행하기 전에 플러시가 동작하여 영속성 컨텍스트의 변경사항이 DB에 반영된다.
            // em.flush();  // 벌크 연산을 실행한 다음에, 영속성 컨텍스트를 플러시 해서 DB에 반영되지 않은 부분을 DB에 반영한다.
            // em.clear(); // 그리고 영속성 컨텍스트를 초기화하면 문제를 해결할 수 있다.   
      
            // then
            assertThat(resultCount).isEqualTo(3);
        }
        ```

* **(2) 스프링 데이터 JPA에서 벌크 연산**

    * ① 벌크 연산을 하는 메소드를 추가한다.
    
        ```java
        @Modifying(clearAutomatically = true)
        @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
        int bulkAgePlus(@Param("age") int age);
        ```
      
        * `@Modifying` : 스프링 데이터 JPA에서 벌크 연산을 할 때 사용한다. 
      
            * 사용할 수 있는 옵션은 다음과 같다.
              
                * `clearAutomatically` : 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트를 초기화 할지 지정한다.
    
                    * Ex) `clearAutomatically = true` : 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트를 초기화한다.
                    
                        * 해당 옵션을 사용하지 않고 회원을 `findByUsername()`로 다시 조회하면 영속성 컨텍스트에 과거 값이 남아 있어서 문제가 될 수 있다.
                        
                        * **벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 있는 엔티티 상태가 달라질 수 있다.**
                        
                        * 만약 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화 하자.    
    
                * `flushAutomatically` : 벌크성 쿼리를 실행하기 전에 영속성 컨텍스트를 플러시 할지 지정한다.

                    * JPQL을 실행하면, 해당 JPQL과 관련 있는 엔티티만 플러시한다.
                    
                    * 이 상태에서 `clearAutomatically`가 실행되면 플러시 되지 않은 내용에서 문제가 발생할 수 있다.    

                    * 이러한 이유로 `flushAutomatically` 속성이 추가된 것이다.
                    
                        * 해당 옵션은 JPQL을 실행하기 전에 영속성 컨텍스트의 모든 내용을 플러시 해서 `clearAutomatically`로 인한 문제를 방지한다.
    
            * 벌크 연산을 할 때, `@Modifying` 애노테이션을 사용하지 않으면 예외(`QueryExecutionRequestException`)가 발생한다.

    * ② 테스트 코드를 작성한다.
    
        ```java
        @Test
        public void bulkUpdate() {
            // given
            memberRepository.save(new Member("member1", 10));
            memberRepository.save(new Member("member2", 19));
            memberRepository.save(new Member("member3", 20));
            memberRepository.save(new Member("member4", 21));
            memberRepository.save(new Member("member5", 40));
        
            /*
            * 벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에,
            * 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 엔티티 상태가 달라질 수 있다.
            * 예를 들면, member5는 DB에는 41살 이지만, 영속성 컨텍스트에는 40살이다.
            * */
        
            // when
            int resultCount = memberRepository.bulkAgePlus(20); // 해당 JPQL를 실행하기 전에 플러시가 동작하여 영속성 컨텍스트의 변경사항이 DB에 반영된다.

            List<Member> result = memberRepository.findByUsername("member5");
            Member member5 = result.get(0);
            System.out.println("member5 = " + member5);
        
            // then
            assertThat(resultCount).isEqualTo(3);
        }
        ```
    
#### 10) @EntityGraph

* **(1) N+1 문제가 발생하는 경우**

    * 다음과 같은 테스트 코드를 작성한다.

        ```java
        @Test
        public void findMemberLazy() {
            // given
            Team teamA = new Team("teamA");
            Team teamB = new Team("teamB");
            teamRepository.save(teamA);
            teamRepository.save(teamB);
        
            Member member1 = new Member("member1", 10, teamA);
            Member member2 = new Member("member2", 10, teamB);
            memberRepository.save(member1);
            memberRepository.save(member2);
        
            em.flush();
            em.clear();
            
            // when
            List<Member> members = memberRepository.findAll();
            
            for (Member member : members) {
                System.out.println("member = " + member.getUsername());
                System.out.println("member.teamClass = " + member.getTeam().getClass()); // team이 프록시인지 실제 엔티티인지 확인한다. 
                System.out.println("member.team = " + member.getTeam().getName()); // N + 1 문제 발생
            }
        }
        ```
            
        * Member와 연관된 Team은 지연 로딩(LAZY)으로 설정되어 있다.
        
            * Member를 조회할 때, Team 필드에는 프록시 객체를 넣어둔다.
            
            * 그래서 `member.getTeam()`까지는 데이터베이스를 조회하지 않는다.
            
            * 그러나 `member.getTeam().getName()`를 하는 순간, 데이터베이스를 조회한다.
            
                * 즉, `Team`의 필드를 건드릴 때 위와 같이 동작한다.
        
        * 위의 코드는 `N + 1` 문제가 발생한다.
    
            * 먼저, Member를 조회하는 쿼리가 발생한다. (`1`)
    
                ```java
                select
                    member0_.member_id as member_i1_0_,
                    member0_.age as age2_0_,
                    member0_.team_id as team_id4_0_,
                    member0_.username as username3_0_ 
                from
                    member member0_
                ```
              
            * 그리고 team 필드를 사용할 때 마다 추가적인 N개의 쿼리가 발생한다. (`N`)
    
                ```java
                // teamA
                select
                    team0_.team_id as team_id1_1_0_,
                    team0_.name as name2_1_0_ 
                from
                    team team0_ 
                where
                    team0_.team_id=1
                ```
              
                ```java
                // teamB
                select
                    team0_.team_id as team_id1_1_0_,
                    team0_.name as name2_1_0_ 
                from
                    team team0_ 
                where
                    team0_.team_id=2
                ```
              
                * 이번 예제에서는 추가 쿼리가 2번 발생했다.
                
                * 만약 2개의 Member에 teamA만 설정 했다면 추가 쿼리는 1번만 발생하게 될 것이다.
                
                    * 그 이유는 영속성 컨텍스트에 이미 존재하기 때문이다.
          
* **(2) 엔티티를 조회할 때 연관된 엔티티들을 함께 조회하는 방법**

    * ① JPQL의 `페치 조인`을 사용한다.

        * ⓐ MemberRepository에 쿼리 메소드를 추가한다.

            ```java
            @Query("select m from Member m left join fetch m.team")
            List<Member> findMemberFetchJoin();
            ```

        * ⓑ 테스트 코드(findMemberLazy())의 내용을 수정한다.
        
            ```java
            @Test
            public void findMemberLazy() {
                // ...
                    
                List<Member> members = memberRepository.findMemberFetchJoin();
            
                // ...
            }
            ```
    
    * ② `@EntityGraph` : 어떤 엔티티를 조회하는 시점에 **연관된 엔티티들을 SQL 한 번에 함께 조회하는 기능**이다.

        * @EntityGraph는 사실상 페치 조인(FETCH JOIN)의 간편 버전이다.
        
        * 기본적으로 LEFT OUTER JOIN만 지원한다.

        * 페치 조인(엔티티 그래프)을 수행하면 특정 엔티티의 연관된 엔티티를 프록시 객체가 아닌 실제 엔티티 객체로 조회한다.
          
        * @EntityGraph를 사용하는 방법은 다음과 같다.
    
            * ⓐ MemberRepository에 쿼리 메소드를 추가한다.
    
                ```java
                // 공통 메서드를 오버라이딩
                @Override
                @EntityGraph(attributePaths = {"team"})
                List<Member> findAll();
                
                // JPQL + 엔티티 그래프
                // JPQL는 작성 했는데 페치 조인만 적용하고 싶은 경우에도 엔티티 그래프를 사용 할 수 있다.
                @EntityGraph(attributePaths = {"team"})
                @Query("select m from Member m") 
                List<Member> findMemberEntityGraph();
                
                // 메소드 이름으로 쿼리를 생성하는 경우에도 엔티티 그래프를 사용 할 수 있다.
                // 회원 엔티티를 조회할 때, 연관된 팀 엔티티도 함께 조회한다.
                @EntityGraph(attributePaths = {"team"})
                List<Member> findEntityGraphByUsername(@Param("username") String username);
                ```
                
                * `attributePaths` : 쿼리 수행 시, 즉시 로딩(EAGER LOADING)으로 함께 조회할 필드명을 지정한다.
    
            * ⓑ 테스트 코드(findMemberLazy())의 내용을 수정한다.

                ```java
                @Test
                public void findMemberLazy() {
                    // ...
                        
                    List<Member> members = memberRepository.findMemberEntityGraph();
                    // memberRepository.findEntityGraphByUsername("member1");
                    // memberRepository.findAll();
                
                    // ...
                }
                ```
    
    * ③ @NamedEntityGraph
    
        * ⓐ `@NamedEntityGraph`는 Named 엔티티 그래프를 정의할 때 사용한다.
                
            ```java
            @NamedEntityGraph(name = "Member.all", attributeNodes = @NamedAttributeNode("team"))
            @Entity
            public class Member {
              //...
            }
            ```
            
            * `name` : 엔티티 그래프의 이름을 정의한다.
            
            * `attributeNodes` : 함께 조회할 속성을 선택한다. 
              
                * 이때, `@NamedAttributeNode`를 사용하고 그 값으로 함께 조회할 속성을 선택하면 된다.

        * ⓑ `@EntityGraph`으로 Named 엔티티 그래프를 사용한다.
                
            ```java
            public interface MemberRepository extends JpaRepository<Member, Long> {
                @EntityGraph("Member.all") 
                List<Member> findMemberEntityGraph();  
            }
            ```
          
            * Named 엔티티 그래프는 잘 사용하지 않는 기능으로 보인다.
        
#### 11) JPA Hint & Lock

* **(1) JPA Hint**

    * `JPA Hint`는 SQL 힌트가 아닌 JPA 구현체(하이버네이트 등)에게 제공하는 힌트다.
    
    * JPA 쿼리 힌트를 사용하는 방법은 다음과 같다.
    
        * ① JPA 쿼리 힌트를 사용하는 메소드를 추가한다.
    
            ```java
            @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
            Member findReadOnlyByUsername(String username);
            ```
          
            * `readOnly`로 설정하면 변경 감지 기능이 동작하지 않는다.
            
            * 그리고 내부적으로 성능 최적화를 해서 스냅샷을 만들지 않는다.
            
                * 즉, 데이터를 변경하지 않고 화면에 출력하는 용도로만 사용한다.
                
                * 성능 테스트를 했을 때, 느리다면 해당 힌트를 적용하자. 
                  
                    * 무조건적으로 적용하는 것은 좋지 않다.

        * ② 테스트 코드를 작성한다.
    
            ```java
            @Test
            public void queryHint() {
                // given
                Member member1 = new Member("member1", 10);
                memberRepository.save(member1);
                em.flush();
                em.clear();
            
                // when
                Member findMember = memberRepository.findReadOnlyByUsername(member1.getUsername());
                findMember.setUsername("member2");
            
                em.flush(); // Update 쿼리가 실행되지 않는다.
            }
            ```

* **(2) @Lock**

    * 쿼리 시 락을 걸려면 `org.springframework.data.jpa.repository.Lock` 애노테이션을 사용하면 된다.

       ```java
       @Lock(LockModeType.PESSIMISTIC_WRITE)
       List<Member> findLockByUsername(String username);
       ```
      
    * JPA가 제공하는 락은 `자바 ORM 표준 JPA 프로그래밍` 책에서 `16.1 절`을 참고하자.
    
        * 실시간 트래픽이 많은 서비스에서는 가급적 락을 걸면 안된다.

## 5. 확장 기능

#### 1) 사용자 정의 리포지토리 구현

* **(1) 사용자 정의 리포지토리를 구현하는 이유**

    * 스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동으로 생성하기 때문에 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하게 되면 구현해야 하는 기능이 너무 많다.
 
    * 다음과 같은 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면 `사용자 정의 인터페이스`를 작성한다.

        * ① JPA 직접 사용(`EntityManager`)
          
        * ② **스프링 JDBC Template 사용** 
          
        * ③ MyBatis 사용
          
        * ④ 데이터베이스 커넥션 직접 사용
          
        * ⑤ **Querydsl 사용**
    
            * 복잡한 동적 쿼리를 작성하는 경우

* **(2) 사용자 정의 리포지토리를 작성하는 방법**

    * ① 사용자 정의 인터페이스를 작성한다.
    
        ```java
        public interface MemberRepositoryCustom {
            List<Member> findMemberCustom();
        }
        ```
      
    * ② 사용자 정의 인터페이스를 구현한 클래스를 작성한다.

        ```java
        @RequiredArgsConstructor
        public class MemberRepositoryCustomImpl implements MemberRepositoryCustom { 
            
            // JPA 직접 사용
            private final EntityManager em;
        
            @Override
            public List<Member> findMemberCustom() {
                return em.createQuery("select m from Member m")
                        .getResultList();
            }
            
        }
        ```

        * 클래스 이름을 짓는 규칙을 따르면 스프링 데이터 JPA가 사용자 정의 구현 클래스로 인식한다.

            * 클래스 이름을 짓는 규칙은 기본적으로 `리포지토리 인터페이스명 + Impl`이다.
        
                * Ex) `MemberRepositoryImpl`
        
            * **스프링 데이터 2.x 부터 사용자 정의 인터페이스를 구현한 클래스의 이름을 `사용자 정의 인터페이스명 + Impl`로 하는 것이 가능하다.**
    
                * Ex) `MemberRepositoryCustomImpl`    
    
                    * 해당 방식이 사용자 정의 인터페이스명과 구현 클래스명이 비슷하면서 더 직관적이므로 해당 방식을 사용하는 것을 권장한다.
    
        * `@RequiredArgsConstructor` : final 필드만을 가지고 생성자를 만들어준다.
        
            * 스프링은 생성자가 하나인 경우에 `@Autowired`가 없더라도 자동으로 의존성을 주입한다.
            
            * 그리고 스프링 데이터 JPA를 사용하면 `EntityManager`에 `@PersistenceContext` 대신에 `@Autowired`를 사용할 수 있다.
            
            * 그래서 위의 코드처럼 생성자 주입으로 작성할 수 있다.
        
    * ③ 특정 리포지토리 인터페이스에 사용자 정의 인터페이스를 상속 받도록 한다.
    
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
        }
        ```
      
    * ④ 사용자 정의 메서드를 호출한다.
    
        ```java
        List<Member> result = memberRepository.findMemberCustom();
        ```

* **(3) 사용자 정의 구현 클래스 이름 끝에 Impl 대신 다른 이름으로 변경하기** 
  
    * 자바 설정 파일에 아래 내용처럼 적용하면 변경할 수 있다.
    
        ```java
        @EnableJpaRepositories(basePackages = "study.datajpa.repository",
                               repositoryImplementationPostfix = "Impl")
        ```

    * 하지만 왠만하면 관례를 따르도록 하자.
    
* **(4) 참고 사항**

    * 항상 사용자 정의 리포지토리가 필요한 것은 아니다. 그냥 임의의 리포지토리를 만들어도 된다. 

        * 핵심 비즈니스 로직이 있는 리포지토리와 화면에 맞춘 통계 쿼리가 있는 리포지토리는 분리해야 한다.

        * 예를 들어 MemberQueryRepository를 인터페이스가 아닌 클래스로 작성하고 스프링 빈으로 등록한 다음, 직접 사용해도 된다. 

            ```java
            @Repository
            @RequiredArgsConstructor
            public class MemberQueryRepository {
            
                private final EntityManager em;
            
                List<Member> findAllMembers(){
                    return em.createQuery("select m from Member m ")
                            .getResultList();
                }
            
            }
            ```
    
            * 물론 이 경우에는 스프링 데이터 JPA 와는 아무런 관계 없이 별도로 동작한다. 

#### 2) Auditing

* **(1) Auditing**

    * `Auditing` : 엔티티를 변경하는 시점에 언제, 누가 변경 했는지에 대한 정보를 기록하는 기능이다.
            
        * Ex) 등록일, 수정일, 등록자, 수정자
    
* **(2) 순수 JPA를 사용한 Auditing 구현**

    * ① 공통 매핑 정보를 제공하는 JpaBaseEntity 클래스를 작성한다.

        ```java
        @MappedSuperclass
        @Getter
        public class JpaBaseEntity {
        
            @Column(updatable = false)
            private LocalDateTime createdDate;
        
            private LocalDateTime updatedDate;
        
            // persist() 하기 전에 이벤트가 발생
            @PrePersist
            public void prePersist() {
                LocalDateTime now = LocalDateTime.now();
                createdDate = now;
                updatedDate = now;
            }
        
            // update() 하기 전에 이벤트가 발생
            @PreUpdate
            public void preUpdate() {
                updatedDate = LocalDateTime.now();
            }
            
        }
        ```
      
        * JPA는 주요 이벤트 애노테이션을 제공한다. (`@PrePersist`, `@PostPersist`, `@PreUpdate`, `@PostUpdate`)

            * `@PrePersist` : 해당 애노테이션을 적용한 메소드가 `persist()`를 하기 전에 호출된다.

            * `@PreUpdate` : 해당 애노테이션을 적용한 메소드가 수정하기 전에 호출된다.
            
            * ...

    * ② 특정 엔티티가 JpaBaseEntity 클래스를 상속 받도록 한다.
    
        ```java
        public class Member extends JpaBaseEntity {}
        ```

    * ③ 테스트 코드를 작성한다.
    
        ```java
        @SpringBootTest
        @Transactional
        @Rollback(false)
        public class MemberTest {
        
            @PersistenceContext
            EntityManager em;
        
            @Autowired
            MemberRepository memberRepository;
        
            // ...
        
            @Test
            public void JpaEventBaseEntity() throws Exception {
                // given
                Member member = new Member("member1");
                memberRepository.save(member); //@PrePersist가 적용된 메소드 호출
        
                Thread.sleep(100);
                member.setUsername("member2");
                em.flush(); // @PreUpdate가 적용된 메소드 호출
                em.clear();
        
                // when
                Member findMember = memberRepository.findById(member.getId()).get();
        
                // then
                System.out.println("findMember.createdDate = " + findMember.getCreatedDate());
                System.out.println("findMember.updatedDate = " + findMember.getUpdatedDate());
            }
        }
        ```

* **(3) 스프링 데이터 JPA를 사용한 Auditing 구현**

    * ① 스프링 부트의 메인 클래스(`DataJpaApplication`)에 `@EnableJpaAuditing`를 적용한다.
    
        ```java
        @EnableJpaAuditing
        @SpringBootApplication
        public class DataJpaApplication {
        
            public static void main(String[] args) {
                SpringApplication.run(DataJpaApplication.class, args);
            }
        
        }
        ```
      
        * `@EnableJpaAuditing` : Auditing 기능을 활성화한다.

    * ② `BaseEntity` 클래스를 작성한다.
     
        ```java
        @EntityListeners(AuditingEntityListener.class)
        @MappedSuperclass
        @Getter
        public class BaseEntity {
        
            // 등록일
            @CreatedDate
            @Column(updatable = false)
            private LocalDateTime createdDate;
        
            // 수정일
            @LastModifiedDate
            private LocalDateTime lastModifiedDate;
        
            /* 등록되거나 수정될 때 마다, auditorProvider()를 호출한 결과를 자동으로 채워 넣는다. */
            // 등록자
            @CreatedBy
            @Column(updatable = false)
            private String createdBy;
        
            // 수정자
            @LastModifiedBy
            private String lastModifiedBy;
        
        }
        ```

        * 소스코드 설명
          
            * `@EntityListeners(AuditingEntityListener.class)` : 해당 클래스에 Auditing 기능을 포함시킨다.
            
            * `@MappedSuperclass` : 자식 클래스에게 공통 매핑 정보를 제공하는 부모 클래스를 작성할 때 사용한다.
            
                * 여기서 공통 매핑 정보는 createdDate, lastModifiedDate, createdBy, lastModifiedBy를 말한다.
    
            * `@Column(updatable = false)` : 해당 컬럼의 값은 처음 값을 지정한 이후, 수정하지 못하도록 한다.

        * 등록(수정)일, 등록(수정)자 관련 애노테이션
          
            * `@CreatedDate` : 엔티티가 생성되어 저장될 때, 시간이 자동 저장된다.
            
            * `@LastModifiedDate` : 조회한 엔티티의 값을 변경할 때, 시간이 자동 저장된다. 
            
            * `@CreatedBy` : 엔티티가 생성되어 저장될 때, 생성자가 자동 저장된다.
            
            * `@LastModifiedBy` : 조회한 엔티티의 값을 변경할 때, 수정자가 자동 저장된다. 

    * ③ 자바 설정 파일에서 등록자, 수정자를 처리해주는 `AuditorAware` 스프링 빈을 등록한다. (등록자, 수정자 필드에 Auditing 기능을 적용하는 경우에만 작성함)

        ```java
        @EnableJpaAuditing
        @SpringBootApplication
        public class DataJpaApplication {
        
            public static void main(String[] args) {
                SpringApplication.run(DataJpaApplication.class, args);
            }
        
            @Bean
            public AuditorAware<String> auditorProvider() {
                // 랜덤 UUID 생성
                return () -> Optional.of(UUID.randomUUID().toString());
            }
        }
        ```
        
        * 위의 코드에서는 랜덤한 UUID를 생성해서 지정하였다.
        
        * 하지만 실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 유저 ID를 가져와서 지정한다.

    * ④ BaseEntity 클래스를 상속 받는다.
        
        ```java
        public class Member extends BaseEntity {}
        ```

    * ⑤ 앞서 작성한 테스트 코드(MemberTest의 JpaEventBaseEntity())를 실행해서 확인한다.

* **(4) 등록, 수정 시간과 등록, 수정자를 분리하기**

    ```java
    @EntityListeners(AuditingEntityListener.class)
    @MappedSuperclass
    @Getter
    public class BaseTimeEntity {
        @CreatedDate
        @Column(updatable = false)
        private LocalDateTime createdDate;
    
        @LastModifiedDate
        private LocalDateTime lastModifiedDate;
    }
    
    @Getter
    public class BaseEntity extends BaseTimeEntity {
        @CreatedBy
        @Column(updatable = false)
        private String createdBy;
    
        @LastModifiedBy
        private String lastModifiedBy;
    }
    ```

    * 실무에서 대부분의 엔티티는 등록 시간과 수정 시간이 필요하지만, 등록자와 수정자는 필요없는 경우가 있다.
  
    * 그래서 위와 같이 Base 타입을 분리한 다음, 원하는 타입을 선택해서 상속할 수 있도록 한다.
    
        * 예를 들어, 등록 시간과 수정 시간만 필요하다면 `BaseTimeEntity`를 상속하고 둘 다 필요하다면 `BaseEntity`를 상속한다. 
    
#### 3) Web 확장

* **(1) 도메인 클래스 컨버터**

    * `도메인 클래스 컨버터`는 HTTP 요청으로 전달받은 엔티티의 ID로 엔티티 객체를 찾아서 바인딩한다.
    
        ```java
        @RestController
        @RequiredArgsConstructor
        public class MemberController {
        
            private final MemberRepository memberRepository;
        
            // 도메인 클래스 컨버터 사용 전
            @GetMapping("/members/{id}")
            public String findMember(@PathVariable("id") Long id) {
                Member member = memberRepository.findById(id).get();
                return member.getUsername();
            }
        
            // 도메인 클래스 컨버터 사용 후
            @GetMapping("/members2/{id}")
            public String findMember2(@PathVariable("id") Member member) {
                return member.getUsername();
            }
        
        
            @PostConstruct
            public void init() {
                memberRepository.save(new Member("userA"));
            }
        
        }
        ```
      
        * 도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, 이 엔티티는 단순 조회용으로만 사용해야 한다. 
        
        * 트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영되지 않는다.

* **(2) 페이징과 정렬**

    * 스프링 MVC에서 스프링 데이터가 제공하는 페이징과 정렬 사용하기
      
        ```java
        @RestController
        @RequiredArgsConstructor
        public class MemberController {
        
            private final MemberRepository memberRepository;
                  
            @GetMapping("/members")
            public Page<Member> list(Pageable pageable) {
                Page<Member> page = memberRepository.findAll(pageable);
                return page;
            }
                  
        }
        ```

        * `Page` : 페이징에 대한 결과 정보를 의미한다.
        
        * `Pageable` : 페이징에 대한 파라미터 정보를 의미한다.
        
            * `Pageable` 타입의 파라미터는 다음과 같은 `요청 파라미터` 정보로 만들어진다. 
            
                * Ex) `/members?page=0&size=3&sort=id,desc&sort=username,desc`
                         
                    * `page` : 현재 페이지를 의미한다. (0 부터 시작함)
                    
                    * `size` : 한 페이지에 표시할 데이터 수를 의미한다.
                    
                    * `sort` : 정렬 조건을 의미한다. (ASC | DESC)
                    
                        * ASC는 기본 값이므로 생략 가능하다.
        
                            * Ex) `sort=username`
    
                * 사용자가 웹 브라우저에서 요청 파라미터 정보를 입력하면 `PageRequest` 객체를 생성해서 컨트롤러의 메소드 파라미터로 전달한다.

    * Pageable 기본 값을 변경하기
    
        * 글로벌 설정 (스프링 부트)
        
            ```
            # 기본 페이지 사이즈
            spring.data.web.pageable.default-page-size=20
            # 최대 페이지 사이즈
            spring.data.web.pageable.max-page-size=2000
            ```
          
            * Pageable 기본 값은 사용자가 요청 파라미터 정보를 입력하지 않을 때, 사용할 값이다.
              
            * 여기서 `page-size`는 한 페이지에 표시할 데이터 수를 의미한다.
          
        * 개별 설정
          
            * `@PageableDefault` 애노테이션을 사용한다.
            
                ```java
                @GetMapping("/members")
                public Page<Member> list(@PageableDefault(size = 5, sort = "username") Pageable pageable) {
                    Page<Member> page = memberRepository.findAll(pageable);
                    return page;
                }
                ```
              
                * 개별 설정은 글로벌 설정 보다 우선적으로 적용된다.
          
    * 접두사        

        * 사용해야 할 페이징 정보가 둘 이상이면 접두사로 구분한다.
        
        * `@Qualifier`에 접두사명을 추가한다. `"{접두사명}_xxx”`
        
            * Ex) `/members?member_page=0&order_page=1`
    
                ```java
                public String list(
                        @Qualifier("member") Pageable memberPageable, 
                        @Qualifier("order") Pageable orderPageable, ...
                ```
                      
    * Page의 내용을 DTO로 변환하기       

        * 엔티티를 외부 API로 노출하면 다양한 문제가 발생한다. 그래서 엔티티는 꼭 DTO로 변환해서 반환해야 한다.
     
            * 외부 API로 반환할 때는 컨트롤러의 리턴 타입을 `Page<Member>`가 아닌 `Page<MemberDto>`로 사용해야 한다.
     
        * Page는 `map()`을 지원해서 내부 데이터를 다른 것으로 변경할 수 있다.

            * ① DTO를 작성한다.

                ```java
                @Data
                public class MemberDto {          
                    private Long id;
                
                    private String username;
                    
                    public MemberDto(Member member) {
                        this.id = member.getId();
                        this.username = member.getUsername();
                    }
                }
                ```
              
                * DTO에서 메소드의 파라미터로 엔티티(Member)를 사용해도 된다. 
    
                    * 단, DTO의 필드로 엔티티를 사용하는 것은 안된다.

            * ② `Page.map()`을 사용해서 엔티티를 DTO로 변경한다.
            
                ```java
                @GetMapping("/members")
                public Page<MemberDto> list(@PageableDefault(size = 5) Pageable pageable) {
                    return memberRepository.findAll(pageable)
                      .map(MemberDto::new);
                }
                ```
              
                * Page 타입은 컨트롤러에서 반환해도 된다.

    * Page를 1 부터 시작하기      

        * 스프링 데이터는 Page를 0 부터 시작한다. 만약 1 부터 시작하려면 어떻게 해야할까?
        
            * 첫 번째 방법 (실질적인 해결책)
            
                * ① `Pageable`과 `Page`를 핸들러 메소드의 파라미터와 리턴 값으로 사용하지 않는다.
                  
                * ② 페이징을 요청하는 클래스를 직접 만들어서 핸들러 메소드의 파라미터로 사용한다. 
                
                    * 리포지토리에 전달할 `PageRequest`(Pageable 구현체)를 직접 만든다. 
    
                    * 리포지토리에서 조회할 때, 앞서 만든 PageRequest를 전달한다.
                
                * ③ 핸들러 메소드의 리턴 값도 Page를 대신하는 클래스를 만들어서 제공한다.
                
            * 두 번째 방법
            
                * `application.properties`에서 `spring.data.web.pageable.one-indexed-parameters`를 `true`로 설정한다. 
                                
                    * 예를 들어, 클라이언트가 `http://localhost:8080/members?page=1` 처럼 1번 페이지를 요청한다. 
                    
                        ```json
                        {
                            "content": [
                                {
                                    "id": 1,
                                    "username": "user0",
                                    "teamName": null
                                },
                                {
                                    "id": 2,
                                    "username": "user1",
                                    "teamName": null
                                },
                                {
                                    "id": 3,
                                    "username": "user2",
                                    "teamName": null
                                },
                                {
                                    "id": 4,
                                    "username": "user3",
                                    "teamName": null
                                },
                                {
                                    "id": 5,
                                    "username": "user4",
                                    "teamName": null
                                }
                            ],
                            "pageable": {
                                "sort": {
                                    "sorted": false,
                                    "unsorted": true,
                                    "empty": true
                                },
                                "offset": 0,
                                "pageNumber": 0, // 0 인덱스
                                "pageSize": 5,
                                "paged": true,
                                "unpaged": false
                            },
                            "last": false,
                            "totalPages": 20,
                            "totalElements": 100,
                            "numberOfElements": 5,
                            "first": true,
                            "number": 0, // 0 인덱스
                            "size": 5,
                            "sort": {
                                "sorted": false,
                                "unsorted": true,
                                "empty": true
                            },
                            "empty": false
                        }
                        ```
                        
                    * 그런데 이 방법은 web에서 page 파라미터를 -1 처리를 할 뿐이다.
                    
                    * 따라서 응답 값인 Page에서 페이지의 인덱스가 모두 0 부터 시작하는 한계가 있다.

        * 가급적 권장하는 것은 페이지 인덱스를 0 부터 처리하자.
        
## 6. 스프링 데이터 JPA 분석

#### 1) 스프링 데이터 JPA 구현체 분석

* **(1) 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체**
  
    * 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체는 `SimpleJpaRepository`다.
    
        * `org.springframework.data.jpa.repository.support.SimpleJpaRepository`
    
    * [참고] JpaRepository 인터페이스 안에서 왼쪽 아이콘을 클릭하면 확인 가능하다.
    
* **(2) SimpleJpaRepository의 코드를 살펴보기**

    * ① 클래스 레벨에 `@Repository` 적용
    
        * 컴포넌트 스캔의 대상이 되어 스프링 빈으로 등록된다.
        
        * 하부 기술의 예외를 스프링이 추상화한 예외로 변환한다.

            * 하부 기술을 JDBC에서 JPA로 변경 하더라도 예외를 처리하는 메커니즘이 동일하다는 장점이 있다.

    * ② 클래스 레벨에 `@Transactional(readOnly = true)` 적용
    
        * 기본적으로 읽기 전용으로 트랜잭션을 처리 하도록 클래스 레벨에 애노테이션이 지정되어 있다.
        
            * 저장, 변경, 삭제를 하는 메소드 보다 조회하는 메소드가 더 많기 때문이다.
        
        * 데이터를 조회만 하고 변경하지 않는 트랜잭션에서는 `readOnly = true` 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있다.
        
            * 즉, `readOnly = true`는 플러시를 생략한다.

    * ③ 메소드 레벨에 `@Transactional` 적용
        
        * JPA의 모든 변경은 트랜잭션 안에서 이루어져야 한다.
            
            * `스프링 데이터 JPA`에서 변경(등록, 수정, 삭제) 메소드는 트랜잭션 안에서 처리해야 한다. (그렇지 않으면 예외가 발생함)
            
                ```java
                @Transactional
                @Override
                public <S extends T> S save(S entity) {
            
                    if (entityInformation.isNew(entity)) {
                        em.persist(entity);
                        return entity;
                    } else {
                        return em.merge(entity);
                    }
                }
                ```
                
                * 스프링 데이터 JPA에서 등록, 삭제 메소드는 `@Transactional`로 트랜잭션 처리가 되어 있다.

            * 서비스 계층에서 트랜잭션을 시작하면 리포지토리는 해당 트랜잭션을 전파 받아서 사용한다.
         
            * 서비스 계층에서 트랜잭션을 시작하지 않으면 리포지토리에서 트랜잭션을 시작한다.
            
            * 그래서 스프링 데이터 JPA를 사용할 때 트랜잭션이 없어도 데이터 등록, 변경이 가능했던 것이다. 
              
                * 사실은 트랜잭션이 리포지토리 계층에 이미 걸려있는 것이다.
    
    * ④ `save()` 메소드
        
        * 새로운 엔티티면 저장(`persist`)하고 새로운 엔티티가 아니면 병합(`merge`)한다.

#### 2) 새로운 엔티티를 구별하는 방법

* **(1) 새로운 엔티티를 판단하는 기본 전략**

    * ① 식별자가 객체일 때는 값이 `null`이면 새로운 엔티티로 판단한다.
    
        * `em.persist()`를 호출한 이후, 식별자가 설정된다.
    
    * ② 식별자가 자바 기본 타입일 때는 값이 `0`이면 새로운 엔티티로 판단한다. 
    
        * `Persistable` 인터페이스를 구현해서 판단 로직을 변경할 수 있다.

* **(2) 식별자를 자동으로 할당받는 경우**

    * JPA 식별자 생성 전략이 `@Id + @GeneratedValue`면 `save()`를 호출하는 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 정상적으로 동작한다.

        * `save()`를 호출하는 시점에는 식별자가 없기 때문에 새로운 엔티티를 판단하는 if 문 내부로 진입한 다음, `em.persist()`를 호출해서 식별자를 할당받는다.

            ```java
            @Repository
            @Transactional(readOnly = true)
            public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
                // ...
            
                @Transactional
                @Override
                public <S extends T> S save(S entity) {
            
                    if (entityInformation.isNew(entity)) { // 해당 if 문 내부로 진입한다.
                        em.persist(entity); // persist()를 호출한다.
                        return entity;
                    } else {
                        return em.merge(entity);
                    }
                }
                
            }
            ```

            * [참고] `@GeneratedValue`를 사용하면 `em.persist()`를 호출하는 시점에 식별자를 조회해서 할당한다.

* **(3) 직접 식별자를 할당해야 되는 경우**

    * `@Id`만 사용해서 직접 식별자를 할당하게 되면 이미 식별자 값이 있는 상태로 `save()`를 호출하게 된다. 
    
    * 그러면 `merge()`가 호출되며 `merge()`는 우선 DB에서 값을 조회하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율적이다.
    
    * 따라서 **`Persistable` 인터페이스를 구현해서 새로운 엔티티를 판단하는 로직을 직접 작성한다.**
    
        ```java
        @Entity
        @EntityListeners(AuditingEntityListener.class) // BaseEntity를 사용하면 해당 애노테이션은 생략할 수 있다.
        @NoArgsConstructor(access = AccessLevel.PROTECTED)
        public class Item implements Persistable<String> { // Persistable<식별자 타입>
        
            @Id
            private String id;
            
            @CreatedDate
            private LocalDateTime createdDate;
        
            public Item(String id) {
                this.id = id;
            }
        
            // 오버라이딩
            @Override
            public String getId() {
                return id;
            }
            
            /*
            * @CreatedDate는 persist()가 호출되어 엔티티를 저장할 때 시간이 자동 저장된다.   
            * save()가 호출되면 내부에서 isNew()로 새로운 엔티티 여부를 판단하는 if 문을 만나게 된다.
            * 여기서 true가 되면 persist()를 호출하면서 엔티티를 저장한다.  
            * 따라서 새로운 엔티티 여부를 판단하는 isNew()의 로직은 다음과 같이 작성한다.
            * -> createdDate가 null이면 새로운 엔티티로 판단한다.
            * -> createdDate의 값이 있다면 새로운 엔티티가 아닌 것으로 판단한다.
            * */        
            @Override
            public boolean isNew() { // 새로운 엔티티를 판단하는 로직
                return createdDate == null;
            }
        
        }
        ```
      
        ```java
        @SpringBootTest
        class ItemRepositoryTest {
        
            @Autowired ItemRepository itemRepository;
        
            @Test
            public void save() {
                Item item = new Item("A");
                itemRepository.save(item);
            }
        
        }
        ```
  
## 7. 나머지 기능들

* 아래 내용은 실무에서 자주 사용되는 기능은 아니지만 알고 있으면 가끔씩 편리하게 사용 할 수 있는 기능이다.

* 아래 내용에 대해서 궁금하다면 해당 강좌를 참고하자.

#### 1) Specifications (명세)

#### 2) Query By Example

#### 3) Projections

#### 4) 네이티브 쿼리


