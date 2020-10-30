# 김영한님의 실전! 스프링 데이터 JPA
> 아래 내용은 [실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84 "실전! 스프링 데이터 JPA") 강좌를 정리한 내용 입니다.

## 1. 프로젝트 환경설정

* `@PersistenceContext` : EntityManager를 주입 받을 때 사용한다.

* JPA 기본편 내용 복습

    * JPA의 모든 데이터 변경은 트랜잭션 안에서 이루어져야 한다.

    * JPA는 같은 트랜잭션 안에서 영속성 컨텍스트의 동일성을 보장한다.

    * (예시에서 `member == findMember` 이다.)

* `@Transactional`

    * 트랜잭션을 처리 할 때 사용한다.
    
    * @Transactional를 테스트 코드에 사용하면 기본적으로 테스트가 종료된 다음, 바로 롤백을 한다.
    
    * @Rollback(false)를 추가하면 롤백을 하지 않고 커밋을 한다. 
    
## 2. 예제 도메인 모델

* 실무에서는 가급적 `@Setter`를 사용하지 않고 비즈니스 메소드를 따로 만드는 것이 좋다.

* `@NoArgsConstructor(access = AccessLevel.PROTECTED)`

* `@ToString`는 연관관계가 없는 필드만 지정한다. (무한루프 방지 하기 위함)

* `양방향 연관관계`는 `연관관계 편의 메소드`를 만들어 한번에 처리하는 것이 좋다. `changeTeam()`

## 3. 공통 인터페이스 기능

#### 1) 공통 인터페이스 설정

* 스프링 설정에 자바 설정 파일을 사용하면 `@EnableJpaRepositories` 애노테이션을 추가하고 `basePackages`에 리포지토리를 검색할 패키지 위치를 지정한다. 스프링 부트 사용 시 생략 가능하다. `@SpringBootApplication`

 ```java
@Configuration
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
public class AppConfig {}
 ```

* `스프링 데이터 JPA`는 애플리케이션을 실행할 때, basePackage에 있는 리포지토리 인터페이스들을 찾아서 해당 인터페이스를 구현한 클래스를 동적으로 생성한 다음, 스프링 빈으로 등록한다.

* 따라서 개발자가 직접 구현 클래스를 만들지 않아도 된다.

* `JpaRepository` 인터페이스를 상속 받으면 컴포넌트 스캔의 대상이 되며 `@Repository`를 생략 할 수 있다.

#### 2) 공통 인터페이스 적용

* `JpaRepository`의 제네릭을 보면 `T`는 엔티티 타입, `ID`는 식별자 타입(PK)을 의미한다.

#### 3) 공통 인터페이스 분석

* `JpaRepository`는 공통 CRUD를 제공한다.

* 제네릭은 <엔티티 타입, 식별자 타입> 설정

 ```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
 ```

* 공통 인터페이스 구성

    * JpaRepository 인터페이스의 계층 구조
    
        * 스프링 데이터 : Repository, CrudRepository, PagingAndSortingRepository가 있는데 스프링 데이터가 프로젝트가 공통으로 사용하는 인터페이스다.
        
        * 스프링 데이터 JPA : JpaRepository 인터페이스는 여기에 추가로 JPA에 특화된 기능을 제공한다.
    
    * 제네릭 타입
    
        * T : 엔티티 , ID : 엔티티의 식별자 타입 , S : 엔티티와 그 자식 타입 
    
    * JpaRepository의 주요 메소드
    
        * `save(S)` : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합한다.
        
        * `delete(T)` : 엔티티 하나를 삭제한다. 내부에서 EntityManager.remove()를 호출한다.
        
        * `findById(ID)` : 엔티티 하나를 조회한다. 내부에서 EntityManager.find()를 호출한다.
        
        * `getOne(ID)` : 엔티티를 프록시로 조회한다. 내부에서 EntityManager.getReference()를 호출한다. 
        
        * `findAll(...)` : 모든 엔티티를 조회한다. 정렬(`Sort`)이나 페이징(`Pageable`) 조건을 파라미터로 제공 할 수 있다.

