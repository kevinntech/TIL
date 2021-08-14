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
            
* 쿼리 파라미터 로그 남기기

    * `org.hibernate.type` 옵션은 SQL 실행 파라미터를 로그로 남긴다.
    
    * 외부 라이브러리 사용
    
        * 스프링 부트를 사용하면 해당 라이브러리만 추가하면 된다.
        
            * `implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'`
    
* 자주 사용되는 설정

    * `spring.jpa.hibernate.ddl-auto` : DDL 자동 생성 기능
    
        ```
        spring.jpa.hibernate.ddl-auto = create
        ```
        
    * `spring.jpa.properties.hibernate.show_sql` : 하이버네이트가 실행한 SQL을 System.out에 출력한다.

        ```
        spring.jpa.properties.hibernate.show_sql=true
        ```
      
    * `spring.jpa.properties.hibernate.format_sql` : 하이버네이트가 실행한 SQL을 가독성 있게 표현한다.

        ```
        spring.jpa.properties.hibernate.format_sql=true
        ```
      
    * `logging.level.org.hibernate.SQL` : 하이버네이트가 실행한 SQL을 로거를 통해 출력한다.

        ```
        logging.level.org.hibernate.SQL=debug
        ```

        * 모든 로그 출력은 로거를 통해 남겨야 한다.

    * 쿼리 파라미터를 로그로 출력하기
    
        * ① `logging.level.org.hibernate.type.descriptor.sql` : 쿼리 파라미터를 로그로 출력한다.
    
            ```
            logging.level.org.hibernate.type.descriptor.sql=trace
            ```

            * 하이버네이트가 실행한 SQL에서 물음표(`?`)로 표기된 부분을 쿼리 파라미터라고 한다.

        * ② 외부 라이브러리 사용하기 (`p6spy`)
    
            * 스프링 부트를 사용하면 이 라이브러리만 추가하면 된다.
            
                ```
                implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'
                ```

                * 쿼리 파라미터를 로그로 남기는 외부 라이브러리는 시스템 자원을 사용하므로, 개발 단계에서는 편하게 사용해도 된다. 
                  
                * 하지만 운영 시스템에 적용하려면 꼭 성능 테스트를 하고 사용하는 것이 좋다.

            * [참고] https://github.com/gavlyukovskiy/spring-boot-data-source-decorator
    
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

            * `@Repository`를 생략 할 수 있다.
            
                * `JpaRepository` 인터페이스를 상속 받으면 컴포넌트 스캔의 대상이 된다.
                
                * JPA 예외를 스프링 예외로 변환한다. 

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
        
            * 정렬(`Sort`)이나 페이징(`Pageable`) 조건을 파라미터로 제공 할 수 있다.

## 4. 쿼리 메소드 기능

* 스프링 데이터 JPA가 제공하는 `쿼리 메소드 기능` 3가지

    * ① 메소드 이름으로 쿼리 생성
    
    * ② 메소드 이름으로 JPA NamedQuery 호출
    
    * ③ `@Query`를 사용해서 리포지토리 메소드에 직접 쿼리를 정의

#### 1) 메소드 이름으로 쿼리 생성

