# 김영한님의 자바 ORM 표준 JPA 프로그래밍 - 기본편
> 아래 내용은 [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic "자바 ORM 표준 JPA 프로그래밍 - 기본편") 강좌를 정리한 내용 입니다.

## 8. 프록시와 연관관계 관리

### 1. 프록시(Proxy)

* `프록시 객체`는 실제 엔티티 객체를 대신하는 데이터베이스 조회를 지연 할 수 있는 가짜 객체를 말한다.

* `em.find()` vs `em.getReference()` 

    * `em.find()` : 데이터베이스를 통해서 실제 엔티티 객체를 조회한다.
    
    * `em.getReference()` : 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체를 반환한다.

* 프록시 특징

    * 프록시 객체는 실제 객체의 참조(target)를 보관한다.

    * 프록시 객체의 메소드를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.
    
* 프록시 객체의 초기화
    
    * `프록시 객체의 초기화`는 프록시 객체가 실제 사용될 때, 데이터베이스를 조회해서 실제 엔티티 객체를 생성하는 것을 말한다.
    
        * 즉, 프록시 객체의 메소드를 호출하면 초기화가 진행된다.

            ```java
            Member member = em.getReference(Member.class, “id1”);
            member.getName(); // 프록시 객체의 메소드 호출
            ```

    * 프록시의 초기화 과정
    
        * ① 프록시 객체에 member.getName()을 호출해서 실제 데이터를 조회한다.
          
        * ② 프록시 객체는 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티 생성을 요청하는데 이것을 초기화라 한다.
          
        * ③ 영속성 컨텍스트는 데이터베이스를 조회해서 실제 엔티티 객체를 생성한다.
          
        * ④ 프록시 객체는 생성된 실제 엔티티 객체의 참조를 Member 타입의 target 멤버변수에 보관한다.
          
        * ⑤ 프록시 객체는 실제 엔티티 객체의 getName()을 호출해서 결과를 반환한다.
        
* 프록시 정리

    * 프록시 객체는 처음 사용할 때 한 번만 초기화된다.

    * 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것이 아니다. 

    * 프록시 객체가 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 할 수 있게 되는 것이다.

    * 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시, 주의해야 한다. 

        * `==` 비교 대신에 `instanceof`를 사용해야 한다.

    * 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 데이터베이스를 조회할 필요가 없으므로 `em.getReference()`를 호출해도 프록시가 아닌 실제 엔티티를 반환한다.

    * 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화 하면 문제가 발생한다.

        * 하이버네이트는 `org.hibernate.LazyInitializationException` 예외를 발생시킨다.

        * 문제가 발생하는 이유는 프록시의 초기화 요청은 영속성 컨텍스트를 통해 이루어지기 때문이다. 
        
### 2. 즉시 로딩과 지연 로딩

* JPA는 개발자가 연관된 엔티티의 조회 시점을 선택 할 수 있도록 두 가지 방법을 제공한다.

    * ① `즉시 로딩(EAGER LOADING)` : 엔티티를 조회할 때, 연관된 엔티티도 함께 조회하는 방식이다.
    
        * 예시 : `em.find(Member.class, "member1")` 처럼 호출할 때, 회원 엔티티와 연관된 팀 엔티티도 함께 조회한다.
    
        * 설정 방법 : `@ManyToOne(fetch = FetchType.EAGER)`
    
    * ② `지연 로딩(LAZY LOADING)`: 연관된 엔티티를 실제 사용할 때까지 데이터베이스 조회를 지연하는 방식이다.
    
        * 연관된 엔티티를 프록시로 조회한다. 프록시를 실제 사용할 때, 초기화하면서 데이터베이스를 조회한다.
    
        * 예시 : `member.getTeam().getName()`처럼 조회한 팀 엔티티를 실제 사용하는 시점에 데이터베이스를 조회해서 프록시 객체를 초기화한다.
    
            * 조회 대상이 영속성 컨텍스트에 이미 있으면 프록시 객체가 아닌 실제 객체를 사용한다.
    
        * 설정 방법 : `@ManyToOne(fetch = FetchType.LAZY)`
        
* 주의사항

    * 실무에서는 모든 연관관계에 지연 로딩(LAZY LOADING)를 사용해야 한다.

    * 즉시 로딩(EAGER LOADING)을 적용하면 예상하지 못한 쿼리가 발생한다.

    * 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.

        * `N+1 문제` : 처음에 쿼리 1개를 실행 했을 때, 얻은 결과의 개수(N개)만큼 추가 쿼리가 발생하는 문제를 말한다.

        * N+1 문제 해결 방안 (연관된 엔티티를 함께 조회해야 되는 경우)
        
            * JPQL의 fetch 조인을 이용한다.

            * 또는 `@EntityGraph`를 이용한다.
            
            * 또는 `@BatchSize`를 이용 할 수도 있다.

    * `@ManyToOne`, `@OneToOne`은 기본 값이 즉시 로딩이므로 LAZY로 설정해야 한다.
    
        * `@XToOne` 시리즈 

### 3. 영속성 전이: CASCADE

* CASCADE는 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용한다.

    * `@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)`

* 부모만 영속 상태로 만들면 연관된 자식 엔티티까지 함께 영속화해서 저장한다.

* 주의사항

    * 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없다.
    
    * 단지 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐이다.
    
    * 하나의 부모만 자식들을 관리 한다면 CASCADE 속성을 사용 할 수 있다.
    
* CASCADE의 종류

    * ALL: 모두 적용

    * PERSIST: 영속

    * REMOVE: 삭제
    
    * ...
    
### 4. 고아 객체

* `고아 객체(ORPHAN) 제거`는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능이다.

    * `@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST, orphanRemoval = true)`