## 4. 쿼리 메소드 기능

* 스프링 데이터 JPA가 제공하는 `쿼리 메소드 기능 3가지`

    * 메소드 이름으로 쿼리 생성
    * 메소드 이름으로 JPA NamedQuery 호출
    * @Query 어노테이션을 사용해서 리포지토리 인터페이스에 쿼리 직접 정리

#### 1) 메소드 이름으로 쿼리 생성

* 스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행한다.

 ```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);

}
 ```

* 메소드 이름으로 사용 할 수 있는 키워드

    * [스프링 데이터 JPA 공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods "") 참고

* 검색 조건으로 사용되는 컬럼이 2개 이하일 때 사용하기 좋은 방식

#### 2) JPA NamedQuery 

* JPA NamedQuery

    * 스프링 데이터 JPA는 메소드 이름으로 JPA의 NamedQuery를 호출하는 기능을 제공한다.

    * NamedQuery는 쿼리에 이름을 부여해서 사용하는 방법이다.

* @NamedQuery 애노테이션으로 Named 쿼리 정의

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

    * 스프링 데이터 JPA는 선언한 "도메인 클래스 + .(점) + 메서드 이름"으로 Named 쿼리를 찾아서 실행한다.
    
    * 만약 실행할 Named 쿼리가 없으면 "메소드 이름으로 쿼리 생성" 전략을 사용한다.
    
    * 필요하면 전략을 변경할 수 있지만 권장하지 않는다.

 ```java
public interface MemberRepository extends JpaRepository<Member, Long> { //** 여기 선언한 Member 도메인 클래스
      List<Member> findByUsername(@Param("username") String username);
}
 ```

* `@Param`는 이름 기반 파라미터를 바인딩할 때 사용하는 애노테이션이다.

* Named 쿼리는 실무에서 거의 사용되지 않는다.

#### 3) @Query, 리포지토리 메소드에 쿼리 정의하기

* 리포지토리 메소드에 `@Query`를 사용하여 직접 쿼리를 정의 할 수 있다. 

 ```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);

}
 ```

* 실행할 메서드에 정적 쿼리를 직접 작성하므로 이름 없는 Named 쿼리라 할 수 있다.

* JPA Named 쿼리처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있다는 장점이 있다. 

#### 4) @Query, 값, DTO 조회하기

* 단순히 값 하나를 조회

     ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        @Query("select m.username from Member m")
        List<String> findUsernameList();
    }
     ```

* DTO로 직접 조회

    * DTO(Data Transfer Object) : 계층 간 데이터 교환을 위한 객체이다.

        1. DTO를 만든다.
        
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
           
        2. DTO로 직접 조회하려면 JPA의 new 명령어를 사용해야 한다. 그리고 다음과 같이 생성자가 맞는 DTO가 필요하다.
        
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

* 파라미터 바인딩

   * 스프링 데이터 JPA는 `유연한 반환 타입`을 지원한다.
   
   * 결과가 한 건 이상이면 `컬렉션 인터페이스`를 사용하고, 단건이면 반환 타입을 지정한다.
   
        ```java
        List<Member> findListByUsername(String username);           // 컬렉션
        Member findMemberByUsername(String username);               // 단건
        Optional<Member> findOptionalByUsername(String username);   // 단건 Optional
        ```
     
   * 조회 결과가 많거나 없으면 어떻게 될까?
   
     1. 컬렉션 조회인 경우
     
        * 조회 결과가 없으면 빈 컬렉션 반환
     
     2. 단건 조회인 경우
     
        * 조회 결과가 없으면 `null` 반환
        
        * 조회 결과가 2건 이상이면 `javax.persistence.NonUniqueResultException` 예외가 발생한다.
     
     