* 스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행한다.

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
    
      List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
    
    }
    ```

    * `Username` : username이 ~ 와 같으면
    
    * `AgeGreaterThan` : age가 ~ 보다 크면
    
* 위의 메소드는 다음과 같은 JPQL을 생성한다.

    ```java
    select m 
    from Member m 
    where m.username = :username and m.age > :age
    ```

* 메소드 이름으로 사용 할 수 있는 키워드는 다음과 같다.

    * [스프링 데이터 JPA 공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods "") 참고

* 스프링 데이터 JPA가 제공하는 쿼리 메소드 기능

    * 조회 : `find...By` , `read...By` , `query...By`, `get...By`
    
        * Ex) `findHelloBy`처럼 `...`에 식별하기 위한 내용이 들어가도 된다.
    
            * `By` 키워드 다음에는 where 조건을 지정한다.
    
                * `findHelloBy`처럼 where 조건을 지정하지 않으면 전체 조회가 된다.
    
    * AND : `And`
    
        * where 절에 and가 추가된다.
        
    * OR : `Or`
    
        * where 절에 or가 추가된다.

    * COUNT : `count...By`
    
        * 반환 타입 : `long`
        
    * EXISTS : `exists...By` 
    
        * 반환 타입 : `boolean`   

    * 삭제 : `delete...By`, `remove...By`
    
        * 반환 타입 : `long`
        
    * DISTINCT : `findDistinct`, `findMemberDistinctBy`
    
    * LIMIT : `findFirst3`, `findFirst`, `findTop`, `findTop3` 

* 참고사항

    * `메소드 이름으로 쿼리 생성 방식`은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다. 
    
    * 그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
    
    * 그리고 **해당 기능은 검색 조건으로 사용되는 컬럼이 2개 이하일 때 사용하기 좋다.**

#### 2) JPA NamedQuery - 사용 X

* JPA NamedQuery

    * `NamedQuery`는 쿼리에 이름을 부여해서 호출하는 방법이다.
    
        * **실무에서 거의 사용되지 않는다.**

* 실습

    * ① `@NamedQuery` 애노테이션으로 Named 쿼리를 정의한다.
    
         ```java
        @Entity
        @NamedQuery(
                name="Member.findByUsername",
                query="select m from Member m where m.username = :username")
        public class Member {
            //...
        }
         ```
    
    * ② 스프링 데이터 JPA로 Named 쿼리 호출한다.
    
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> { //** 제네릭에 있는 Member 엔티티
            @Query(name = "Member.findByUsername") // 생략 가능
            List<Member> findByUsername(@Param("username") String username);
        }
        ```

        * 쿼리 메소드 기능의 실행 순서

            * ① 스프링 데이터 JPA는 선언한 `엔티티 이름 + .(점) + 메서드 이름`으로 Named 쿼리를 찾아서 실행한다.
            
            * ② 만약 실행할 Named 쿼리가 없으면 `메소드 이름으로 쿼리 생성` 방식을 사용한다.
                
                * 필요하면 전략을 변경할 수 있지만 권장하지 않는다.
                
        * `Named 쿼리`는 애플리케이션을 시작하는 시점에 문법 오류를 발견 할 수 있다.
        
        * `@Param` : 이름 기반 파라미터를 바인딩 할 때 사용한다.

#### 3) @Query, 리포지토리 메소드에 직접 쿼리 정의하기

* `@Query`는 리포지토리 메소드에 직접 쿼리를 정의할 때 사용한다.

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
    
        @Query("select m from Member m where m.username = :username and m.age = :age")
        List<Member> findUser(@Param("username") String username, @Param("age") int age);
    
    }
    ```

    * 실행할 메서드에 정적 쿼리를 직접 작성하므로 `이름 없는 Named 쿼리`라 할 수 있다.
    
    * `@Query`는 애플리케이션을 시작하는 시점에 문법 오류를 발견 할 수 있다는 장점이 있다. 

#### 4) @Query, 값과 DTO 조회하기

* 단순히 값 하나를 조회

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        @Query("select m.username from Member m")
        List<String> findUsernameList();
    }
    ```

    * 위의 코드는 엔티티(`m`)가 아닌 값 하나(`m.username`)를 조회한다. 