* 주의사항

    * `고아 객체 제거`는 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 판단하고 삭제하는 기능이다.

    * 특정 엔티티가 개인 소유하는 엔티티에만 사용해야 한다.

    * orphanRemoval는 `@OneToOne`, `@OneToMany`에만 사용할 수 있다.

    * 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다. (부모를 제거하면 자식은 고아가 되기 때문이다.)

* 영속성 전이 + 고아 객체, 생명주기

    * `CascadeType.ALL + orphanRemovel=true` 

        * 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있다.

## 9. 값 타입

### 1. JPA의 데이터 타입 분류

* JPA의 데이터 타입은 크게 `엔티티 타입`과 `값 타입`으로 나눌 수 있다.

    * (1) 엔티티 타입

        * `엔티티 타입`은 `@Entity`로 정의하는 객체이다.

        * 데이터가 변해도 식별자를 통해 지속해서 추적 할 수 있다.

        * 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능하다. 

    * (2) 값 타입(Value Type)

        * `값 타입`은 int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체를 말한다.

        * 식별자가 없고 값만 있으므로 변경 시 추적 할 수 없다.

        * 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체된다.
        
* 값 타입 분류

    * `값 타입`은 크게 3가지로 나눌 수 있다.

        * ① `기본값 타입`
    
            * 자바 기본 타입(int, double)
        
            * 래퍼 클래스(Integer, Long)
        
            * String

        * ② `임베디드 타입(embedded type, 복합 값 타입)`

        * ③ `컬렉션 값 타입(collection value type)`
        
* (1) 기본 값 타입

    * 예를 들어 `String name`, `int age`가 기본 값 타입이다.

    * 생명주기를 엔티티에 의존한다.
  
        * Ex) 회원 엔티티를 삭제하면 이름, 나이 필드도 함께 삭제된다.

    * 값 타입은 공유하면 안 된다.

        * Ex) 회원 이름 변경 시 다른 회원의 이름도 함께 변경되면 안 되기 때문이다.

    * [참고] 자바의 기본 타입은 절대 공유되지 않는다.

        * int, double 같은 기본 타입(primitive type)은 절대 공유되지 않는다.

        * 기본 타입은 항상 값을 복사하기 때문이다.

        * Integer 같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경 불가능하다.
        
* (2) 임베디드 타입(복합 값 타입)

    * 임베디드 타입(embedded type)은 새로운 값 타입을 직접 정의해서 사용하는 것을 말한다.
    
    * 주로 기본 값 타입들을 모아서 만들기 때문에 “복합 값 타입”이라고도 한다.
    
    * 임베디드 타입은 int, String과 같은 값 타입이다.

    * 사용 방법

        * `@Embeddable` : 값 타입을 정의하는 곳에 사용한다.
    
        * `@Embedded` : 값 타입을 사용하는 곳에 사용한다.

        * 예시
        
            ```java
            @Entity
            public class Member{
            
                @Id @GeneratedValue
                @Column(name = "MEMBER_ID")
                private Long id;
            
                @Column(name = "USERNAME")
                private String username;
            
                // Period
                @Embedded
                private Period workPeriod;
            
                // 주소
                @Embedded
                private Address homeAddress;
            
                // Getter, Setter
            }
            ```
          
            ```java
            @Embeddable
            public class Address {
                private String city;
                private String street;
                private String zipcode;
            
                // 기본 생성자 필수
                public Address() {
                }
            
                public Address(String city, String street, String zipcode) {
                    this.city = city;
                    this.street = street;
                    this.zipcode = zipcode;
                }
          
                /*
                * Getter만 만들기
                * "값 타입"을 여러 엔티티에서 공유하면 위험하다. 그 이유는 부작용(side effect)이 발생하기 때문이다.
                * 불변 객체(immutable object)로 만들기 위해, 생성자로만 값을 설정하고 수정자(Setter)는 만들지 않는다.
                * */
          
            }

            ```

    * 임베디드 타입과 테이블 매핑
    
        * 임베디드 타입은 엔티티의 값일 뿐이다. 따라서 값이 속한 엔티티의 테이블에 매핑한다.
    
        * 임베디드 타입을 사용하기 전과 후에 **매핑하는 테이블은 같다.**
    
        * 임베디드 타입 덕분에 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능하다.
        
    * @AttributeOverride

        * 한 엔티티에서 같은 값 타입을 사용하면 테이블에 매핑하는 컬럼 명이 중복된다.

        * 이때는 `@AttributeOverrides`, `@AttributeOverride`를 사용해서 컬럼 명 속성을 재 정의한다.
       
    * 임베디드 타입과 null

        * 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null이 된다.
        
    * [참고] 자바에서 제공하는 객체 비교는 2가지가 있다.

        * ① `동일성(identity) 비교` : 인스턴스의 참조 값을 비교, == 사용

        * ② `동등성(equivalence) 비교` : 인스턴스의 값을 비교, equals() 사용

