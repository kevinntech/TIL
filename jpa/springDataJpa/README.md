# 김영한님의 실전! 스프링 데이터 JPA
> 아래 내용은 [실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84 "실전! 스프링 데이터 JPA") 강좌를 정리한 내용 입니다.

## 1. 프로젝트 환경설정

* `JUnit Vintage` : JUnit 4로 작성한 테스트 코드를 실행할 때, 사용하는 모듈이다.

* H2 데이터베이스 설치

    * Mac을 사용하는 경우, `chmod 755 h2.sh`로 권한을 부여해야 된다.

* 모든 로그 출력은 가급적 로거를 통해 남겨야 한다.

    * `show_sql` 옵션은 `System.out`에 하이버네이트 실행 SQL을 남긴다.
    
    * `org.hibernate.SQL` 옵션은 `로거(logger)`를 통해 하이버네이트 실행 SQL을 남긴다.
    
         ```
        spring:
          ...
        
          jpa:
            hibernate:
              ddl-auto: create
            properties:
              hibernate:
                #show_sql: true
                format_sql: true
        
        logging.level:
          org.hibernate.SQL: debug
          #org.hibernate.type: trace
         ```

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
            
* 쿼리 파라미터 로그 남기기

    * `org.hibernate.type` 옵션은 SQL 실행 파라미터를 로그로 남긴다.
    
    * 외부 라이브러리 사용
    
        * 스프링 부트를 사용하면 해당 라이브러리만 추가하면 된다.
        
            * `implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'`
        
        * 참고: 쿼리 파라미터를 로그로 남기는 외부 라이브러리는 시스템 자원을 사용하므로, 개발 단계에서는 편하게 사용해도 된다. 하지만 운영시스템에 적용하려면 꼭 성능테스트를 하고 사용하는 것이 좋다.
    
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
    public class Member{
    
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
    
        public void changeTeam(Team team){
            this.team = team;
            team.getMembers().add(this);
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

    * **스프링 부트를 사용하면 해당 애노테이션을 생략 할 수 있다.** 
    
    * `@SpringBootApplication`가 위치한 패키지 이하 부터는 자동으로 컴포넌트를 스캔하기 때문이다.

* 스프링 데이터 JPA를 사용할 때, 인터페이스만 작성해도 동작한 이유

    * `스프링 데이터 JPA`는 애플리케이션을 실행할 때, `JpaRepository`를 상속 받은 리포지토리 인터페이스들을 찾아서
    
    * 해당 인터페이스의 구현체를 생성해서 주입해준다.

        * 따라서 개발자가 직접 구현 클래스를 만들지 않아도 된다.

* `JpaRepository` 인터페이스를 상속 받으면 컴포넌트 스캔의 대상이 되며 `@Repository`를 생략 할 수 있다.

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
    
        * `T` : 엔티티 , `ID` : 엔티티의 식별자 타입 , `S` : 엔티티와 그 자식 타입 
    
    * JpaRepository의 주요 메소드
    
        * `save(S)` : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합(merge)한다.
        
        * `delete(T)` : 엔티티 하나를 삭제한다. 
        
            * 내부에서 `EntityManager.remove()`를 호출한다.
        
        * `findById(ID)` : 엔티티 하나를 조회한다. 
        
            * 내부에서 `EntityManager.find()`를 호출한다.
        
        * `getOne(ID)` : 엔티티를 프록시로 조회한다. 
        
            * 내부에서 `EntityManager.getReference()`를 호출한다. 
        
        * `findAll(...)` : 모든 엔티티를 조회한다. 
        
            * 정렬(`Sort`)이나 페이징(`Pageable`) 조건을 파라미터로 제공 할 수 있다.

## 4. 쿼리 메소드 기능

* 스프링 데이터 JPA가 제공하는 `쿼리 메소드 기능 3가지`

    * ① 메소드 이름으로 쿼리 생성
    
    * ② 메소드 이름으로 JPA NamedQuery 호출
    
    * ③ @Query 어노테이션을 사용해서 리포지토리 인터페이스에 쿼리 직접 정리

#### 1) 메소드 이름으로 쿼리 생성

* 스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행한다.

     ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
    
        List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
    
    }
     ```

* 메소드 이름으로 사용 할 수 있는 키워드

 ``   * [스프링 데이터 JPA 공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods "") 참고

* 스프링 데이터 JPA가 제공하는 쿼리 메소드 기능

    * 조회 : `find...By` , `read...By` , `query...By`, `get...By`
    
        * `findHelloBy`처럼 ...에 식별하기 위한 내용이 들어가도 된다.

    * COUNT : `count...By`
    
        * 반환타입 : `long`
        
    * EXISTS : `exists...By` 
    
        * 반환타입 : `boolean`   

    * 삭제 : `delete...By`, `remove...By`
    
        * 반환타입 : `long`
        
    * DISTINCT : `findDistinct`, `findMemberDistinctBy`
    
    * LIMIT : `findFirst3`, `findFirst`, `findTop`, `findTop3` 

* 참고사항

    * `메소드 이름으로 쿼리 생성 방식`은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다. 
    
    * 그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
    
    * 그리고 해당 기능은 검색 조건으로 사용되는 컬럼이 2개 이하일 때 사용하기 좋다.

#### 2) JPA NamedQuery - 사용 X

* JPA NamedQuery

    * 스프링 데이터 JPA는 메소드 이름으로 JPA의 `NamedQuery`를 호출하는 기능을 제공한다.

    * `NamedQuery`는 쿼리에 이름을 부여해서 사용하는 방법이다.
    
    * `NamedQuery`**는 실무에서 거의 사용되지 않는다.**

* `@NamedQuery` 애노테이션으로 Named 쿼리 정의

     ```java
    @Entity
    @NamedQuery(
            name="Member.findByUsername",
            query="select m from Member m where m.username = :username")
    public class Member {
        //...
    }
     ```

* 스프링 데이터 JPA로 Named 쿼리 호출

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> { //** 제네릭에 있는 Member 엔티티
        @Query(name = "Member.findByUsername") // 생략 가능
        List<Member> findByUsername(@Param("username") String username);
    }
    ```

    * 스프링 데이터 JPA는 선언한 `엔티티 이름 + .(점) + 메서드 이름`으로 Named 쿼리를 찾아서 실행한다.
    
        * 만약 실행할 Named 쿼리가 없으면 `메소드 이름으로 쿼리 생성` 방식을 사용한다.
        
        * 필요하면 전략을 변경할 수 있지만 권장하지 않는다.
    
    * `@Param`는 이름 기반 파라미터를 바인딩 할 때 사용하는 애노테이션이다.

#### 3) @Query, 리포지토리 메소드에 쿼리 정의하기

* 리포지토리 메소드에 `@Query`를 사용하여 직접 쿼리를 정의 할 수 있다. 

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
    
        @Query("select m from Member m where m.username = :username and m.age = :age")
        List<Member> findUser(@Param("username") String username, @Param("age") int age);
    
    }
    ```

* 실행할 메서드에 정적 쿼리를 직접 작성하므로 `이름 없는 Named 쿼리`라 할 수 있다.

* `JPA Named 쿼리`처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있다는 장점이 있다. 

#### 4) @Query, 값과 DTO 조회하기

* 단순히 값 하나를 조회

     ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        @Query("select m.username from Member m")
        List<String> findUsernameList();
    }
     ```