* DTO로 직접 조회

    * `DTO(Data Transfer Object)` : 계층 간 데이터 교환을 위한 객체이다.

        * ① DTO를 작성한다.
        
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
           
        * ② DTO로 직접 조회하려면 `new` 명령어를 사용한다. 그리고 DTO에 일치하는 생성자가 있어야 한다.
        
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
        // 쿼리 메소드 정의
        @Query("select m from Member m where m.username in :names")
        List<Member> findByNames(@Param("names") Collection<String> names);
        ```
        
        ```java
        // 테스트 코드
        @Test
        public void findByNames(){
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

* 스프링 데이터 JPA는 `유연한 반환 타입`을 지원한다.
   
   * 결과가 한 건 이상이면 `컬렉션 인터페이스`를 사용하고, 단건이면 반환 타입을 지정한다.
   
        ```java
        List<Member> findListByUsername(String username);           // 컬렉션
        Member findMemberByUsername(String username);               // 단건
        Optional<Member> findOptionalByUsername(String username);   // 단건 Optional
        ```
     
* **조회 결과가 많거나 없으면 어떻게 될까?**

    * ① 컬렉션 조회인 경우
    
        * 조회 결과가 없으면 비어있는 컬렉션을 반환한다.
        
            * `null` 체크를 하지 않아도 됨
    
    * ② 단건 조회인 경우
    
        * 조회 결과가 없으면 `null`을 반환한다.
        
            * 조회 결과가 `null` 일 수도 있다면 반환 타입으로 단건 Optional(`Optional<Member>`)를 사용하자.
        
        * 조회 결과가 2건 이상이면 `javax.persistence.NonUniqueResultException` 예외가 발생한다.
     
#### 7) 순수 JPA 페이징과 정렬

* JPA에서 페이징을 어떻게 할 것인가?

   * 다음 조건으로 페이징과 정렬을 사용하는 코드를 보자.
   
      * **검색 조건** : 나이가 10살
      
      * **정렬 조건** : 이름으로 내림차순
      
      * **페이징 조건** : 첫 번째 페이지, 페이지 당 보여줄 데이터는 3건

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
        
            // 전체 개수
            public long totalCount(int age){
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

#### 8) 스프링 데이터 JPA 페이징과 정렬

* `스프링 데이터 JPA`는 쿼리 메소드에 `페이징`과 `정렬` 기능을 사용 할 수 있도록 **2 가지 파라미터**를 제공한다.

    * ① `org.springframework.data.domain.Sort` : 정렬 기능

        ```java
        Sort sort = Sort.by(Sort.Direction.ASC, "id");
      
        String sortProperty = sort.toString().contains("id") ? "id" : "updatedDate";
        ```

    * ② `org.springframework.data.domain.Pageable` : 페이징 기능 (내부에 Sort 포함)

* 파라미터에 `Pageable`을 사용하면 다음과 같은 `특별한 반환 타입`을 사용 할 수 있다. 

    * ① `org.springframework.data.domain.Page` : 추가 count 쿼리를 호출하는 페이징 결과를 반환한다.
    
        * `Page` 인덱스는 0 부터 시작이다. 
    
    * ② `org.springframework.data.domain.Slice` : 추가 count 쿼리를 호출하지 않는 페이징 결과를 반환한다.
        
        * 다음 페이지 존재 여부를 확인 할 수 있다. (내부적으로 limit + 1 조회)
    
    * ③ `자바 컬렉션` (`List` ...) : 추가 count 쿼리 없이 결과만 반환한다.
    
        * 예를 들어, 데이터를 10개만 끊어서 가져올 때 사용 할 수 있다.
    
* 예시
  
    * 쿼리 메소드 정의하기

        ```java
        public interface MemberRepository extends Repository<Member, Long> {
            Page<Member> findByAge(int age, Pageable pageable); //count 쿼리 사용
            
            Slice<Member> findByAge(int age, Pageable pageable); //count 쿼리 사용 안 함
            
            List<Member> findByAge(int age, Pageable pageable); //count 쿼리 사용 안 함
            
            List<Member> findByAge(int age, Sort sort);
        }
        ```
  
    * 쿼리 메소드 사용하기
    
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
            // 0 페이지에서 3개를 가져온다. 그리고 username으로 내림차순 정렬한다.
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
      
        * `findByAge()`
        
            * 두 번째 파라미터 `Pageable`은 인터페이스이며 페이징 처리에 필요한 정보를 제공한다.
        
            * 그리고 파라미터 타입이 `Pageable`이면 해당 인터페이스를 구현한 `PageRequest` 객체를 전달한다.
        
        * `PageRequest` : Pageable 인터페이스의 구현체이다.
        
        * `PageRequest.of()`
        
            * **첫 번째 파라미터**에는 **현재 페이지**를, **두 번째 파라미터**에는 **조회할 데이터 수**를 전달한다.
         
            * 여기에 추가로 **정렬 정보**도 파라미터로 전달 할 수 있다. 참고로 페이지는 0 부터 시작한다.
    
                * `PageRequest.of(int page, int size, Sort sort)`

* `Page` 인터페이스

    * `Page`를 사용하면 전체 페이지 수를 알아내기 위해서 추가적인 카운트 쿼리가 실행된다. 
    
        * 기본적으로 카운트 쿼리는 실제로 실행되는 쿼리에서 파생된다.
	
            ```java
            public interface Page<T> extends Slice<T> {
              int getTotalPages(); //전체 페이지 수
          
              long getTotalElements(); //전체 데이터 수
          
              <U> Page<U> map(Function<? super T, ? extends U> converter); //변환기
            }
            ```

* `Slice` 인터페이스

    * `Slice`를 사용하면 추가 카운트 쿼리를 실행하지 않고 다음 페이지 존재 여부를 확인 할 수 있다. (내부적으로 `limit + 1`로 조회한다.)

        ```java
        public interface Slice<T> extends Streamable<T> {
            int getNumber();                  // 현재 페이지
            int getSize();                    // 페이지 크기 (한 페이지에 표시할 데이터 수)
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

    * 더보기 기능을 구현할 때, 사용 할 수 있다.

* 추가적인 내용

    * count 쿼리를 분리하기
    
        * count 쿼리에는 `left join` 조인을 하는 것이 의미 없기 때문에 이러한 경우에 사용 할 수 있다.
    
            ```java
            @Query(value = "select m from Member m left join m.team t",
                   countQuery = "select count(m) from Member m")
            Page<Member> findByAge(int age, Pageable pageable);
            ```

            * value 속성은 컨텐츠를 가져오는 쿼리를 작성하고 countQuery는 카운트 쿼리를 작성한다.

            * 정렬 조건이 복잡해지면 `Sort`로 해결하기 어렵다. 
            
                * 이러한 경우에는 `@Query`의 JPQL로 작성된 쿼리에 정렬 기준을 직접 지정하자.
    
    * 페이지를 유지하면서 엔티티를 DTO로 변환하기
    
        * API를 작성 할 때, 컨트롤러에서 엔티티를 바로 반환하면 안된다. 
        
        * 그 이유는 엔티티를 변경하면 API의 스펙이 변경되기 때문이다. (API 장애가 발생 할 수 있음)
         
        * 그래서 엔티티는 절대 외부에 노출 시키지 않고 DTO로 변환해서 반환해야 한다.  
    
            ```java
            Page<Member> page = memberRepository.findByAge(age, pageRequest);
            Page<MemberDto> toMap = page.map(m -> new MemberDto(m.getId(), m.getUsername(), null));
            ```
          
            * `page.map()` : Page 내의 요소를 다른 요소로 변환한다.  

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
    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
    ```
  
    * 벌크성 수정, 삭제 쿼리는 `@Modifying` 애노테이션을 사용해야 한다.
    
        * 사용하지 않으면 `QueryExecutionRequestException` 예외가 발생한다.
    
    * `@Modifying(clearAutomatically = true)`로 설정하면 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트를 초기화한다.
    
        * 해당 옵션을 사용하지 않고 회원을 `findByUsername()`로 다시 조회하면 영속성 컨텍스트에 과거 값이 남아서 문제가 될 수 있다.

            * **벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 있는 엔티티 상태가 달라질 수 있다.**

        * 만약 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화 하자.

* 벌크성 수정 쿼리에 대한 테스트 코드

    ```java
    @Test
    public void bulkUpdate(){
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
        int resultCount = memberRepository.bulkAgePlus(20); // 해당 JPQL 실행 시, 플러시 동작
        //em.clear(); // 순수 JPA 사용 시 해결방법
  
        List<Member> result = memberRepository.findByUsername("member5");
        Member member5 = result.get(0);
        System.out.println("member5 = " + member5);
    
        // then
        assertThat(resultCount).isEqualTo(3);
    }
    ```

#### 10) @EntityGraph

* `N+1` 문제가 발생하는 경우

    ```java
    @Test
    public void findMemberLazy(){
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
    
        teamRepository.save(teamA);
        teamRepository.save(teamB);
    
        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 10, teamA);
    
        memberRepository.save(member1);
        memberRepository.save(member2);
    
        em.flush();
        em.clear();
    
        /*  */
        List<Member> members = memberRepository.findAll();
    
        for (Member member : members) {
            System.out.println("member = " + member.getUsername());
            System.out.println("member.team = " + member.getTeam().getName());
        }
    }
    ```
        
    * Member와 연괸된 Team은 지연 로딩(LAZY)으로 설정되어 있다.
    
        * Member를 조회할 때, Team 필드에는 프록시 객체를 넣어둔다.
        
        * 그래서 `member.getTeam()`까지는 데이터베이스를 조회하지 않는다.
        
        * 그러나 `member.getTeam().getName()`를 하는 순간, 데이터베이스를 조회한다.
        
            * `Team`의 필드를 건드릴 때, 위와 같이 동작한다.
    
    * 위의 코드는 `N+1` 문제가 발생한다.

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
          
        * 그리고 team 필드를 사용 할 때 마다 추가적인 N개의 쿼리가 발생한다. (`N`)

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
          
* 엔티티를 조회할 때 연관된 엔티티들을 함께 조회하는 방법

    * `JPQL`의 `페치 조인`을 사용한다.

        ```java
        @Query("select m from Member m left join fetch m.team")
        List<Member> findMemberFetchJoin();
        ```
      
    * `@EntityGraph` : 엔티티를 조회하는 시점에 연관된 엔티티를 함께 조회하는 기능이다.
    
        * 사실상 페치 조인(FETCH JOIN)의 간편 버전이다.
        
        * 기본적으로 LEFT OUTER JOIN을 사용한다.
    
           ```java
           // 공통 메서드를 오버라이딩
           @Override
           @EntityGraph(attributePaths = {"team"})
           List<Member> findAll();
           
           // JPQL + 엔티티 그래프
           // 쿼리는 작성 했는데 페치 조인만 적용하고 싶은 경우에 엔티티 그래프를 사용 할 수 있다.
           @EntityGraph(attributePaths = {"team"})
           @Query("select m from Member m") 
           List<Member> findMemberEntityGraph();
           
           // 메소드 이름으로 쿼리를 생성하는 경우에 사용하면 특히 편리하다.
           // 회원 엔티티를 조회할 때, 연관된 팀 엔티티도 함께 조회한다.
           @EntityGraph(attributePaths = {"team"})
           List<Member> findEntityGraphByUsername(@Param("username") String username);
           ```
          
            * `attributePaths` : 쿼리 수행 시, 즉시 로딩(EAGER LOADING)으로 함께 조회할 필드명을 지정한다

    * `@NamedEntityGraph`
    
        * ① `@NamedEntityGraph`는 Named 엔티티 그래프를 정의할 때 사용한다.
                
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

        * ② `@EntityGraph`으로 Named 엔티티 그래프를 사용한다.
                
            ```java
            public interface MemberRepository extends JpaRepository<Member, Long> {
                @EntityGraph("Member.all") 
                List<Member> findMemberEntityGraph();  
            }
            ```
          
            * Named 엔티티 그래프는 잘 사용하지 않는 기능으로 보인다.
        
#### 11) JPA Hint & Lock

* JPA Hint

    * `JPA Hint`는 SQL 힌트가 아닌 JPA 구현체(하이버네이트...)에게 제공하는 힌트다.
    
    * JPA 쿼리 힌트를 사용하는 방법은 다음과 같다.
    
        ```java
        @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
        Member findReadOnlyByUsername(String username);
        ```
      
        * `readOnly`로 설정하면 변경 감지 기능이 동작하지 않는다.
        
        * 그리고 내부적으로 성능 최적화를 해서 스냅샷을 만들지 않는다.
        
            * 즉, 데이터를 변경하지 않고 화면에 출력하는 용도로만 사용한다.
            
            * 성능 테스트를 했을 때, 느리다면 해당 힌트를 적용하자. 무조건적으로 적용하는 것은 좋지 않다.
     
* @Lock

    * 쿼리 시 락을 걸려면 `org.springframework.data.jpa.repository.Lock` 애노테이션을 사용하면 된다.

       ```java
       @Lock(LockModeType.PESSIMISTIC_WRITE)
       List<Member> findLockByUsername(String username);
       ```
      
    * JPA가 제공하는 락은 `자바 ORM 표준 JPA 프로그래밍` 책에서 `16.1 절`을 참고하자.
    
        * 실시간 트래픽이 많은 서비스에서는 가급적 락을 걸면 안된다.

## 5. 확장 기능

#### 1) 사용자 정의 리포지토리 구현

* 사용자 정의 리포지토리를 구현하는 이유

    * 스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동으로 생성하기 때문에
 
    * 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하게 되면 구현해야 하는 기능이 너무 많다.
 
    * 다음과 같은 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면 `사용자 정의 인터페이스`를 작성한다.

        * ① JPA 직접 사용(`EntityManager`)
          
        * ② **스프링 JDBC Template 사용** 
          
        * ③ MyBatis 사용
          
        * ④ 데이터베이스 커넥션 직접 사용
          
        * ⑤ **Querydsl 사용**
    
            * 복잡한 동적 쿼리를 작성하는 경우

* 사용자 정의 리포지토리를 구현하는 방법

    * ① 사용자 정의 인터페이스를 작성한다.
    
        ```java
        public interface MemberRepositoryCustom {
            List<Member> findMemberCustom();
        }
        ```
      
    * ② 사용자 정의 인터페이스를 구현한 클래스를 작성한다.
    
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

        * 클래스 이름을 짓는 규칙은 `리포지토리 인터페이스명 + Impl` 이다.
    
        * 이렇게 하면 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록한다.
        
            * **스프링 데이터 2.x 부터**는 클래스 이름을 짓는 규칙을 `사용자 정의 인터페이스명 + Impl`로 하는 것이 가능하다.
    
                ```java
                @RequiredArgsConstructor
                public class MemberRepositoryCustomImpl implements MemberRepositoryCustom{
                    //...
                }
                ```
    
                * 예를 들어, `MemberRepositoryImpl` 대신에 `MemberRepositoryCustomImpl`과 같이 작성해도 된다.
                
                * 해당 방식이 사용자 정의 인터페이스 이름과 구현 클래스 이름이 비슷하므로 더 직관적이므로 해당 방식을 사용하는 것을 권장한다.

        * `@RequiredArgsConstructor` : final 필드 만을 가지고 생성자를 만들어준다.
        
            * 스프링은 생성자가 하나인 경우에 `@Autowired`가 없더라도 자동으로 의존성을 주입한다.
            
            * 그리고 스프링 데이터 JPA를 사용하면 `EntityManager`에 `@PersistenceContext` 대신에 `@Autowired`를 사용 할 수 있다.
            
            * 그래서 위의 코드처럼 생성자 주입으로 작성 할 수 있다.
        
    * ③ 사용자 정의 인터페이스를 상속 받는다.
    
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
        }
        ```
      
    * ④ 사용자 정의 메서드를 호출한다.
    
        ```java
        List<Member> result = memberRepository.findMemberCustom();
        ```

* Impl 대신 다른 이름으로 변경하고 싶으면 ? 
  
    * 해당 강좌를 참고하자.

    * 왠만하면 관례를 따르도록 하자.
    
* 참고 사항

    * 항상 사용자 정의 리포지토리가 필요한 것은 아니다. 
      
    * 그냥 임의의 리포지토리를 만들어도 된다. 
      
        * 예를 들어 MemberQueryRepository를 인터페이스가 아닌 클래스로 만들고 스프링 빈으로 등록해서 그냥 직접 사용해도 된다. 

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
          
            * 핵심 비즈니스 로직이 있는 리포지토리와 화면에 맞춘 통계 쿼리가 있는 리포지토리를 분리해야 한다. 

        * 물론 이 경우 스프링 데이터 JPA와는 아무런 관계 없이 별도로 동작한다. 

#### 2) Auditing

* `Auditing` : 엔티티를 변경하는 시점에 언제, 누가 변경 했는지에 대한 정보를 기록하는 기능이다.
        
    * Ex) 등록일, 수정일, 등록자, 수정자
 
* Auditing 구현
  
    * (1) 순수 JPA 사용
    
        * ① 등록일, 수정일을 적용한 클래스 작성
    
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
          
            * JPA 주요 이벤트 애노테이션 : `@PrePersist`, `@PostPersist`, `@PreUpdate`, `@PostUpdate`

                * `@PrePersist` : `persist()` 하기 전에 호출된다.

                * `@PreUpdate` : 수정하기 전에 호출된다.
                
                * ...

        * ② 앞서 작성한 클래스를 상속 받는다.
        
            ```java
            public class Member extends JpaBaseEntity {}
            ```   

    * (2) 스프링 데이터 JPA 사용
    
        * ① 스프링 부트 설정 클래스(`DataJpaApplication`)에 `@EnableJpaAuditing`를 적용한다.
        
            ```java
            @EnableJpaAuditing
            @SpringBootApplication
            public class DataJpaApplication {
            
                public static void main(String[] args) {
                    SpringApplication.run(DataJpaApplication.class, args);
                }
            
            }
            ```
          
            * `@EnableJpaAuditing` : JPA Auditing을 활성화한다.

        * ② `BaseEntity` 클래스를 작성한다.
        
        * ③ `BaseEntity` 클래스에 다음 내용을 적용한다.

            * `@EntityListeners`, `@MappedSuperclass`를 적용한다.

                ```java
                @EntityListeners(AuditingEntityListener.class)
                @MappedSuperclass
                @Getter
                public class BaseEntity {
                    // ...
                }
                ```

                * `@EntityListeners(AuditingEntityListener.class)` : 해당 클래스에 Auditing 기능을 포함시킨다.

                * `@MappedSuperclass` : `BaseEntity`를 상속받는 Entity 클래스가 `BaseEntity`의 필드를 컬럼으로 인식하도록 한다.

                    * `BaseEntity`의 필드라는 것은 createdDate, lastModifiedDate 등을 말한다.
            
            * 그리고 다음 애노테이션을 사용한다.
            
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

                * `@CreatedDate` : 엔티티가 생성되어 저장될 때, 시간이 자동 저장된다.
                
                * `@LastModifiedDate` : 조회한 엔티티의 값을 변경할 때, 시간이 자동 저장된다. 
                
                * `@CreatedBy` : 엔티티가 생성되어 저장될 때, 생성자가 자동 저장된다.
                
                * `@LastModifiedBy` : 조회한 엔티티의 값을 변경할 때, 수정자가 자동 저장된다. 
    
                * `@Column(updatable = false)`는 등록 시간, 등록자는 처음 값을 지정한 이후, 수정하지 못하도록 한다.

            * (등록자, 수정자 필드가 있는 경우) 등록자, 수정자를 처리해주는 `AuditorAware` 스프링 빈을 등록한다.

                ```java
                @Bean
                public AuditorAware<String> auditorProvider(){
                    // 랜덤 UUID 생성
                    return () -> Optional.of(UUID.randomUUID().toString());
                }
                ```

                * 위의 코드에서는 랜덤한 UUID를 생성해서 지정하였다.

                * 하지만 실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 유저 ID를 지정한다.

        * ④ BaseEntity 클래스를 상속 받는다.
            
            ```java
            public class Member extends BaseEntity {}
            ```

    * (3) 등록 시간, 수정 시간과 등록자, 수정자를 분리하기
    
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
        
        @EntityListeners(AuditingEntityListener.class)
        @MappedSuperclass
        @Getter
        public class BaseEntity extends BaseTimeEntity {
            @CreatedBy
            @Column(updatable = false)
            private String createdBy;
        
            @LastModifiedBy
            private String lastModifiedBy;
        }
        ```

        * 실무에서 대부분의 엔티티는 등록 시간, 수정 시간이 필요하지만, 등록자, 수정자는 필요 없는 경우가 있다.
      
        * 그래서 위와 같이 Base 타입을 분리하고, 원하는 타입을 선택해서 상속한다.
        
            * 등록, 수정 시간만 필요하다면 `BaseTimeEntity`를 상속하고 둘다 필요하다면 `BaseEntity`를 상속한다. 

    * (4) Auditing를 전체 적용하기

        * XML 파일을 만들어서 `스프링 데이터 JPA`가 제공하는 이벤트를 엔티티 전체에 적용 할 수 있다.
        
            * 자세한 내용은 해당 강좌를 참고하자.
    
#### 3) Web 확장

* `도메인 클래스 컨버터`는 HTTP 요청으로 전달받은 엔티티의 ID로 엔티티 객체를 찾아서 바인딩한다.
  
    * 예시

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
      
        * 주의사항 
          
            * 도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, 이 엔티티는 단순 조회용으로만 사용해야 한다. 
            
            * 트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영되지 않는다.

* 페이징과 정렬

    * 스프링 MVC에서 스프링 데이터의 페이징과 정렬 사용하기
      
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

        * `Page` : 페이징에 대한 결과 정보를 의미한다.
        
        * `Pageable` : 페이징에 대한 파라미터 정보를 의미한다.
        
        * `Pageable` 타입의 파라미터는 다음과 같은 `요청 파라미터` 정보로 만들어진다. 
        
            * Ex) `/members?page=0&size=3&sort=id,desc&sort=username,desc`
                     
                * `page` : 현재 페이지, 0 부터 시작한다.
                
                * `size` : 한 페이지에 표시할 데이터 수
                
                * `sort` : 정렬 조건을 정의한다. (ASC | DESC)
                
                    * ASC는 기본 값이므로 생략 가능하다.

            * 사용자가 웹 브라우저에서 요청 파라미터 정보를 입력하면 `PageRequest` 객체를 생성해서 주입한다.

    * Pageable 기본 값 변경

        * 글로벌 설정 : 스프링 부트
        
            ```java
            spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/ 
            spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/
            ```
          
        * 개별 설정 : `@PageableDefault` 애노테이션을 사용한다.
        
            ```java
            @GetMapping("/members")
            public Page<Member> list(@PageableDefault(size = 5, sort = "username") Pageable pageable){
                Page<Member> page = memberRepository.findAll(pageable);
                return page;
            }
            ```
          
            * 개별 설정은 글로벌 설정 보다 우선적으로 적용된다.
          
    * 접두사        

        * 사용해야 할 페이징 정보가 둘 이상이면 접두사로 구분한다.
        
            * `@Qualifier`에 접두사명을 추가한다. "{접두사명}_xxx”
            
                * Ex) `/members?member_page=0&order_page=1`
        
                    ```java
                    public String list(
                            @Qualifier("member") Pageable memberPageable, 
                            @Qualifier("order") Pageable orderPageable, ...
                    ```
                      
    * Page의 내용을 DTO로 변환하기       

        * 엔티티를 외부 API로 노출하면 다양한 문제가 발생한다. 그래서 엔티티를 꼭 DTO로 변환해서 반환해야 한다.
     
            * 외부 API로 반환할 때는 컨트롤러의 리턴 타입을 `Page<Member>`가 아닌 `Page<MemberDto>`로 사용해야 한다.
     
        * Page는 `map()`을 지원해서 내부 데이터를 다른 것으로 변경할 수 있다.

            * ① DTO를 작성한다.

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
              
                * DTO에서 엔티티(Member)를 필드가 아닌 파라미터로 사용하는 것은 좋은 방법이다. 

            * ② `Page.map()`을 사용해서 엔티티를 DTO로 변경한다.
            
                ```java
                @GetMapping("/members")
                public Page<MemberDto> list(@PageableDefault(size = 5) Pageable pageable){
                    return memberRepository.findAll(pageable)
                      .map(MemberDto::new);
                }
                ```

    * Page를 1 부터 시작하기      

        * 스프링 데이터는 Page를 0부터 시작한다. 만약 1부터 시작하려면 어떻게 해야 될까?
        
            * 첫 번째 방법 - 실질적인 해결책
            
                * `Pageable`, `Page`를 파라미터와 응답 값으로 사용하지 않고, 직접 클래스를 만들어서 처리한다. 
                
                * 그리고 직접 `PageRequest`(Pageable 구현체)를 생성한 다음, 리포지토리에 전달한다. 
                
                * 물론 응답 값도 Page 대신에 직접 만들어서 제공해야 한다.
                
            * 두 번째 방법
            
                * `application.properties`에서 `spring.data.web.pageable.one-indexed-parameters`를 `true`로 설정한다. 
                                
                    * Ex) `http://localhost:8080/members?page=1` 처럼 페이지 1을 요청한다. 
                    
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
                        
                        * 그런데 이 방법은 web에서 page 파라미터를 -1 처리 할 뿐이다.
                        
                        * 따라서 응답 값인 Page에 모두 0 페이지 인덱스를 사용하는 한계가 있다.

        * 가급적 권장하는 것은 페이지 인덱스를 0 부터 처리하자.
        