* (3) 값 타입 컬렉션

    * 값 타입 컬렉션?

        * `값 타입 컬렉션`은 **값 타입을 컬렉션에 넣어서 사용하는 것**을 말한다.

            * 값 타입을 하나 이상 저장할 때, 값 타입 컬렉션을 사용한다.
                
        * `@ElementCollection`, `@CollectionTable`를 사용해서 맵핑한다.
        
        * 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다. 
        
        * 컬렉션을 저장하기 위한 별도의 테이블이 필요하다.
        
        * 값 타입 컬렉션은 기본적으로 지연 로딩 전략을 사용한다.
        
    * 값 타입 컬렉션 맵핑 예시
              
        ```java
        @Entity
        public class Member{
        
            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private Long id;
        
            @Column(name = "USERNAME")
            private String username;
        
            @Embedded
            private Address homeAddress;
        
            @ElementCollection
            @CollectionTable(name = "FAVORITE_FOOD",
                             joinColumns = @JoinColumn(name = "MEMBER_ID"))
            @Column(name = "FOOD_NAME")
            private Set<String> favoriteFoods = new HashSet<>();
        
            @ElementCollection
            @CollectionTable(name = "ADDRESS",
                             joinColumns = @JoinColumn(name = "MEMBER_ID"))
            private List<Address> addressHistory = new ArrayList<>();
        
            // Getter, Setter
      
        }
        ```
      
    * 값 타입 컬렉션의 제약사항

        * 값 타입 컬렉션에 변경사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.

        * 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다.

        * 컬럼에 null을 입력 할 수 없고, 같은 값을 중복해서 저장 할 수 없다.
        
    * 값 타입 컬렉션 대안

        * 실무에서는 `값 타입 컬렉션` 대신에 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용한다.
        
            * 다대일 일대다 양방향 관계로 하면 update 쿼리를 없앨 수 있다.
        
        * `영속성 전이(Cascade)` + `고아 객체 제거(ORPHAN REMOVE)` 기능을 적용하면 값 타입 컬렉션처럼 사용 할 수 있다.
        
        * 예시
        
            * ① AddressEntity 만들기
         
                ```java
                @Entity
                @Table(name = "ADDRESS")
                public class AddressEntity {
                
                    @Id @GeneratedValue
                    private Long id;
                
                    private Address address;
                
                    public AddressEntity() {
                    }
                
                    public AddressEntity(String city, String street, String zipcode) {
                        this.address = new Address(city, street, zipcode);
                    }
                
                    // Getter, Setter
                
                }
                ```
            
            * ② Member 엔티티 코드 변경
            
                ```java
                @Entity
                public class Member{
                
                    @Id @GeneratedValue
                    @Column(name = "MEMBER_ID")
                    private Long id;
                
                    @Column(name = "USERNAME")
                    private String username;
                
                    @Embedded
                    private Address homeAddress;
                
                    // 일대다 단방향 맵핑
                    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
                    @JoinColumn(name = "MEMBER_ID")
                    private List<AddressEntity> addressHistory = new ArrayList<>();
                
                }
                ```
              
## 10. 객체지향 쿼리 언어

### 1. 객체지향 쿼리 언어 소개 

* JPA는 다양한 쿼리 방법을 지원

    * JPQL
    
    * JPA Criteria
    
    * QueryDSL
    
    * 네이티브 SQL

    * JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

* JPQL

    * 애플리케이션이 필요한 데이터만 DB에서 가져오려면 결국 검색 조건이 포함된 SQL이 필요하다. 

    * JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.

    * SQL과 문법이 유사하다. (SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원)
    
    * JPQL은 엔티티 객체를 대상으로 쿼리한다.
    
    * SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
    
* Criteria

    * JPQL을 생성하는 빌더 역할을 한다.
    
    * 문자가 아닌 자바 코드로 JPQL을 작성할 수 있다.

    * **단점: 너무 복잡하고 실용성이 없다.** 
    
    * `Criteria` 대신에 `QueryDSL` 사용 권장
    
* QueryDSL

    * JPQL을 생성하는 빌더 역할을 한다.

    * 문자가 아닌 자바 코드로 JPQL을 작성할 수 있다.
   
    * 컴파일 시점에 문법 오류를 찾을 수 있음
    
    * 동적 쿼리를 작성하는데 편리하다.
    
        * `동적 쿼리`는 상황에 따라 분기 처리를 통해 SQL을 동적으로 만드는 것을 말한다.
    
    * 단순하고 쉽기 때문에 실무에서 사용하는 것을 권장한다.

* 네이티브 SQL

    * SQL을 직접 사용 할 수 있는 기능을 제공한다.
    
    * JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 사용할 때 이용한다.
     
    * 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

        ```java
        String sql =
            “SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = ‘kim’";
        List<Member> resultList =
                    em.createNativeQuery(sql, Member.class).getResultList();
        ```

* JDBC 직접 사용, SpringJdbcTemplate 등

    * JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스 등을 함께 사용 할 수 있다.
    
    * 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시를 할 필요가 있다.
    
        * 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 flush() 해야 한다.

### 2. JPQL

* JPQL(Java Persistence Query Language, 자바 영속성 쿼리 언어)

    * JPQL는 테이블이 아닌 엔티티 객체를 대상으로 검색하는 객체지향 쿼리 언어이다.
    
    * 애플리케이션이 필요한 데이터만 DB에서 가져오려면 결국 검색 조건이 포함된 SQL이 필요하다. 
        
    * JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
    
    * JPQL은 결국 SQL로 변환된다.
    
* JPQL 문법

    * 문법
    
        ```java
        SELECT 별칭 (별칭.필드명) 
        FROM 엔티티명 AS 별칭
        WHERE 조건식
        ```
      
    * 예시

        * `select m from Member as m where m.age > 18`
        
            * 엔티티와 속성은 대소문자를 구분한다. (Member, age)
            
            * JPQL 키워드는 대소문자를 구분하지 않는다. (SELECT, FROM, where)
            
            * 테이블 이름이 아닌 엔티티 이름을 사용한다. (Member) 
            
            * 별칭이 필수다. (m) 
            
            * as는 생략 가능하다.

* TypedQuery, Query

    * `TypedQuery` : 반환 타입이 명확할 때 사용한다.
    
        ```java
        TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
        ```
    
    * `Query` : 반환 타입이 명확하지 않을 때 사용한다.
    
        ```java
        // username은 String , age는 int이므로 타입 정보가 명확하지 않다.
        Query query = em.createQuery("select m.username, m.age from Member m");
        ```
      