* DTO로 직접 조회

    * `DTO(Data Transfer Object)` : 계층 간 데이터 교환을 위한 객체이다.

        * ① DTO를 만든다.
        
            ```java
            @Data
            public class MemberDTO {
            
                private Long id;
                
                private String username;
                
                private String teamName;
                
                public MemberDTO(Long id, String username, String teamName) {
                    this.id = id;
                    this.username = username;
                    this.teamName = teamName;
                }
            }
            ```
           
        * ② DTO로 직접 조회하려면 JPA의 new 명령어를 사용해야 한다. 그리고 아래 코드처럼 DTO에 매칭되는 생성자가 있어야 한다.
        
            ```java
            public interface MemberRepository extends JpaRepository<Member, Long> {           
                @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
                List<MemberDTO> findMemberDto();
            }
            ```
           
#### 5) 파라미터 바인딩

* 파라미터 바인딩

   * 스프링 데이터 JPA는 `위치 기반 파라미터 바인딩`과 `이름 기반 파라미터 바인딩`을 모두 지원한다.
    
     ```java
     select m from Member m where m.username = ?0 //위치 기반 
     select m from Member m where m.username = :name //이름 기반
     ```
  
        * 코드 가독성과 유지보수를 위해 `이름 기반 파라미터 바인딩`을 사용하자.