## 6. 스프링 데이터 JPA 분석

#### 1) 스프링 데이터 JPA 구현체 분석

* 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체 (`SimpleJpaRepository`)

    * `org.springframework.data.jpa.repository.support.SimpleJpaRepository`
    
    * `SimpleJpaRepository`의 코드를 살펴보기
    
        * ① `@Repository` 적용
        
            * 컴포넌트 스캔의 대상이 되어 스프링 빈으로 등록된다.
            
            * JPA 예외를 스프링이 추상화한 예외로 변환한다.
    
                * 하부 기술을 JDBC에서 JPA로 변경 하더라도 예외를 처리하는 메커니즘이 동일하다는 장점이 있다.
            
        * ② `@Transactional` 적용
            
            * JPA의 모든 변경은 트랜잭션 안에서 이루어져야 한다.
                
                * `스프링 데이터 JPA`는 변경(등록, 수정, 삭제) 메소드를 트랜잭션 안에서 처리해야 한다. (그렇지 않으면 예외가 발생함)
                
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
                  
                    * 사실은 트랜잭션이 리포지토리 계층에 이미 걸려있는 것임

        * ③ `@Transactional(readOnly = true)`
            
            * 데이터를 조회만 하고 변경하지 않는 트랜잭션에서는 `readOnly = true` 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음

            * 즉, `readOnly = true`는 플러시를 생략한다.

        * ④ `save()` 메소드
            
            * 새로운 엔티티면 저장(`persist`)하고 새로운 엔티티가 아니면 병합(`merge`)한다.