* 결과 조회 API

    * `query.getResultList()` : 결과가 하나 이상일 때, 리스트를 반환한다.
    
        * 결과가 없으면 비어있는 리스트를 반환한다.
        
        ```java
        List<Member> resultList = em.createQuery("select m from Member m", Member.class).getResultList();

        for (Member member1 : resultList) {
            System.out.println("member = " + member1);
        }
        ```
    
    * `query.getSingleResult()` : 결과가 정확히 하나일 때, 단일 객체를 반환한다.

        * 결과가 없으면: `javax.persistence.NoResultException` 예외 발생
        
        * 둘 이상이면: `javax.persistence.NonUniqueResultException` 예외 발생

* 파라미터 바인딩

    * JPQL는 `이름 기준 파라미터 바인딩`과 `위치 기준 파라미터 바인딩`을 제공한다.
    
        * `이름 기준 파라미터`는 파라미터를 이름으로 구분하는 방법이다.
        
            * 앞에 `:`를 사용한다.
            
                ```java
                TypedQuery<Member> query = em.createQuery("select m from Member m where m.username = :username", Member.class);
                query.setParameter("username", "member1");
                Member singleResult = query.getSingleResult();
                
                System.out.println("singleResult = " + singleResult.getUsername());
                ```
              
        * `위치 기준 파라미터`는 ? 다음에 위치 값을 주면 된다. (위치 값은 1부터 시작한다.)
        
            ```java
            Member singleResult = em.createQuery("select m from Member m where m.username = ?1", Member.class)
                    .setParameter(1, "member1")
                    .getSingleResult();
    
            System.out.println("singleResult = " + singleResult.getUsername());
            ```

    * `이름 기준 파라미터 바인딩 방식`을 사용하는 것이 좋다.
    
* 프로젝션

    * `프로젝션(projection)`은 SELECT 절에 조회할 대상을 지정하는 것을 말한다.
    
        * 프로젝션 대상은 엔티티, 임베디드 타입, 스칼라 타입이 있다.
        
            * 스칼라 타입은 숫자, 문자 등 기본 데이터 타입을 의미한다.
            
        * DISTINCT로 중복을 제거 할 수 있다.
        
        * 엔티티 프로젝션으로 조회한 엔티티는 영속성 컨텍스트에서 관리된다.
    
    * 예시
    
        ```java
        SELECT m FROM Member m                    // 엔티티 프로젝션 
        SELECT m.team FROM Member m               // 엔티티 프로젝션
        SELECT m.address FROM Member m            // 임베디드 타입 프로젝션
        SELECT m.username, m.age FROM Member m    // 스칼라 타입 프로젝션
        ```
      
    * 프로젝션에서 여러 값을 조회하는 방법
    
        * ① Query 타입으로 조회
        
        * ② Object[] 타입으로 조회

            ```java
            List<Object[]> resultList = em.createQuery("select distinct m.username, m.age from Member m")
                    .getResultList();

            for (Object[] row : resultList) {
                System.out.println("username = " + row[0]);
                System.out.println("age = " + row[1]);
            }
            ```

        * ③ new 명령어로 조회
        
            * DTO를 작성한다.

                ```java
                public class MemberDTO {
                
                    private String username;
                
                    private int age;
              
                    public MemberDTO(String username, int age) {
                        this.username = username;
                        this.age = age;
                    }
        
                    // Getter, Setter
                }
                ```
              
            * 여러 값을 DTO로 조회한다.

                ```java          
                List<MemberDTO> result = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
                        .getResultList();
                
                for (MemberDTO dto : result) {
                    System.out.println("username = " + dto.getUsername());
                    System.out.println("age = " + dto.getAge());
                }
                ```
              
                * 패키지 명을 포함한 전체 클래스명을 입력해야 한다.
                
                * 순서와 타입이 일치하는 생성자가 필요하다.

* 페이징 API

    * JPA는 페이징을 다음 두 API로 추상화 했다.

        * `setFirstResult(int startPosition)` : 조회 시작 위치 (0부터 시작한다)
        
        * `setMaxResults(int maxResult)` : 조회할 데이터 수
        
    * 예시

        ```java          
        // 0 번째 부터 시작해서 10개의 데이터를 조회한다.
        List<Member> result = em.createQuery("select m from Member m order by m.age desc", Member.class)
                .setFirstResult(0)
                .setMaxResults(10)
                .getResultList();
        ```

* 집합과 정렬

    * 집합은 집합함수와 함께 통계 정보를 구할 때, 사용한다.

        * 집합 함수
        
            * `COUNT` : 결과 수를 구한다.
            
            * `MAX, MIN` : 최대, 최소 값을 구한다. 문자, 숫자, 날짜 등에 사용한다.
            
            * `AVG` : 평균 값을 구한다. 숫자 타입만 사용 할 수 있다.
            
            * `SUM` : 합계를 구한다. 숫자 타입만 사용 할 수 있다. 

        * 집합 함수 사용 시 참고사항
        
            * NULL 값은 무시하므로 통계에 잡히지 않는다.
            
            * 값이 없는데 SUM, AVG, MAX, MIN 함수를 사용하면 NULL 값이 된다. 단 COUNT는 0이 된다.
            
            * DISTINCT를 집합 함수 안에 사용해서 중복된 값을 제거하고 집합을 구할 수 있다.
            
                * select COUNT(DISTINCT m.age) from Member m

        * GROUP BY, HAVING
        
            * `GROUP BY` : 특정 그룹끼리 묶어준다.
            
            * `HAVING` : GROUP BY로 그룹화한 통계 데이터를 기준으로 필터링한다.

                ```java          
                // 팀 이름을 기준으로 그룹별로 묶은 다음, 평균 나이가 10살 이상인 그룹을 조회한다.
                SELECT t.name, COUNT(m.age), SUM(m.age), AVG(m.age), MAX(m.age), MIN(m.age)
                FROM Member m LEFT JOIN m.team t
                GROUP BY t.name  
                HAVING AVG(m.age) >= 10
                ```

        * 정렬(ORDER BY)
        
            * `ORDER BY` : 결과를 정렬할 때 사용한다.
            
                ```java          
                // 나이를 기준으로 내림차순으로 정렬하고 나이가 같으면 이름을 기준으로 오름차순으로 정렬한다.
                SELECT t.name, COUNT(m.age) AS cnt  
                FROM Member m LEFT JOIN m.team t  
                GROUP BY t.name  
                ORDER BY t.name ASC, cnt DESC
                ```
              
            * `ASC` : 오름차순(기본 값), `DESC` : 내림차순