* 컬렉션 파라미터 바인딩

   * `Collection` 타입으로 in 절을 지원한다.
   
     ```java
     @Query("select m from Member m where m.username in :names")
     List<Member> findByNames(@Param("names") Collection<String> names);
     ```

#### 6) 반환 타입

* 스프링 데이터 JPA는 `유연한 반환 타입`을 지원한다.

   * 스프링 데이터 JPA는 `유연한 반환 타입`을 지원한다.
   
   * 결과가 한 건 이상이면 `컬렉션 인터페이스`를 사용하고, 단건이면 반환 타입을 지정한다.
   
        ```java
        List<Member> findListByUsername(String username);           // 컬렉션
        Member findMemberByUsername(String username);               // 단건
        Optional<Member> findOptionalByUsername(String username);   // 단건 Optional
        ```
     
* 조회 결과가 많거나 없으면 어떻게 될까?

    * ① 컬렉션 조회인 경우
    
        * 조회 결과가 없으면 빈 컬렉션을 반환
    
    * ② 단건 조회인 경우
    
        * 조회 결과가 없으면 `null`을 반환
        
        * 조회 결과가 2건 이상이면 `javax.persistence.NonUniqueResultException` 예외가 발생한다.
     
#### 7) 순수 JPA 페이징과 정렬

* JPA에서 페이징을 어떻게 할 것인가?

   * 다음 조건으로 페이징과 정렬을 사용하는 코드를 보자.
   
      * 검색 조건 : 나이가 10살
      
      * 정렬 조건 : 이름으로 내림차순
      
      * 페이징 조건 : 첫 번째 페이지, 페이지 당 보여줄 데이터는 3건

        ```java
        @Repository
        public class MemberJpaRepository {
            // 몇 번째(offset) 부터 시작해서 몇 개(limit)를 조회한다.
            public List<Member> findByPage(int age, int offset, int limit){ 
                return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
                        .setParameter("age", age)
                        .setFirstResult(offset)
                        .setMaxResults(limit)
                        .getResultList();
            }
        
            public long totalCount(int age){
                return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
                        .setParameter("age", age)
                        .getSingleResult();
        
            }
        }
        ```
        
#### 8) 스프링 데이터 JPA 페이징과 정렬

* 스프링 데이터 JPA는 쿼리 메소드에 `페이징과 정렬` 기능을 사용 할 수 있도록 2가지 파라미터를 제공한다.

    * `org.springframework.data.domain.Sort` : 정렬 기능
    
    * `org.springframework.data.domain.Pageable` : 페이징 기능 (내부에 Sort 포함)