#### 2) 새로운 엔티티를 구별하는 방법

* 새로운 엔티티를 판단하는 기본 전략

    * ① 식별자가 객체일 때는 `null`이면 새로운 엔티티로 판단한다.
    
        * `em.persist()`를 호출한 이후, 식별자가 설정된다.
    
    * ② 식별자가 자바 기본 타입일 때는 `0`이면 새로운 엔티티로 판단한다. 
    
        * `Persistable` 인터페이스를 구현해서 판단 로직을 변경 할 수 있다.

* 직접 식별자를 할당해야되는 경우

    * JPA 식별자 생성 전략이 `@GeneratedValue`면 `save()` 호출 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 정상적으로 동작한다. 
    
    * 그런데 `@Id`만 사용해서 직접 식별자를 할당하게 되면 이미 식별자 값이 있는 상태로 `save()`를 호출하게 된다. 
    
    * 따라서 이 경우에는 `merge()`가 호출되며 `merge()`는 우선 DB에서 값을 조회하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율적이다.
    
    * 그래서 **`Persistable` 인터페이스를 구현해서 새로운 엔티티를 판단하는 로직을 직접 작성한다.**
    
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
        
            // 오버라이딩
            @Override
            public String getId() {
                return id;
            }
            
            /*
            * @CreatedDate는 persist()가 되기 전에 호출된다.   
            * 그래서 판단 로직을 다음과 같이 구성한다.
            * createdDate가 null이면 새로운 객체로 판단한다.
            * createdDate의 값이 있다면 새로운 객체가 아니다.
            * */        
            @Override
            public boolean isNew() {
                return createdDate == null;
            }
        
        }
        ```
      
        ```java
        @SpringBootTest
        class ItemRepositoryTest {
        
            @Autowired ItemRepository itemRepository;
        
            @Test
            public void save(){
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