* 조인

    * 내부 조인은 INNER JOIN을 사용한다. (INNER는 생략 가능)
    
        ```java          
        // 내부 조인
        String query = "select m from Member m inner join m.team t";
        List<Member> result = em.createQuery(query, Member.class)
                .getResultList();
        ```
      
        * JPQL 조인은 연관 필드를 사용한다.
        
            * 연관 필드는 다른 엔티티와 연관관계를 가지기 위해 사용하는 필드를 말한다.
            
        * JPQL은 JOIN 명령어 다음에 조인할 객체의 연관 필드를 사용한다.
        
    * 외부 조인은 `LEFT [OUTER] JOIN`을 사용한다. (OUTER는 생략 가능)
    
        ```java          
        // 외부 조인
        String query = "select m from Member m left join m.team t";
        List<Member> result = em.createQuery(query, Member.class)
                .getResultList();
        ```
      
    * 세타 조인은 전혀 관계없는 엔티티를 조인할 때 사용한다. (내부 조인만 지원)
    
        ```java          
        // 세타 조인
        String query = "select m from Member m, Team t where m.username = t.name";
        ```
      
    * ON 절
    
        * ON절을 사용하면 **조인 대상을 필터링**하고 조인 할 수 있다.
        
            * 예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
            
                ```java          
                String query = "select m from Member m left join m.team t on t.name = 'teamA'";
                ```
        
        * **연관관계가 없는 엔티티 외부 조인**이 가능하다.
        
            * 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
            
                ```java          
                String query = "select m from Member m left join Team t on m.username = t.name";
                ```
              
* 서브쿼리

    * `서브쿼리(SubQuery)`은 쿼리문 안에 포함되어 있는 또 다른 쿼리문을 말한다.
    
        * JPQL에서 서브쿼리는 WHERE, HAVING 절에서만 사용 할 수 있다.
        
            * 하이버네이트는 SELECT 절의 서브쿼리도 허용한다.
            
            * FROM절의 서브쿼리는 지원하지 않는다.
            
                * 조인으로 해결 할 수 있다면 조인으로 해결한다.
                
                * 또는 네이티브 쿼리로 해결 할 수도 있다.
            
        * 예시 - 나이가 평균보다 많은 회원
            
            ```java          
            select m from Member m
            where m.age > (select avg(m2.age) from Member m2)
            ```

    * 서브쿼리 지원 함수
 
        * EXISTS
        
            * 문법 : `[NOT] EXISTS (subquery)`
            
            * 설명 : 서브쿼리에 결과가 존재하면 참이다. (NOT은 반대) 

            ```java          
            // 팀A에 소속인 회원
            select m from Member m 
            where exists ( select t from m.team t WHERE t.name = '팀A')
            ```

        * {ALL | ANY | SOME} 
        
            * 문법 : `{ALL | ANY | SOME} (subquery)`
            
            * 설명 : 비교 연산자와 같이 사용한다.
            
                * ALL : 조건을 모두 만족하면 참이다.
                
                * ANY, SOME: 둘은 같은 의미. 조건을 하나라도 만족하면 참이다.

            ```java          
            // 전체 상품 각각의 재고보다 주문량이 많은 주문들
            select o from Order o
            where o.orderAmount > ALL (select p.stockAmount from Product p)
          
            // 어떤 팀이든 팀에 소속된 회원 
            select m from Member m
            where m.team = ANY (select t from Team t)
            ```

        * IN
        
            * 문법 : `[NOT] IN (subquery)`
            
            * 설명 : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참이다.
            
* 조건식

    * 타입 표현 - 대소문자 구분하지 않음
    
        * 문자
        
            * 작은 따옴표 사이에 표현
            
            * 작음 따옴표를 표현하고 싶으면 작은 따옴표 2개('') 사용
            
            * 예시) `'HELLO'`, `'She''s'`
            
        * 숫자
        
            * L (Long 타입 지정)
            
            * D (Double 타입 지정)
            
            * F (Float 타입 지정)
            
            * 예시) `10L`, `10D`, `10F`

        * 날짜
        
            * DATE {d ‘yyyy-mm-dd’}
            
            * TIME {t ‘hh:mm:ss’}
            
            * TIMESTAMP {ts 'yyyy-mm-dd hh:mm:ss.f}
            
            * 예시) `m.createDate = {d ‘2021-01-03’}`
           
        * Boolean
        
            * TRUE, FALSE
            
        * ENUM
        
            * 패키지명을 포함한 전체 이름을 사용해야 한다.
            
            * 예시) `jpql.MemberType.ADMIN`
            
        * 엔티티 타입
        
            * 엔티티의 타입을 표현한다. 주로 상속 관계에서 사용한다.
            
            * 예시) `TYPE(m) = Member`
            
    * CASE 식
    
        * 기본 CASE 식
        
            * 문법
            
                ```java          
                CASE  
                    {WHEN <조건식> THEN <스칼라식>}+  
                    ELSE <스칼라식>  
                END
                ```

            * 예시
            
                ```java          
                select
                    case when m.age <= 10 then '학생요금' 
                         when m.age >= 60 then '경로요금'
                         else '일반요금'
                    end
                from Member m
                ```
              
        * 단순 CASE 식 
        
            * 문법
            
                ```java          
                CASE <조건대상>  
                    {WHEN <스칼라식1> THEN <스칼라식2>}+
                    ELSE <스칼라식>
                END
                ```

            * 예시
            
                ```java          
                select
                    case t.name
                        when '팀A' then '인센티브110%' 
                        when '팀B' then '인센티브120%'
                        else '인센티브105%'
                    end
                from Team t
                ```
              
        * COALESCE
        
            * 문법 : `COALESCE(<스칼라식>, {,<스칼라식>}+)`
            
            * 설명 : 스칼라식을 차례대로 조회해서 null이 아니면 반환한다.

            * 예시
            
                ```java
                // m.username이 null이 아니면 m.username을 반환하고, null이면 이름 없는 회원을 반환한다.
                select coalesce(m.username,'이름 없는 회원') from Member m
                ```
              
        * NULLIF
        
            * 문법 : `NULLIF(<스칼라식>, <스칼라식>)`
            
            * 설명 : 두 값이 같으면 null을 반환하고 다르면 첫번째 값을 반환한다.

            * 예시
            
                ```java
                // 사용자 이름이 '관리자'면 null을 반환하고 나머지는 본인의 이름을 반환한다. (관리자의 이름을 숨길 때 사용)
                select NULLIF(m.username, '관리자') from Member m
                ```