* 파라미터에 Pageable을 사용하면 다음과 같은 `특별한 반환 타입`을 사용 할 수 있다. 

    * `org.springframework.data.domain.Page` : 추가 count 쿼리를 호출하는 페이징 결과
    
    * `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 다음 페이지만 확인 가능하다. (내부적으로 limit + 1 조회)
    
    * `List` (자바 컬렉션): 추가 count 쿼리 없이 결과만 반환한다.
    
* 페이징과 정렬 사용 예제 - 정의

    ```java
     public interface MemberRepository extends Repository<Member, Long> {
        Page<Member> findByAge(int age, Pageable pageable); //count 쿼리 사용
    
        Slice<Member> findByAge(int age, Pageable pageable); //count 쿼리 사용 안 함
    
        List<Member> findByAge(int age, Pageable pageable); //count 쿼리 사용 안 함
    
        List<Member> findByAge(int age, Sort sort);
     }
    ```
  
* 페이징과 정렬 사용 예제 - 실행

    ```java
    @Test
    public void page() throws Exception {
        //given
        memberRepository.save(new Member("member1", 10)); 
        memberRepository.save(new Member("member2", 10)); 
        memberRepository.save(new Member("member3", 10)); 
        memberRepository.save(new Member("member4", 10)); 
        memberRepository.save(new Member("member5", 10));

        //when
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
        Page<Member> page = memberRepository.findByAge(10, pageRequest);

        //then
        List<Member> content = page.getContent();
        assertThat(content.size()).isEqualTo(3);
        assertThat(page.getTotalElements()).isEqualTo(5);
        assertThat(page.getNumber()).isEqualTo(0);
        assertThat(page.getTotalPages()).isEqualTo(2);
        assertThat(page.isFirst()).isTrue();
        assertThat(page.hasNext()).isTrue();
    }
    ```
  
    * 두 번째 파라미터로 받은 `Pagable`은 인터페이스이며 페이징 처리에 필요한 정보를 제공한다.
    
    * 그리고 실제 사용할 때는 해당 인터페이스를 구현한 `PageRequest` 객체를 사용한다.
    
    * `PageRequest.of()`의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를 입력한다.
     
    * 여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다. 참고로 페이지는 0 부터 시작한다.

* Page 인터페이스

    ```java
    public interface Page<T> extends Slice<T> {
      int getTotalPages(); //전체 페이지 수
  
      long getTotalElements(); //전체 데이터 수
  
      <U> Page<U> map(Function<? super T, ? extends U> converter); //변환기
    }
    ```

* Slice 인터페이스

    ```java
    public interface Slice<T> extends Streamable<T> {
        int getNumber();                  // 현재 페이지
        int getSize();                    // 페이지 크기
        int getNumberOfElements();        // 현재 페이지에 나올 데이터 수
        List<T> getContent();             // 조회된 데이터
        boolean hasContent();             // 조회된 데이터 존재 여부
        Sort getSort();                   // 정렬 정보
        boolean isFirst();                // 현재 페이지가 첫 페이지 인지 여부
        boolean isLast();                 // 현재 페이지가 마지막 페이지 인지 여부
        boolean hasNext();                // 다음 페이지 여부
        boolean hasPrevious();            // 이전 페이지 여부
        Pageable getPageable();           // 페이지 요청 정보
        Pageable nextPageable();          // 다음 페이지 객체
        Pageable previousPageable();      // 이전 페이지 객체
        <U> Slice<U> map(Function<? super T, ? extends U> converter); //변환기
    }
    ```
  
* 다음과 같은 추가적인 내용은 해당 강좌를 참고하자.

    * count 쿼리를 분리
    
    * Top, First 사용
    
    * 페이지를 유지하면서 엔티티를 DTO로 변환하기

#### 9) 벌크성 수정 쿼리

* JPA를 사용한 벌크성 수정 쿼리

    ```java
    public int bulkAgePlus(int age){
        return em.createQuery(
                "update Member m set m.age = m.age + 1" +
                        " where m.age >= :age")
                .setParameter("age", age)
                .executeUpdate();
    }
    ```

* 스프링 데이터 JPA를 사용한 벌크성 수정 쿼리

    ```java
    @Modifying
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
    ```
  
    * 벌크성 수정, 삭제 쿼리는 @Modifying 애노테이션을 사용해야 한다.
    
        * 사용하지 않으면 QueryExecutionRequestException 예외가 발생한다.
    
    * 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트를 초기화하려면 `@Modifying(clearAutomatically = true)`로 설정한다.
    
        * 이 옵션 없이 회원을 findById로 다시 조회하면 영속성 컨텍스트에 과거 값이 남아서 문제가 될 수 있다.
         
        * 만약 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화 하자.
        
    * 벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 있는 엔티티 상태가 달라질 수 있다.

#### 10) @EntityGraph

* member와 team은 지연 로딩 관계이다. 

* 따라서 team의 데이터를 조회할 때 마다 쿼리가 실행된다. (N+1 문제 발생함)

* 엔티티를 조회할 때 연관된 엔티티들을 함께 조회하는 방법

    * JPQL에서 페치 조인을 사용하면 된다.

        ```java
        @Query("select m from Member m left join fetch m.team")
        List<Member> findMemberFetchJoin();
        ```
      
    * 엔티티 그래프 기능은 엔티티를 조회하는 시점에 연관된 엔티티를 함께 조회하는 기능이다.
    
        * EntityGraph는 사실상 페치 조인(FETCH JOIN)의 간편 버전
        
        * LEFT OUTER JOIN 사용
    
       ```java
       //공통 메서드 오버라이드
       @Override
       @EntityGraph(attributePaths = {"team"}) List<Member> findAll();
       
       //JPQL + 엔티티 그래프
       @EntityGraph(attributePaths = {"team"})
       @Query("select m from Member m") List<Member> findMemberEntityGraph();
       
       //메소드 이름으로 쿼리를 생성하는 경우에 사용하면 특히 편리하다.
       @EntityGraph(attributePaths = {"team"})
       List<Member> findByUsername(@Param("username") String username);
       ``` 

    * @NamedEntityGraph
    
        * Named 엔티티 그래프는 @NamedEntityGraph로 정의한다.
        
            * name : 엔티티 그래프의 이름을 정의한다.
            
            * attributeNodes : 함께 조회할 속성을 선택한다. 이때, @NamedAttributeNode를 사용하고 그 값으로 함께 조회할 속성을 선택하면 된다.
        
           ```java
           @NamedEntityGraph(name = "Member.all", attributeNodes = @NamedAttributeNode("team"))
           @Entity
           public class Member {}
          
           @EntityGraph("Member.all") 
           @Query("select m from Member m")
           List<Member> findMemberEntityGraph();
           ```         
        
        * Named 엔티티 그래프는 잘 사용하지 않는 것 같음.
        
#### 11) JPA Hint & Lock

* JPA Hint

    * `JPA Hint`는 SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트다.
    
    * JPA 쿼리 힌트를 사용하는 방법은 다음과 같다.
    
       ```java
       @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
       Member findReadOnlyByUsername(String username);
       ```
          * readOnly로 설정하면 변경 감지 기능이 동작하지 않는다.
          
* Lock

    * 쿼리 시 락을 걸려면 `org.springframework.data.jpa.repository.Lock` 애노테이션을 사용하면 된다.
    
    * JPA가 제공하는 락은 자바 ORM 표준 JPA 프로그래밍 책에서 16.1 절을 참고하자.
    
       ```java
       @Lock(LockModeType.PESSIMISTIC_WRITE)
       List<Member> findByUsername(String name);
       ```

## 5. 확장 기능

#### 1) 사용자 정의 리포지토리 구현

* 사용자 정의 리포지토리를 구현하는 이유

    * 스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동으로 생성한다.
 
    * 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하면 구현해야 하는 기능이 너무 많다.
 
    * 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면 어떻게 해야될까?

        * JPA 직접 사용(`EntityManager`)
        * 스프링 JDBC Template 사용 
        * MyBatis 사용
        * 데이터베이스 커넥션 직접 사용 등등... 
        * Querydsl 사용

* 사용자 정의 리포지토리를 구현하는 방법

    * 사용자 정의 인터페이스를 작성
    
        ```java
        public interface MemberRepositoryCustom {
            List<Member> findMemberCustom();
        }
        ```
      
    * 사용자 정의 인터페이스를 구현한 클래스 작성
    
        * 클래스 이름을 짓는 규칙은 `리포지토리 인터페이스 이름 + Impl` 이다.
        
        * 이렇게 하면 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록한다.
        
        * 스프링 데이터 2.x 부터는 클래스 이름을 짓는 규칙을 `사용자 정의 인터페이스 명 + Impl`로 하는 것이 가능하다.
    
            * 예를 들어 아래 예제의 `MemberRepositoryImpl` 대신에 `MemberRepositoryCustomImpl` 같이 작성해도 된다.
            
            * 해당 방식이 사용자 정의 인터페이스 이름과 구현 클래스 이름이 비슷하므로 더 직관적이므로 해당 방식을 사용하는 것을 권장한다.

        ```java
        @RequiredArgsConstructor
        public class MemberRepositoryImpl implements MemberRepositoryCustom{
        
            // JPA 직접 사용
            private final EntityManager em;
        
            @Override
            public List<Member> findMemberCustom() {
                return em.createQuery("select m from Member m")
                        .getResultList();
            }
        }
        ```

    * 리포지토리 인터페이스에서 사용자 정의 인터페이스를 상속 받는다.
    
        ```java
        @RequiredArgsConstructor
        public class MemberRepositoryImpl implements MemberRepositoryCustom{
        
            // JPA 직접 사용
            private final EntityManager em;
        
            @Override
            public List<Member> findMemberCustom() {
                return em.createQuery("select m from Member m")
                        .getResultList();
            }
        }
        ```
* Impl 대신 다른 이름으로 변경하고 싶으면 해당 강좌를 참고하자.

    * 왠만하면 관례를 따르자.

#### 2) Auditing

* Auditing

    * `Auditing`는 엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하는 기능이다.

        * 등록일
        
        * 수정일 
        
        * 등록자 
        
        * 수정자
 
* Auditing 구현
  
    * 순수 JPA 사용
    
        * 등록일, 수정일을 적용한 클래스 작성
    
            ```java
            @MappedSuperclass
            @Getter
            public class JpaBaseEntity {
            
                @Column(updatable = false)
                private LocalDateTime createdDate;
            
                private LocalDateTime updatedDate;
            
                // persist() 하기 전에 이벤트가 발생
                @PrePersist
                public void prePersist(){
                    LocalDateTime now = LocalDateTime.now();
                    createdDate = now;
                    updatedDate = now;
                }
            
                // update() 하기 전에 이벤트가 발생
                @PreUpdate
                public void preUpdate(){
                    updatedDate = LocalDateTime.now();
                }
                
            }
            ```

        * 앞서 작성한 클래스를 상속 받는다.
        
            ```java
            public class Member extends JpaBaseEntity {}
            ```   

    * 스프링 데이터 JPA 사용
    
        * 설정
        
            * `@EnableJpaAuditing`를 스프링 부트 설정 클래스에 적용 해야 함
            
                 ```java
                 @EnableJpaAuditing
                 @SpringBootApplication
                 public class DataJpaApplication {
                 
                 	public static void main(String[] args) {
                 		SpringApplication.run(DataJpaApplication.class, args);
                 	}
                 
                 }
                 ```
                     
            * `@EntityListeners(AuditingEntityListener.class)`를 적용한다.
    
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
                
                    // 등록자
                    @CreatedBy
                    @Column(updatable = false)
                    private String createdBy;
                
                    // 수정자
                    @LastModifiedBy
                    private String lastModifiedBy;
                
                }
                ```

            * 등록자, 수정자를 처리해주는 `AuditorAware` 스프링 빈을 등록한다.
            
                ```java
                @Bean
                public AuditorAware<String> auditorProvider(){
                    // 랜덤 UUID 생성
                    return () -> Optional.of(UUID.randomUUID().toString());
                }
                ```
                
                * 실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받음

        * 앞서 작성한 클래스를 상속 받는다.
        
            ```java
            public class Member extends BaseEntity {}
            ```

    * @EntityListeners(AuditingEntityListener.class)를 생략하고 스프링 데이터 JPA가 제공하는 이벤트를 엔티티 전체에 적용하는 방법이 있는데 해당 강좌를 참고하자.
    
