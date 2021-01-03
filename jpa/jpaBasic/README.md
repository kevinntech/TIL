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

        * N+1 문제 : 처음에 쿼리 1개를 실행 했더니 추가 쿼리 N개가 실행되는 문제를 말한다.

        * N+1 문제 해결 방안 (연관된 엔티티를 함께 조회해야 되는 경우)
        
            * JPQL의 fetch 조인을 이용한다.

            * 또는 `@EntityGraph`를 이용한다.
            
            * `@BatchSize + EAGER LOADING`를 이용 할 수도 있다. (추천 X)

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
    
* JPQL 예시

    * `select m from Member as m where m.age > 18`
    
        * 엔티티와 속성은 대소문자를 구분한다. (Member, age)
        
        * JPQL 키워드는 대소문자를 구분하지 않는다. (SELECT, FROM, where)
        
        * 테이블 이름이 아닌 엔티티 이름을 사용한다. (Member) 
        
        * 별칭이 필수다. (m) 
        
        * as는 생략 가능하다.

* TypeQuery, Query

    * `TypeQuery` : 반환 타입이 명확할 때 사용한다.
    
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
    








        

        

      