* JPQL 기본 함수

    * 문자 함수
    
        * `CONCAT(문자1, 문자2)` : 문자를 합친다.
        
            * `CONCAT('A', 'B')` = AB
            
        * `SUBSTRING(문자, 위치[, 길이])` : 위치부터 시작해 길이만큼 문자를 구한다. 길이 값이 없으면 나머지 전체 길이를 뜻한다
        
            * `SUBSTRING(‘ABCDEF’, 2, 3)` = BCD
            
        * `TRIM(문자)` : TRIM 문자를 제거한다. (LEADING : 왼쪽만, TRAILING : 오른쪽만, BOTH : 양쪽, TRIM 문자의 기본 값은 공백)
        
            * `TRIM('ABC')` = 'ABC'
            
        * `LOWER(문자)` : 소문자로 변경한다.
        
            * `LOWER('ABC')` = abc

        * `UPPER(문자)` : 대문자로 변경한다.
        
            * `UPPER('abc')` = ABC

        * `LENGTH(문자)` : 문자 길이를 알려준다.
        
            * `LENGTH('ABC')` = 3
            
        * `LOCATE(찾을 문자, 원본 문자[, 검색 시작 위치])` : 검색위치부터 문자를 검색한다. 1부터 시작하며 못 찾으면 0을 반환한다.
        
            * `LOCATE(‘DE’, ‘ABCDEFG’)` = 4

    * 수학함수
    
        * `ABS(수학식)` : 절대값을 구한다.
        
            * `ABS(-10)` = 10
            
        * `SQRT(수학식)` : 제곱근을 구한다
        
            * `SQRT(4)` = 2.0
            
        * `MOD(수학식, 나눌 수)` : 나머지를 구한다
        
            * `MOD(4, 3)` = 1
            
        * `SIZE(컬렉션 값 연관 경로식)` : 컬렉션의 크기를 구한다
        
            * `SIZE(t.members)`

    * 날짜함수
    
        * `CURRENT_DATE` : 현재 날짜
        
        * `CURRENT_TIME` : 현재 시간 
        
        * `CURRENT_TIMESTAMP` : 현재 날짜 + 시간

        * 하이버네이트는 날짜 타입에서 년,월,일,시간,분,초 값을 구하는 기능을 지원한다.
        
            ```java
            select year(CURRENT_TIMESTAMP), month(CURRENT_TIMESTAMP), day(CURRENT_TIMESTAMP) from Member;
            ```
 
* 사용자 정의 함수 호출

    * JPA는 사용자 정의 함수를 지원한다.
    
        * 하이버네이트를 사용하면 내가 사용하는 DB 방언 클래스를 상속받고 사용자 정의 함수를 등록한다.

            ```java
            public class MyH2Dialect extends H2Dialect{
          
                public MyH2Dialect(){
                    registerFunction("group_concat", new StandardFunction("group_concat", StandardBasicTypes.STRING));
                }
          
            }
            ```
          
        * 상속받은 방언 클래스를 `hibernate.dialect`에 등록한다.

            ```html
            <property name="hibernate.dialect" value="dialect.MyH2Dialect" />
            ```

        * 사용자 정의 함수는 다음과 같이 사용하면 된다.
        
            * JPA 문법

                ```java
                select function('group_concat', m.username) from Member m
                ```
              
            * 하이버네이트 문법

                ```java
                select group_concat(m.username) from Member m
                ```
              