#### 3) Web 확장

* `도메인 클래스 컨버터`는 HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩 해준다.
  
    * 도메인 클래스 컨버터 사용 예시

        ```java
        @RestController
        @RequiredArgsConstructor
        public class MemberController {
        
            private final MemberRepository memberRepository;
        
            // 도메인 클래스 컨버터 사용 전
            @GetMapping("/members/{id}")
            public String findMember(@PathVariable("id") Long id){
                Member member = memberRepository.findById(id).get();
                return member.getUsername();
            }
        
            // 도메인 클래스 컨버터 사용 후
            @GetMapping("/members2/{id}")
            public String findMember2(@PathVariable("id") Member member){
                return member.getUsername();
            }
        
        
            @PostConstruct
            public void init(){
                memberRepository.save(new Member("userA"));
            }
        
        }
        ```
      
        * 주의: 도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, 이 엔티티는 단순 조회용으로만 사용해야 한다. 
        * (트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영되지 않는다.)

* 페이징과 정렬
  
    * 페이징과 정렬 예제

        ```java
        @RestController
        @RequiredArgsConstructor
        public class MemberController {
        
            private final MemberRepository memberRepository;
                  
            @GetMapping("/members")
            public Page<Member> list(Pageable pageable){
                Page<Member> page = memberRepository.findAll(pageable);
                return page;
            }
                  
        }
        ```
      
        * 파라미터로 `Pageable`을 받을 수 있다.
        
        * `Pageable`은 다음 요청 파라미터 정보로 만들어진다.
        
            * [예시] /members?page=0&size=3&sort=id,desc&sort=username,desc   
                     
            * `page` : 현재 페이지, 0부터 시작한다.
            
            * `size` : 한 페이지에 노출할 데이터 건수
            
            * `sort` : 정렬 조건을 정의한다.
        
        * `Pageable`은 인터페이스이며 실제는 `PageRequest` 객체가 생성된다.
        
    * Pageable 기본값 변경

        * 글로벌 설정: 스프링 부트
        
            ```java
            spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/ 
            spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/
            ```
          
        * 개별 설정: `@PageableDefault` 애노테이션을 사용
        
            ```java
            @GetMapping("/members")
            public Page<Member> list(@PageableDefault(size = 5, sort = "username") Pageable pageable){
                Page<Member> page = memberRepository.findAll(pageable);
                return page;
            }
            ```
          
    * 접두사        

        * 사용 해야 할 페이징 정보가 둘 이상이면 접두사로 구분한다.
        
        * `@Qualifier`에 접두사명을 추가한다. "{접두사명}_xxx”
        
        * 예시: `/members?member_page=0&order_page=1`

            ```java
            public String list(
                    @Qualifier("member") Pageable memberPageable, 
                    @Qualifier("order") Pageable orderPageable, ...
            ```
                  
    * Page 내용을 DTO로 변환하기       

        * 엔티티를 API로 노출하면 다양한 문제가 발생한다. 그래서 엔티티를 꼭 DTO로 변환해서 반환해야 한다.
         
        * Page는 `map()`을 지원해서 내부 데이터를 다른 것으로 변경할 수 있다.

            ```java
            @Data
            public class MemberDto {          
                private Long id;
            
                private String username;
                
                public MemberDto(Member member){
                    this.id = member.getId();
                    this.username = member.getUsername();
                }
            }
            ```
                  
            ```java
            @GetMapping("/members")
            public Page<MemberDto> list(@PageableDefault(size = 5) Pageable pageable){
                return memberRepository.findAll(pageable)
                        .map(MemberDto::new);
            }
            ```
          
    * Page를 1부터 시작하기       

        * 스프링 데이터는 Page를 0부터 시작한다. 만약 1부터 시작하려면 어떻게 해야 될까?
        
            * 첫 번째 방법
                * Pageable, Page를 파라미터와 응답 값으로 사용하지 않고, 직접 클래스를 만들어서 처리한다. 
                * 그리고 직접 PageRequest(Pageable 구현체)를 생성해서 리포지토리에 넘긴다. 
                * 물론 응답 값도 Page 대신에 직접 만들어서 제공해야 한다.
              
            * 두 번째 방법
                * spring.data.web.pageable.one-indexed-parameters 를 true로 설정한다. 
                * 그런데 이 방법은 web에서 page 파라미터를 -1 처리 할 뿐이다.
                * 따라서 응답값인 Page 에 모두 0 페이지 인덱스를 사용하는 한계가 있다.

        * 가급적 권장하는 것은 페이지 인덱스를 0 부터 처리하자.
        
