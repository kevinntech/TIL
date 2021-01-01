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
      
    * 값 타입 컬렉션 사용 예시
    
        * 해당 강좌를 참고하자.
        

      