* 경로 표현식

    * `경로 표현식`은 .(점)을 찍어 객체 그래프를 탐색하는 것을 말한다.
    
        ```java
        SELECT m.username     // 상태 필드
        FROM Member m
            JOIN m.team t     // 단일 값 연관 필드
            JOIN m.orders o   // 컬렉션 값 연관 필드
        WHERE t.name = '팀A';
        ```
      
    * 경로 표현식 용어 정리
    
        * 상태 필드(state field) : 단순히 값을 저장하기 위한 필드 
        
            * 예시) m.username
        
        * 연관 필드(association field) : 연관관계를 위한 필드
        
            * 단일 값 연관 필드 : 대상이 엔티티인 필드를 말한다. (@ManyToOne, @OneToOne)
            
                * 예시) m.team
            
            * 컬렉션 값 연관 필드 : 대상이 컬렉션인 필드를 말한다. (@OneToMany, @ManyToMany)   
          
                * 예시) m.orders
                
    * 경로 표현식 특징
    
        * 상태 필드 경로: 경로 탐색의 끝이다. 더는 탐색 할 수 없다.
        
        * 단일 값 연관 경로: 묵시적으로 내부 조인이 발생한다. 계속 탐색 할 수 있다.
        
        * 컬렉션 값 연관 경로: 묵시적으로 내부 조인이 발생한다. 더는 탐색 할 수 없다.
        
            * 컬렉션은 경로 탐색의 끝이다. 컬렉션에서 경로 탐색을 하고 싶으면 명시적 조인을 통해 별칭을 얻어야 한다.
            
            * **묵시적 조인 보다는 항상 명시적 조인을 사용하자. (묵시적 조인은 조인이 일어나는 상황을 파악하기 어려움)**
           
    * 경로 표현식 예시
    
        * 상태 필드 경로 탐색
        
            ```java
            select m.username from Member m
            ```
          
        * 단일 값 연관 경로 탐색
        
            ```java
            select m.team from Member m
            ```
          
        * 컬렉션 값 연관 경로
        
            ```java
            select t.members from Team t
            ```
          
            * t.members 처럼 컬렉션까지는 경로 탐색이 가능하다.
            
            * 하지만 t.members.username 처럼 컬렉션에서 경로 탐색을 시작하는 것은 허락하지 않는다.

            * 만약, 컬렉션에서 경로 탐색을 하고 싶으면, FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭으로 탐색 할 수 있다.

                ```java
                select m.username from Team t join t.members m
                ```
              
    * 명시적 조인, 묵시적 조인
    
        * `명시적 조인` : JOIN 키워드를 직접 사용하는 것
        
            * select m from Member m join m.team t
        
        * `묵시적 조인` : 경로 표현식에 의해 묵시적으로 조인이 발생하는 것 (내부 조인만 가능)
        
            * select m.team from Member m
            
* 페치 조인(fetch join)

    * `페치 조인`은 **연관된 엔티티나 컬렉션을 SQL 한 번에 같이 조회하는 기능**이다.
    
        * SQL 조인의 종류가 아니다.
        
        * JPQL에서 성능 최적화를 위해 제공하는 기능이다.
    
        * join fetch 명령어를 사용한다.
        
            * `[ LEFT [OUTER] | INNER ] JOIN FETCH` 조인경로
            
        * `N+1 문제`는 즉시 로딩, 지연 로딩 둘다에서 발생하는 문제다.
        
            * 해결 방법으로 페치 조인을 사용 할 수 있다.
            
    * 엔티티 페치 조인
    
        * 회원을 조회 하면서 연관된 팀도 함께 조회한다. (SQL 한 번에)
        
            ```java
            select m from Member m join fetch m.team
            ```
          
        * 사용 예시 - 다대일 관계
        
            ```java      
            String jpql = "select m from Member m join fetch m.team";
            List<Member> members = em.createQuery(jpql, Member.class)
                    .getResultList();
            
            for (Member member : members) {
                //페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩X
                System.out.println("username = " + member.getUsername() + ", " +
                                   "teamName = " + member.getTeam().getName());
            }
            ```
      
    * 컬렉션 페치 조인
    
        * 일대다 관계인 컬렉션에서도 페치 조인을 사용 할 수 있다.

            ```java        
            select t from Team t join fetch t.members where t.name = '팀A'
            ```

        * 사용 예시 - 일대다 관계

            ```java        
            String jpql = "select t from Team t join fetch t.members where t.name = '팀A'";
            List<Team> teams = em.createQuery(jpql, Team.class).getResultList();
    
            for(Team team : teams) {
                System.out.println("teamname = " + team.getName() + ", team = " + team);
    
                for (Member member : team.getMembers()) {
                    //페치 조인으로 팀과 회원을 함께 조회해서 지연 로딩이 발생하지 않는다.
                    System.out.println("-> username = " + member.getUsername() + ", member = " + member);
                }
            }
            ```
          
            * 일대다 조인은 결과는 데이터가 더 많아 질 수 있다. `[주의!]` 
            
            * 다대일 조인의 결과는 데이터가 같거나 적어질 수 있다. 
            
    * DISTINCT
    
        * JPQL의 DISTINCT는 SQL에 DISTINCT를 추가하고 애플리케이션에서 엔티티 중복을 제거한다.

            ```java        
            select distinct t
            from Team t join fetch t.members where t.name = '팀A'
            ```
          
            * SQL에 DISTINCT를 추가하더라도 데이터가 다르므로 SQL 결과에서는 중복 제거에 실패한다. (완전히 같아야 중복 제거 됨)
            
                * TEAM_ID: 1, TEAM_NAME : 팀A, MEMBER_ID : 1, TEAM_ID : 1, NAME : 회원1
                
                * TEAM_ID: 1, TEAM_NAME : 팀A, MEMBER_ID : 2, TEAM_ID : 1, NAME : 회원2
            
            * DISTINCT가 추가로 애플리케이션에서 중복 제거를 시도한다. 그리고 같은 식별자를 가진 Team 엔티티를 제거한다.
                        
    * 페치 조인과 일반 조인의 차이
    
        * 일반 조인 실행 시, 연관된 엔티티를 함께 조회하지 않는다.

            ```java        
            select t
            from Team t join t.members m where t.name = '팀A'
            ```
       
        * JPQL은 결과를 반환할 때 연관관계까지 고려하지 않는다. 단지 SELECT 절에 지정한 엔티티만 조회할 뿐이다.
            
            * 일반 조인 실행 예시처럼 팀 엔티티만 조회하고, 연관된 회원 컬렉션은 조회하지 않는다.
            
            * 만약 회원 컬렉션을 지연 로딩으로 설정하면 프록시나 아직 초기화하지 않은 컬렉션 래퍼를 반환한다.
    
        * 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회한다. (즉시 로딩) 
        
        * 즉, 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념이다. 
        
    * 페치 조인의 특징과 한계
    
        * 페치 조인 대상에는 별칭을 줄 수 없다. 
        
            * 하이버네이트는 가능하지만 가급적 사용하지 말자.
             
        * 둘 이상의 컬렉션은 페치 조인 할 수 없다.
        
            * `컬렉션 * 컬렉션`의 카테시안 곱이 만들어지므로 주의해야 한다.

        * 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.
        
            * 일대일,다대일 같은 단일 값 연관 필드들은 페치 조인을 사용해도 페이징 API를 사용 할 수 있다.
                 
            * 하이버네이트에서 컬렉션을 페치 조인하고 페이징 API를 사용하면 경고 로그를 남기면서 메모리에서 페이징 처리를 한다.
            
                * 성능 이슈와 메모리 초과 예외가 발생 할 수 있어서 매우 위험하다.

            * 해결 방법
            
                * 일대다 페치조인을 다대일 페치조인으로 뒤집어서 해결 할 수도 있다.
                
                    ```java        
                    select m from Member m join fetch m.team
                    ```
                  
                * 또는 `@BatchSize`를 사용하여 해결 할 수도 있다. (hibernate.default_batch_fetch_size로 글로벌 설정이 가능하다.)

    * 정리 
    
        * 연관된 엔티티들을 SQL 한 번으로 조회한다. - 성능 최적화 
        
        * 페치 조인은 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선적으로 적용한다.
        
            * @OneToMany(fetch = FetchType.LAZY) //글로벌 로딩 전략
             
        * 페치 조인은 객체 그래프를 유지 할 때 사용하면 효과적이다.
        
        * 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면, 
        
        * 페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적이다. 
        
            * Ex) 통계 쿼리