## 6. 스프링 데이터 JPA 분석

#### 1) 스프링 데이터 JPA 구현체 분석

* 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체

    * `org.springframework.data.jpa.repository.support.SimpleJpaRepository`
    
    * 위에 있는 구현체의 일부 코드를 살펴보자.
    
        * @Repository 적용
        
            * 컴포넌트 스캔의 대상이 되어 스프링 빈으로 등록된다.
            
            * JPA 예외를 스프링이 추상화한 예외로 변환한다.
            
        * @Transactional 트랜잭션 적용
            
            * JPA의 모든 변경은 트랜잭션 안에서 이루어져야 한다.
            
            * 스프링 데이터 JPA는 변경(등록, 수정, 삭제) 메서드에 @Transactional로 트랜잭션 처리가 되어 있다.
            
            * 서비스 계층에서 트랜잭션을 시작하지 않으면 리파지토리에서 트랜잭션을 시작한다.
            
            * 서비스 계층에서 트랜잭션을 시작하면 리파지토리는 해당 트랜잭션을 전파 받아서 사용한다.
            
            * 그래서 스프링 데이터 JPA를 사용할 때 트랜잭션이 없어도 데이터 등록, 변경이 가능했던 것이다. (사실은 트랜잭션이 리포지토리 계층에 이미 걸려있는 것임)

        * `@Transactional(readOnly = true)`
            
            * 데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서 `readOnly = true` 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음

        * `save()` 메서드
            
            * 새로운 엔티티면 저장(`persist`)하고 새로운 엔티티가 아니면 병합(`merge`)한다.

#### 2) 새로운 엔티티를 구별하는 방법

* 새로운 엔티티를 판단하는 기본 전략

    * 식별자가 객체일 때는 `null`로 판단 
    
    * 식별자가 자바 기본 타입일 때는 `0`으로 판단
    
    * `Persistable` 인터페이스를 구현해서 판단 로직 변경 가능
    
* Persistable 구현

    * [참고] JPA 식별자 생성 전략이 `@GenerateValue`면 save() 호출 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 정상 동작한다. 
    
    * 그런데 JPA 식별자 생성 전략이 @Id 만 사용해서 직접 할당하게 되면 이미 식별자 값이 있는 상태로 save()를 호출한다. 
    
    * 따라서 이 경우에는 merge()가 호출된다. merge()는 우선 DB를 호출해서 값을 확인하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율적이다.
    
    * 따라서 Persistable를 사용해서 새로운 엔티티 확인 여부를 직접 구현하는 것이 효과적이다.
    
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
            return null;
        }
    
        @Override
        public boolean isNew() {
            return createdDate == null;
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