* 다형성 쿼리

    * `TYPE`은 엔티티의 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 사용한다.

        * 예) Item 중에 Book, Movie를 조회해라

            ```java        
            select i from Item i
            where type(i) IN (Book, Movie)
            ```

    * `TREAT`은 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.

        * JPA 표준은 FROM, WHERE 절에서 사용, 하이버네이트는 SELECT 절에서도 사용 가능

        * 예) 부모인 Item과 자식 Book이 있다.

            ```java        
            select i from Item i
            where treat(i as Book).author = 'kim'
            ```
          
* 엔티티 직접 사용

    * 기본 키 값

        * JPQL에서 엔티티를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.
        
            ```java        
            select count(m.id) from Member m //엔티티의 아이디를 사용
            select count(m) from Member m //엔티티를 직접 사용
            ```
          
        * JPQL의 파라미터로 엔티티를 직접 사용하더라도 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.
    
            ```java        
            String query = "select m from Member m where m = :member"; 
            List resultList = em.createQuery(query)
                                .setParameter("member", member)
                                .getResultList();
            ```
          
    * 외래 키 값

        * 기본 키 값이 1L인 팀 엔티티를 파라미터로 사용하고 있다. m.team은 현재 team_id라는 외래 키와 매핑되어 있다.
        
            ```java        
            Team team = em.find(Team.class, 1L);
          
            String qlString = "select m from Member m where m.team = :team"; 
            List resultList = em.createQuery(qlString)
                                .setParameter("team", team) .getResultList();
            ```

* Named 쿼리: 정적 쿼리

    * JPQL 쿼리는 동적 쿼리와 정적 쿼리로 나눌 수 있다.

        * 동적 쿼리: 검색 조건에 따라 실행 시점에 쿼리를 생성하는 것을 말한다. 
        
            * `em.createQuery("select ...")`

        * 정적 쿼리: 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용하는 것을 말한다. `Named 쿼리`라고도 한다.
        
            * 애플리케이션 로딩 시점에 JPQL 문법을 체크하여 미리 SQL로 파싱해둔다.
            
            * 따라서 오류를 빨리 확인 할 수 있고 사용하는 시점에는 파싱된 결과를 재사용하므로 성능상 이점이 있다.
            
                ```java        
                @Entity
                @NamedQuery(
                    name = "Member.findByUsername",
                    query="select m from Member m where m.username = :username")
                public class Member {
                ...
                }
              
                List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                                            .setParameter("username", "회원1")
                                            .getResultList();
                ```
              
            * 스프링 데이터 JPA에서는 `@Query`를 지원한다.
        
* 벌크 연산

    * 벌크 연산은 JPQL로 한 번에 여러 데이터를 수정하거나 삭제하는 것을 말한다. (SQL의 UPDATE, DELETE 문)

        * UPDATE 벌크 연산
        
            ```java
            // 재고가 10개 미만인 모든 상품의 가격을 10% 상승한다.
            String query = "update Product p " +
                "set p.prce = p.price * 1.1 " +
                "where p.stockAmount < :stockAmount";
            
            int resultCount = em.createQuery(query)
                                .setParameter("stockAmount", 10)
                                .executeUpdate();
            ```
          
            * 벌크 연산은 executeUpdate()를 사용한다. 벌크 연산으로 영향을 받은 엔티티 건수를 반환한다.

        * DELETE 벌크 연산
        
            ```java
            // 가격이 100원 미만인 상품을 삭제한다.
            String query = "delete from Product p " +
                           "where p.price < :price";
            
            int resultCount = em.createQuery(query)
                                .setParameter("price", 100)
                                .executeUpdate();
            ```
          
    * 벌크 연산의 주의사항

        * **벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다**는 점에 주의해야 한다.

        * 해결책은 다음 2가지가 있다.
        
            * 벌크 연산을 가장 먼저 실행한다. 
            
            * 벌크 연산을 수행한 후, 바로 영속성 컨텍스트를 초기화한다.
            
                * 영속성 컨텍스트를 초기화하면 이후 엔티티를 조회할 때, 벌크 연산이 적용된 데이터베이스에서 엔티티를 조회한다.
            
    * 스프링 데이터 JPA에서는 `@Modifying`를 지원한다.