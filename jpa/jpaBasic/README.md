# 김영한님의 자바 ORM 표준 JPA 프로그래밍 - 기본편
> 아래 내용은 [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic "자바 ORM 표준 JPA 프로그래밍 - 기본편") 강좌를 정리한 내용 입니다.

## 1. JPA 소개

### 1. SQL 중심적인 개발의 문제점

* 데이터베이스에 객체를 저장하려면 반복적인 SQL 문을 작성해야 한다.
  
* SQL에 의존적인 개발을 피하기 어렵다.

### 2. 패러다임의 불일치

* 상속

    * 객체는 상속관계가 있지만, 관계형 데이터베이스는 상속 관계가 없다.
    
    * 객체는 상속이라는 기능을 가지고 있지만 테이블은 상속이라는 기능이 없다.

* 연관관계

    * 객체는 참조를 사용해서 연관된 객체를 조회한다.

    * 반면 테이블은 기본 키(PK) 또는 외래 키(FK)를 사용해서 조인을 한 다음, 연관된 테이블을 조회한다.

### 3. JPA (Java Persistence API)

* JPA란?

    * `JPA(Java Persistence API)`는 자바 진영의 ORM 기술 표준이다.

    * JPA는 애플리케이션과 JDBC 사이에서 동작한다.

        ![image 19](images/img19.png)

        * JAVA 애플리케이션에서 JPA에게 명령을 하면, JPA는 JDBC API를 사용해서 SQL을 만든 다음, DB에 전달한다. 
        
        * 그리고 DB로 부터 그 결과를 받는다.
        
    * JPA는 패러다임의 불일치를 해결한다.

* ORM (Object-Relational Mapping, 객체 관계 매핑)

    * `ORM`는 객체와 관계형 데이터베이스를 매핑(연결) 해주는 것을 말한다.

* JPA는 표준 명세

    * JPA는 자바 ORM 기술에 대한 API 표준 명세를 말한다.

        * 즉, 인터페이스를 모아둔 것이다.

    * JPA의 대표적인 구현체로는 하이버네이트가 있다.
  
* JPA를 사용하는 이유

    * 생산성을 증가 시킨다.
    
        * CRUD가 이미 정의 되어 있다.
        
            * 저장: jpa.persist(member)
            
            * 조회: Member member = jpa.find(memberId)
            
            * 수정: member.setName("변경할 이름")
            
            * 삭제: jpa.remove(member)
       
    * 유지보수 해야 하는 코드 수가 줄어든다.

        * 기존에는 필드를 변경하면 모든 SQL 문을 수정 했었지만 JPA를 사용하면 필드만 추가하면 된다. (DB에 컬럼이 추가되어 있는 경우)

    * 패러다임의 불일치 해결
    
    * 성능 최적화 기능

        * ① 1차 캐시와 동일성(identity) 보장
    
            * 같은 트랜잭션 안에서는 같은 엔티티를 반환 – 약간의 조회 성능 향상
    
            * DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장

                ```java
                String memberId = "100";
                Member m1 = jpa.find(Member.class, memberId); // SQL
                Member m2 = jpa.find(Member.class, memberId); // 캐시
                
                println(m1 == m2) // true
                ```
              
                * 위의 코드에서 SQL은 1번만 실행된다.

        * ② 트랜잭션을 지원하는 쓰기 지연 - INSERT
    
            * 트랜잭션을 커밋 할 때까지 INSERT SQL을 모음
    
            * JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송

        * ③ 지연 로딩과 즉시 로딩
    
            * `지연 로딩(LAZY LOADING)` : 연관된 엔티티를 실제 사용할 때 조회한다.
    
            * `즉시 로딩(EAGER LOADING)` : 엔티티를 조회할 때, 연관된 엔티티도 함께 조회한다.

## 2. JPA 시작

### 1. 데이터베이스 방언

* JPA는 특정 데이터베이스에 종속적이지 않다.

* 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다르다.

    * ① 데이터 타입 : 가변 문자 타입으로 MySQL은 VARCHAR, Oracle은 VARCHAR2를 사용
    
    * ② 다른 함수명 : 문자열을 자르는 함수로 SQL 표준은 SUBSTRING()를 사용하지만 Oracle은 SUBSTR()를 사용
    
    * ③ 페이징 처리 : MySQL은 LIMIT을 사용하지만 Oracle은 ROWNUM을 사용

* `방언(Dialect)` : SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능을 말한다.

* **하이버네이트를 포함한 대부분의 JPA 구현체들은 특정 데이터베이스에 종속되지 않도록 다양한 데이터베이스 방언 클래스를 제공한다.**

    * 하이버네이트는 다양한 데이터베이스 방언을 제공한다.
    
        * ① H2 : `org.hibernate.dialect.H2Dialect`
        
        * ② Oracle 10g : `org.hibernate.dialect.Oracle10gDialect`
        
        * ③ MySQL : `org.hibernate.dialect.MySQL5InnoDBDialect`
        
### 2. 애플리케이션 개발

* (1) 엔티티 매니저 설정

    * ① JPA는 `persistence.xml`이라는 설정 정보를 읽어서 엔티티 매니저 팩토리를 생성한다.
    
        * `EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");`
        
            * `META-INF/persistence.xml`에서 이름이 hello인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성한다.

    * ② 엔티티 매니저 팩토리에서 엔티티 매니저를 생성한다.
    
        * `EntityManager em = emf.createEntityManager();`
        
            * 엔티티 매니저를 사용해서 엔티티를 데이터베이스에 등록/조회/수정/삭제를 할 수 있다.
            
            * 엔티티 매니저는 내부에 데이터소스(데이터베이스 커넥션)을 유지하면서 데이터베이스와 통신한다.
        
            * `DataSource` : 커넥션 풀의 Connection을 관리하기 위한 객체이다. DataSource 객체를 통해서 필요한 Connection을 획득, 반납 등의 작업을 한다.
        
    * ③ 마지막으로 사용이 끝난 엔티티 매니저는 반드시 종료해야 한다.
    
        * `em.close(); // 엔티티 매니저 종료`
        
    * 애플리케이션을 종료할 때, 엔티티 매니저 팩토리도 종료해야 한다.
    
        * `emf.close(); // 엔티티 매니저 팩토리 종료`
        
* (2) 트랜잭션 관리

    * JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 한다.
    
        * ① 트랜잭션을 시작하려면 엔티티 매니저에서 트랜잭션 API를 받아와야 한다.
      
        * ② 트랜잭션 API를 사용해서 비즈니스 로직이 정상 동작하면 트랜잭션을 커밋(commit)하고 예외가 발생하면 트랜잭션을 롤백(rollback)한다.

            ```java
            EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        
            EntityManager em = emf.createEntityManager();
        
            EntityTransaction tx = em.getTransaction(); // 트랜잭션 API
        
            tx.begin(); // 트랜잭션 시작
        
            try{
                // 비즈니스 로직 작성
                tx.commit(); // 트랜잭션 커밋
            }catch (Exception e){
                tx.rollback(); // 예외 발생 시 트랜잭션 롤백
            }finally {
                em.close(); // 엔티티 매니저 종료
            }
        
            emf.close(); // 엔티티 매니저 팩토리 종료
            ```

* (3) 비즈니스 로직 작성

    * ① 등록
    
        * `em.persist()` : 엔티티를 등록한다.
    
    * ② 수정
    
        * JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 가지고 있다. 따라서 엔티티의 값만 변경하면 된다.
        
        * `member.setName("HelloB");`
        
    * ③ 삭제
    
        * `em.remove()` : 엔티티를 삭제한다.
        
    * ④ 한 건 조회
    
        * `em.find()` : 엔티티 하나를 조회한다.
        
        * 메소드의 파라미터로 조회할 `엔티티 타입`과 `엔티티의 식별자 값`을 전달한다.

* (4) 주의사항    

    * ① `EntityManagerFactory`는 하나만 생성해서 애플리케이션 전체에서 공유해서 사용해야 한다.
    
    * ② `EntityManager`는 쓰레드 간에 공유해서는 안 된다.
    
        * 보통, `EntityManager`는 데이터베이스 트랜잭션 단위로 만들며 트랜잭션이 끝날 때, 같이 종료시킨다. 
    
    * ③ JPA의 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.

* (5) JPQL(Java Persistence Query Language)

    * JPA는 SQL을 추상화한 `JPQL`이라는 객체 지향 쿼리 언어를 제공한다.

    * SQL과 문법 유사해서 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등을 지원한다.

    * `JPQL`은 엔티티 객체를 대상으로 쿼리한다.
    
        * -> SQL에 의존적인 개발을 피할 수 있다.

## 3. 영속성 관리

* (1) 영속성 컨텍스트(persistence context)

    * `영속성 컨텍스트`는 **엔티티를 영구 저장하는 환경**을 말한다.
    
    * 영속성 컨텍스트는 논리적인 개념이며 눈에 보이지 않는다.
    
    * 엔티티 매니저를 통해서 영속성 컨텍스트에 접근한다.

* (2) 엔티티의 생명주기

    * 엔티티에는 4가지 상태가 존재한다.

        * ① `비영속(new/transient)` : 영속성 컨텍스트와 전혀 관계가 없는 **새로운** 상태
    
        * ② `영속(managed)` : 영속성 컨텍스트에 **저장**된 상태
    
        * ③ `준영속(detached)` : 영속성 컨텍스트에 저장되었다가 **분리**된 상태
    
        * ④ `삭제(removed)` : **삭제**된 상태

* (3) 엔티티의 각 상태에 대해 자세히 알아보기

    * ① 비영속(new)
    
        * 엔티티 객체를 생성한 상태를 말한다.
          
        * 아직 영속성 컨텍스트나 데이터베이스와는 전혀 관계가 없는 상태다.
        
            ```java
            //객체를 생성한 상태 (비영속)
            Member member = new Member();
            member.setId("member1");
            member.setUsername("회원1");
            ```
          
    * ② 영속(managed)
    
        * 엔티티 매니저를 통해서 엔티티를 영속성 컨텍스트에 저장하면 영속성 컨텍스트가 엔티티를 관리하게 되며 이를 영속 상태라 한다.
               
            ```java
            EntityManager em = emf.createEntityManager();
            em.getTransaction().begin();
            
            //객체를 저장한 상태(영속)
            em.persist(member);
            ```
          
    * ③ 준영속
    
        * 영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다.
               
            ```java
            //회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
            em.detach(member);
            ```
          
    * ④ 삭제
    
        * 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다.
               
            ```java
            //객체를 삭제한 상태(삭제)
            em.remove(member);
            ```

* (4) 영속성 컨텍스트의 이점

    * 1차 캐시
    
        * `1차 캐시`는 영속성 컨텍스트가 내부에 가지고 있는 캐시를 말한다. 
        
        * 영속 상태의 엔티티는 모두 이곳에 저장된다. 
        
        * 쉽게 이야기하면, 영속성 컨텍스트 내부에 Map이 하나 있는데 키는 `@Id`로 매핑한 식별자고 값은 엔티티 인스턴스다.
        
    * 엔티티의 `동일성(identity)`을 보장한다.
    
    * 트랜잭션을 지원하는 쓰기 지연이 가능하다.

        * `트랜잭션을 지원하는 쓰기 지연(transactional write-behind)`  
    
            * 엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 쓰기 지연 SQL 저장소에 INSERT SQL을 차곡 차곡 모아둔다.
        
            * 그리고 트랜잭션을 커밋 할 때 모아둔 쿼리를 데이터베이스에 보내는 것을 말한다.

    * 변경 감지(Dirty Checking)
        
        * 변경 감지는 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능이다.
    
    * 지연 로딩(Lazy Loading)
    
* (5) 영속성 컨텍스트의 이점 - CRUD

    * 엔티티 조회
    
        * 다음 코드를 실행하면 회원(member) 엔티티는 영속 상태가 되며 1차 캐시에 회원 엔티티를 저장한다. 회원 엔티티는 아직 데이터베이스에 저장되지 않았다.

            ```java
            // 엔티티를 생성한 상태(비영속)
            Member member = new Member();
            member.setId("member1");
            member.setUsername("회원1");
            
            // 엔티티를 영속
            em.persist(member);
            ```
          
            * `em.find()`를 호출하면 먼저 1차 캐시에서 엔티티를 찾고 만약 찾는 엔티티가 1차 캐시에 없으면 데이터베이스에서 조회한다.
          
        * 1차 캐시에서 조회하는 경우
        
            ```java            
            //1차 캐시에서 조회
            Member findMember = em.find(Member.class, "member1");
            ```
          
            * ① `em.find()`를 호출하면 먼저 1차 캐시에서 식별자 값으로 엔티티를 찾는다.

            * ② 만약 찾는 엔티티가 있으면 데이터베이스를 조회하지 않고 메모리에 있는 1차 캐시에서 엔티티를 조회한다.
            
        * 데이터베이스에서 조회
        
            ```java            
            //데이터베이스에서 조회
            Member findMember2 = em.find(Member.class, "member2");
            ```
          
            * ① `em.find(Member.class, "member2")`를 실행한다.
              
            * ② member2가 1차 캐시에 없으므로 데이터베이스에서 조회한다.
              
            * ③ 조회한 데이터로 member2 엔티티를 생성해서 1차 캐시에 저장한다. (영속 상태)
              
            * ④ 영속 상태의 엔티티를 반환한다.
            
        * 영속 엔티티의 동일성 보장
        
            ```java            
            Member a = em.find(Member.class, "member1");
            Member b = em.find(Member.class, "member1");
            
            System.out.println(a == b); // 동일성 비교
            ```
          
            * `em.find(Member.class, "member1")`를 반복해서 호출해도 영속성 컨텍스트는 1차 캐시에 있는 같은 엔티티 인스턴스를 반환한다.
        
            * 영속성 컨텍스트는 엔티티의 동일성을 보장한다.
    
                * 동일성(identity) 비교
                
                    * `동일성`은 실제 인스턴스가 같다는 것을 의미한다. 

                    * 따라서 인스턴스의 참조 값을 비교하는 `==` 비교의 결과가 같다는 것을 의미한다.
         
                * 동등성(equality) 비교
                
                    * `동등성`은 실제 인스턴스는 다를 수 있지만 인스턴스가 가지고 있는 값은 같다는 것을 의미한다.
                    
                    * 따라서 인스턴스가 가지고 있는 값을 비교하는 `equals()`의 결과가 같다는 것을 의미한다.
    
    * 엔티티 등록
    
        *  엔티티 매니저를 사용해서 엔티티를 영속성 컨텍스트에 등록해보자.

            ```java            
            EntityManager em = emf.createEntityManager();
            EntityTransaction transaction = em.getTransaction();
            
            //엔티티 매니저는 데이터 변경 시 트랜잭션을 시작해야 한다.
            transaction.begin(); // [트랜잭션] 시작
            
            em.persist(memberA);
            //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
            
            //커밋하는 순간, 데이터베이스에 INSERT SQL을 보낸다.
            transaction.commit(); // [트랜잭션] 커밋
            ```
           
        * ① `em.persist(memberA);`를 실행하면 다음과 같이 동작한다.
        
            * 영속성 컨텍스트는 1차 캐시에 회원 엔티티(memberA)를 저장하면서 동시에 회원 엔티티 정보로 INSERT 쿼리를 만든다.
            
            * 그리고 만들어진 INSERT 쿼리를 `쓰기 지연 SQL 저장소`에 보관한다.
            
        * ② 트랜잭션을 커밋하면 엔티티 매니저는 영속성 컨텍스트를 플러시 하여 `쓰기 지연 SQL 저장소`에 모인 쿼리를 데이터베이스에 보내고 실제 데이터베이스 트랜잭션을 커밋한다. 

    * 엔티티 수정
    
        * JPA로 엔티티를 수정할 때는 단순히 엔티티를 조회해서 데이터만 변경하면 된다.
        
            ```java            
            // 영속 엔티티 조회
            Member memberA = em.find(Member.class, "memberA");
            
            // 영속 엔티티 데이터 수정
            memberA.setUsername("hi");
            memberA.setAge(10);
            
            //em.update(member) 이런 코드가 있어야 하지 않을까? 생각 할 수도 있다. 그렇지만 필요 X
            
            transaction.commit(); // [트랜잭션] 커밋
            ```
          
            * ① 트랜잭션을 커밋하면 엔티티 매니저 내부에서 먼저 플러시(`flush()`)가 호출된다.
            
            * ② 엔티티와 스냅샷을 비교해서 변경된 엔티티를 찾는다.
            
                * `스냅샷`은 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장해두는 것을 말한다.
    
            * ③ 변경된 엔티티가 있으면 수정 쿼리를 생성해서 `쓰기 지연 SQL 저장소`에 보낸다.

            * ④ `쓰기 지연 저장소`의 SQL을 데이터베이스에 보낸다.

            * ⑤ 실제 데이터베이스 트랜잭션을 커밋한다.

    * 플러시(flush)
    
        * `플러시`는 **영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것**을 말한다.

            * 영속성 컨텍스트에 보관된 엔티티를 지우는 것이 아닌 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 것이다
      
        * **플러시의 동작 과정** 

            * ① **변경 감지가 동작**해서 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 변경된 엔티티를 찾는다.

                * **변경된 엔티티는 쿼리를 만들어 `쓰기 지연 SQL 저장소`에 등록한다.**

            * ② **그리고 `쓰기 지연 SQL 저장소`의 쿼리를 데이터베이스에 전송한다.** (등록, 수정, 삭제 쿼리)
            
        * **영속성 컨텍스트를 플러시 하는 방법**

            * ① `em.flush()`를 직접 호출한다.

            * ② **트랜잭션 커밋 시 플러시가 자동 호출**된다.

            * ③ **JPQL 쿼리 실행 시 플러시가 자동 호출**된다.
            
    * 준영속 상태
    
        * `준영속 상태(detached)`는 영속 상태의 엔티티가 영속성 컨텍스트에서 분리 된 것을 말한다.
        
        * 따라서 **준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.**

        * **준영속 상태로 만드는 방법**

            * ① `em.detach(entity)` : 특정 엔티티만 준영속 상태로 전환한다.
            
                * detach()를 호출하는 순간 1차 캐시 부터 쓰기 지연 SQL 저장소까지 해당 엔티티를 관리하기 위한 모든 정보가 제거된다. 
    
            * ② `em.clear()` : 영속성 컨텍스트를 완전히 초기화한다.
    
            * ③ `em.close()` : 영속성 컨텍스트를 종료한다.

    * 엔티티 삭제
    
        * 엔티티를 삭제하려면 **먼저 삭제 대상 엔티티를 조회**한 다음, `em.remove()`**에 삭제 대상 엔티티를 전달하면 된다.**
        
            ```java            
            //삭제 대상 엔티티 조회
            Member memberA = em.find(Member.class, "memberA");
            
            em.remove(memberA); //엔티티 삭제
            ```

## 4. 엔티티 매핑

* 객체와 테이블 매핑

    * (1) @Entity
    
        * `@Entity`는 JPA가 관리하는 "엔티티"로 만든다. 
        
            * 테이블과 매핑 할 클래스에 붙여준다.
        
            * [주의사항] JPA는 엔티티 객체를 생성할 때, 기본 생성자를 사용하므로 기본 생성자가 반드시 있어야 한다.

    * (2) @Table
    
        * `@Table`은 엔티티와 매핑할 테이블을 지정한다.
        
            * `@Table`을 생략하면 엔티티 이름을 테이블 이름으로 매핑한다.

            * uniqueConstraints 속성은 DDL 생성 시에 유니크 제약조건을 만든다.

                ```java            
                @Entity
                @Table( name = "MEMBER", 
                        uniqueConstraints = {@UniqueConstraint(name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"})}
                )
                public class Member {
                
                    @Id @GeneratedValue
                    @Column(name = "ID")
                    private String id;
                
                    @Column(name = "NAME")
                    private String userName;
                
                    private int age;
                }
                ```
    
                * 2개 이상의 복합 유니크 제약조건도 만들 수 있다.

    * (3) 데이터베이스 스키마 자동 생성
    
        * JPA는 애플리케이션 실행 시점에 DDL을 자동으로 생성하는 기능을 제공한다.
        
        * 이렇게 생성된 DDL은 개발 장비에서만 사용해야 한다. 또는 적절히 다듬은 후 사용한다.

            * `hibernate.hbm2ddl.auto` 속성
            
                * `create` : 기존 테이블을 삭제하고 새로 생성한다. 
                
                * `create-drop` : create와 같으나 애플리케이션을 종료할 때, 생성한 테이블을 DROP 한다.
                
                * `update` : 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경사항만 반영한다. (운영 DB에 사용하면 안됨)
                
                * `validate` : 엔티티와 테이블이 정상적으로 매핑 되었는지만 확인한다.
                
                * `none` : 자동 생성 기능을 사용하지 않는다.
            
        * DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.
        
* 필드와 컬럼 매핑

    * (1) @Column
    
        * `@Column`는 필드를 테이블 컬럼에 매핑한다.
        
            * name 속성은 필드와 매핑할 테이블의 컬럼 이름을 지정한다.

            * nullable 속성은 null 값의 허용 여부를 지정한다.
    
            * unique 속성은 하나의 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.

                ```java            
                @Column(unique = true)
                private String username;
                ```
              
                * 두 개 이상의 컬럼에 유니크 제약조건을 걸 때는 클래스 레벨에서 `@Table.uniqueConstraints`를 사용해야 한다. 
    
            * length 속성은 문자 길이 제약조건을 지정할 때 사용한다.

                ```java            
                @Column(length = 100)
                private String data;
                ```

    * (2) @Enumerated
    
        * `@Enumerated`는 자바의 enum 타입을 매핑한다.
        
            * `EnumType.ORDINAL`이 아닌 `EnumType.STRING`를 사용해야 한다.

    * (3) @Temporal
    
        * `@Temporal`는 날짜 타입을 매핑한다.
        
        * 자바의 LocalDate, LocalDateTime을 사용하면 `@Temporal`을 생략 할 수 있다.
        
    * (4) @Lob
    
        * `@Lob`는 BLOB, CLOB 타입을 매핑한다.
        
        * 매핑하는 필드 타입이 문자면 `CLOB`으로 매핑하고 나머지는 `BLOB`으로 매핑한다.

            * CLOB: String, char[], java.sql.CLOB
    
            * BLOB: byte[], java.sql.BLOB

    * (5) @Transient
    
        * `@Transient`는 특정 필드를 데이터베이스에 매핑하지 않는다.
          
* 기본 키 매핑

    * (1) 기본 키 매핑 애노테이션
    
        * ① `@Id` : 필드를 테이블의 기본 키(Primary Key)에 매핑 한다.
    
            * `@Id`가 사용된 필드를 식별자 필드라 한다.
    
        * ② `@GeneratedValue` : 기본 키 자동 생성 전략을 지정한다.
        
    * (2) 기본 키 매핑 방법
    
        * **기본 키를 직접 할당**하려면 `@Id`만 사용한다.
    
        * **기본 키를 자동 생성**하려면 `@Id`에 `@GeneratedValue`를 추가하고 원하는 키 생성 전략을 선택한다.

            * ① `IDENTITY` 전략
    
                * 기본 키 생성을 데이터베이스에 위임하는 전략이다.
        
                * 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용
    
                * 해당 전략은 `트랜잭션을 지원하는 쓰기 지연`이 동작하지 않는다.
                
                    * IDENTITY 전략은 먼저 엔티티를 데이터베이스에 저장한 후에 식별자를 조회해서 엔티티의 식별자에 할당한다.

            * ② `SEQUENCE` 전략
    
                * 데이터베이스 시퀀스 오브젝트를 사용해서 기본 키를 생성한다. 
                
                    * 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트다.
        
                * 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용할 수 있다.
        
                * `@SequenceGenerator`이 필요하다.
                
                    * JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 `allocationSize`를 사용한다.
                    
                    * 여기에 설정한 값만큼 한 번에 시퀀스 값을 증가시키고 나서 그만큼 메모리에 시퀀스 값을 할당한다.
                
                * 해당 전략은 `트랜잭션을 지원하는 쓰기 지연`이 동작한다.
                
                    * SEQUENCE 전략은 em.persist()를 호출할 때 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회한다.
    
                    * 그리고 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다.
    
                    * 이후 트랜잭션을 커밋해서 플러시가 일어나면 엔티티를 데이터베이스에 저장한다.
    
            * ③ `TABLE` 전략
    
                * 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다.
                
                    * 해당 전략은 실무에서 거의 사용되지 않는다.
        
                * TABLE 전략의 장, 단점
                
                    * 장점: 모든 데이터베이스에 적용 가능
    
                    * 단점: 성능이 좋지 않음
                
                * `@TableGenerator`이 필요하다.
    
            * ④ `AUTO` : 선택한 데이터베이스 방언에 따라 자동으로 지정한다. (기본 값)
            
    * (3) 권장하는 식별자 전략
    
        * **데이터베이스 기본 키는 다음 3가지 조건을 모두 만족해야 한다.**
        
            * **① null 값은 허용하지 않는다.**
            
            * **② 유일해야 한다.**
            
            * **③ 변하면 안 된다.**
            
        * 테이블의 기본 키를 선택하는 전략은 크게 2가지가 있다.
                    
            * ① `자연 키(natural key)` : 비즈니스에 의미가 있는 키
    
                * 예 : 주민등록번호, 이메일, 전화번호

            * ② `대리 키(surrogate key)` : 비즈니스와 관련 없는 임의로 만들어진 키, 대체 키로도 불린다.

                * 예 : 오라클 시퀀스, auto_increment, 키 생성 테이블 사용
                
            * 미래까지 위에 있는 조건을 만족하는 자연 키는 찾기 어렵다. **대리 키(대체 키)를 사용하자.**
            
            * 즉, **타입은 Long형으로 하고 대체 키를 기본 키로 사용**하되 **주민등록번호나 이메일처럼 자연 키의 후보가 되는 컬럼들은**
              
            * **필요에 따라 유니크 인덱스를 설정해서 사용**하는 것을 권장한다.

## 5. 연관관계 매핑 기초

* 연관관계가 필요한 이유

    * 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 객체들의 협력 관계를 만들 수 없다.

    * 테이블과 객체 사이에는 다음과 같은 차이가 있다.
    
        * 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다.
    
        * 객체는 참조를 사용해서 연관된 객체를 찾는다.
         
* 단방향 연관관계

    * (1) 객체 지향 모델링 (객체 연관관계 사용)
    
        ![image 1](images/img1.png) 
    
        * ① 객체 연관관계

            * 회원(`Member`) 객체는 `Member.team` 필드로 팀(`Team`) 객체와 연관관계를 맺는다.
    
            * 회원 객체와 팀 객체는 단방향 관계다.
       
                * 회원은 `Member.team` 필드를 통해서 팀을 알 수 있지만, 반대로 팀은 회원을 알 수 없다.

        * ② 테이블 연관관계

            * 회원(`MEMBER`) 테이블은 `TEAM_ID` 외래 키로 팀(`TEAM`) 테이블과 연관관계를 맺는다.

            * 회원 테이블과 팀 테이블은 양방향 관계다. 
            
            * 회원 테이블의 `TEAM_ID` 외래 키를 통해서 회원과 팀을 조인 할 수 있고 반대로 팀과 회원도 조인할 수 있다.

                ```sql            
                 SELECT *
                 FROM MEMBER M JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
              
                 SELECT *
                 FROM TEAM T JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
                ```
          
    * (2) 객체의 참조와 테이블의 외래 키를 매핑한다.
    
        * `연관관계 매핑`은 객체의 참조(`Member.team`)와 테이블의 외래 키(`MEMBER.TEAM_ID`)를 매핑하는 것이다.
          
        * `연관관계 매핑`은 어떤 연관관계 인지(`@ManyToOne`) 그리고 연관 관계를 매핑 할 때, 사용 할 외래 키(`@JoinColumn`)를 지정한다.

            ```java            
            @Entity
            public class Member{
                @Id @GeneratedValue
                @Column(name = "MEMBER_ID")
                private Long id;
          
                @Column(name = "USER_NAME")
                private String username;
          
                @ManyToOne
                @JoinColumn(name = "TEAM_ID")
                private Team team;
            }
            ```

            ```java            
            @Entity
            public class Team{
                @Id @GeneratedValue
                @Column(name = "TEAM_ID")
                private Long id;
          
                private String name;
            }
            ```
          
            * `@ManyToOne` : 다대일(N:1) 관계라는 매핑 정보다.
              
                * 회원과 팀은 다대일 관계다.
            
            * `@JoinColumn` : 연관관계를 매핑 할 때, 사용 할 외래 키를 지정한다.
            
                * name 속성에는 매핑 할 외래 키 이름을 지정한다.
            
                * 회원과 팀 테이블은 `TEAM_ID` 외래 키로 연관관계를 맺으므로 이 값을 지정하면 된다.
            
                * 해당 애노테이션은 생략 할 수 있다.

    * (3) 객체–관계 매핑 (ORM)
    
        * 앞서 살펴본 코드를 실행 했을 때, 객체(`Member`)의 참조와 테이블(`MEMBER`)의 외래 키를 매핑하면 다음 그림과 같다.
        
            ![image 2](images/img2.png)
            
    * (4) 연관관계를 매핑한 엔티티를 저장한다.

        ```java
        // 팀 저장            
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);
        
        // 회원 저장
        Member member = new Member();
        member.setUsername("member1");
        member.setTeam(team); // 회원 -> 팀 참조
        em.persist(member); // 저장
        ```
        
        * 회원 엔티티가 팀 엔티티를 참조하도록 하고 회원 엔티티를 영속성 컨텍스트에 저장 했다.
        
        * JPA는 회원이 참조하는 팀의 식별자(`Team.id`)를 외래 키로 사용해서 적절한 등록 쿼리를 생성한다.

            ```sql
            INSERT INTO TEAM (TEAM_ID, NAME) VALUES (1, ‘TeamA’);
            
            INSERT INTO MEMBER (MEMBER_ID, USERNAME, TEAM_ID) VALUES (2, ‘member1’, 1);
            ```
        
    * (5) 연관 관계가 있는 엔티티를 조회한다.
    
        ```java
        // 조회
        Member findMember = em.find(Member.class, 2L);
        
        // 참조를 사용해서 연관 관계 조회
        Team findTeam = findMember.getTeam();
        ```

        * `findMember.getTeam()`을 사용해서 findMember와 연관된 team 엔티티를 조회 할 수 있다.
        
    * (6) 연관 관계를 수정한다.
    
        ```java
        // 새로운 팀 B
        Team teamB = new Team();
        teamB.setName("TeamB");
        em.persist(teamB);
      
        // member1에 새로운 팀 B를 설정한다.
        findMember.setTeam(teamB);
        ```
      
        * TeamA 소속이던 회원을 새로운 팀 TeamB에 소속되도록 수정한다. 

* 양방향 연관관계와 연관관계의 주인
        
    * (1) 양방향 연관관계

        ![image 3](images/img3.png) 
        
        * ① 양방향 객체 연관관계
          
            * 회원에서 팀으로 접근하고 반대 방향인 팀에서도 회원으로 접근 할 수 있는 양방향 연관관계로 매핑한다.
              
            * 회원과 팀은 다대일 관계다. 반대로 팀에서 회원은 일대다 관계다.
            
                * 일대다 관계는 여러 건과 연관관계를 맺을 수 있으므로 컬렉션을 사용해야 한다.
                        
        * ② 테이블 연관관계
          
            * 데이터베이스 테이블은 외래 키 하나로 양방향으로 조회할 수 있다.
              
            * 두 테이블의 연관관계는 외래 키 하나만으로 양방향 조회가 가능하므로 처음부터 양방향 관계다.
            
    * (2) 양방향 연관관계 매핑

        * ① Member 엔티티
            
            ```java
            @Entity
            public class Member{
              @Id @GeneratedValue
              @Column(name = "MEMBER_ID")
              private Long id;
          
              @Column(name = "USER_NAME")
              private String username;
          
              @ManyToOne
              @JoinColumn(name = "TEAM_ID")
              private Team team;
            }   
            ```

            * 이제 양방향 관계를 매핑하자. Member 엔티티는 단방향과 동일하다.
              
        * ② Team 엔티티
            
            ```java
            @Entity
            public class Team{
              @Id @GeneratedValue
              @Column(name = "TEAM_ID")
              private Long id;
          
              private String name;
          
              // ArrayList로 초기화 해두면 add를 할 때, NPE가 발생하지 않는다.
              @OneToMany(mappedBy = "team")
              private List<Member> members = new ArrayList<>();
            }   
            ```
          
            * 팀에서 회원은 일대다 관계다. 따라서 Team 엔티티에 컬렉션인 `List<Member> members`를 추가한다.

            * 그리고 일대다 관계를 매핑하기 위해 `@OneToMany` 매핑 정보를 사용했다.

            * mappedBy 속성은 양방향 매핑일 때, 사용 하는데 반대쪽 매핑의 필드 이름(team)을 값으로 주면 된다.

        * ③ 일대다 컬렉션 조회

            * 팀에서 회원 컬렉션으로 객체 그래프 탐색을 사용해서 조회한 회원들을 출력한다.

                ```java
                Member findMember = em.find(Member.class, member.getId());
                List<Member> members = findMember.getTeam().getMembers(); // (팀 -> 회원)
                
                for(Member m : members){
                  System.out.println("m = " + m.getUsername());
                }  
                ```

    * (3) 연관관계의 주인과 mappedBy

        * mappedBy 속성을 이해하기 위해서는 객체와 테이블 간에 연관관계를 맺는 차이를 이해해야 한다.
        
            * **객체와 테이블이 연관 관계를 맺는 차이**
    
                * 엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나다. 따라서 둘 사이에 차이가 발생한다.
    
                * ① 객체 연관관계
    
                    * **객체의 양방향 연관관계는** 사실 양방향 관계가 아니라 **서로 다른 단방향 연관관계 2개**다.
        
                        * 회원 → 팀 연관관계 1개 (단방향)
            
                        * 팀 → 회원 연관관계 1개 (단방향)
    
                * ② 테이블 연관관계
    
                    * **테이블은 외래 키 하나로 두 테이블의 연관관계를 관리**한다.
    
                        * 회원 ↔ 팀의 연관관계 1개 (양방향)
                        
            * **객체의 참조 필드 2개 중에서 하나로 외래 키를 관리해야 한다.**
    
                * `Member.team`, `Team.members` 둘 중 어떤 것을 연관관계의 주인으로 정해야 할까?

                    ![image 4](images/img4.png) 
                    
                * 즉, 2개의 참조 필드 중에서 `Member.team` 값을 바꿨을 때, 외래 키(`TEAM_ID`)가 수정 되도록 해야될까?
                  
                * `Team.members` 값을 바꿨을 때, 외래 키(`TEAM_ID`)가 수정 되도록 해야될까?

                    * **연관관계의 주인은 테이블에 외래 키가 있는 곳으로 정해야 한다.**
                    
                    * **연관관계의 주인이 아니면 mappedBy 속성을 지정한다.** 
                    
            * 연관관계의 주인(Owner)

                ![image 5](images/img5.png) 

                * ① **양방향 매핑 규칙**

                    * **양방향 연관관계 매핑 시, 객체의 두 연관관계 중 하나를 연관관계의 주인으로 정해야 한다.**
    
                    * **연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록, 수정) 할 수 있다.**
    
                    * **반면에 주인이 아닌 쪽(mappedBy)은 읽기만 할 수 있다.**

                * ② 연관관계의 주인

                    * **연관관계의 주인은 테이블에 외래 키가 있는 곳으로 정해야 한다.**
    
                    * 여기서는 회원 테이블이 외래 키를 가지고 있으므로 `Member.team`이 연관관계의 주인이 된다.
    
                    * 주인이 아닌 `Team.members`에는 `mappedBy="team"` 속성을 사용해서 주인이 아님을 지정한다.
    
                    * mappedBy 속성의 값으로는 연관관계의 주인인 team을 지정한다.
                    
                * [참고]
                
                    * 데이터베이스 기준으로 테이블의 다대일, 일대다 관계에서는 항상 다 쪽이 외래 키를 가진다.
                      
                    * 다 쪽인 `@ManyToOne`은 항상 연관관계의 주인이 되므로 `mappedBy`를 설정 할 수 없다.

    * (4) 양방향 연관관계의 주의점

        * ① 양방향 매핑 시 가장 많이 하는 실수
        
            * **연관관계의 주인에는 값을 입력하지 않고, 주인이 아닌 방향(역방향)만 값을 입력하는 실수를 한다.**
            
            * 순수한 객체 상태를 고려해서 항상 양쪽에 값을 설정해야 한다. (객체 지향적인 관점)  
            
                ```java
                Team team = new Team();
                team.setName("TeamA");
                em.persist(team);
              
                Member member = new Member();
                member.setUsername("member1");
              
                // 연관관계의 주인에 값 설정
                member.setTeam(team);
                em.persist(member);
              
                // 역방향도 값 설정
                team.getMembers().add(member);
                ```

        * ② **연관관계 편의 메소드를 생성하자.**
        
            * `member.setTeam(team)`과 `team.getMembers().add(member)`를 각각 호출하다 보면 실수로 둘 중 하나만 호출해서 양방향이 깨질 수 있다.
    
            * 그래서 `Member`에 `changeTeam()` 메소드 하나로 양방향 관계를 모두 설정 하도록 변경했다.

                ```java
                @Entity
                public class Member{
                  @Id @GeneratedValue
                  @Column(name = "MEMBER_ID")
                  private Long id;
              
                  @Column(name = "USER_NAME")
                  private String username;
              
                  @ManyToOne
                  @JoinColumn(name = "TEAM_ID")
                  private Team team;
              
                  // 연관관계 편의 메소드
                  public void changeTeam(Team team){
                      this.team = team;
                      team.getMembers().add(this);
                  } 
                }   
                ```

                ```java
                Team team = new Team();
                team.setName("TeamA");
                em.persist(team);
              
                Member member = new Member();
                member.setUsername("member1");
                member.changeTeam(team);
                em.persist(member);
                ```
              
                * 반대편(Team)에서 연관관계 편의 메소드를 만들어서 사용하는 것도 가능하다. 둘 중 한 곳에만 작성하면 된다.

        * ③ **양방향 매핑 시에 무한 루프를 조심하자**
        
            * 롬복(LOMBOK)에서 `toString()`를 만들 때, 참조 타입은 제외하자.

            * `JSON 생성 라이브러리` 
            
                * 양방향 매핑 관계의 엔티티를 JSON으로 직렬화 하면 무한 루프에 빠지게 된다.
            
                * 컨트롤러에서는 절대 엔티티를 반환하지 않는다. 
                
                    * 컨트롤러에서 엔티티를 반환하는 상황에서 엔티티를 변경하면 API 스펙 자체가 변경되는 문제점이 있다. 
                     
                    * 단순 값만 있는 DTO로 변환해서 반환하자. 그러면 `JSON 생성 라이브러리`로 인한 문제는 발생하지 않는다.
                    
                * JSON 생성 라이브러리들은 무한 루프에 빠지지 않도록 하는 애노테이션을 제공한다.
                
                    * Jackson 2.0 버전 이전 : `@JsonManagedReference`, `@JsonBackReference`
                    
                    * Jackson 2.0 버전 이상 : `@JsonIdentityInfo`

    * (5) 양방향 매핑 정리

        * **단방향 매핑만으로도 객체와 테이블의 연관관계 매핑은 이미 완료된 것이다.**

        * 단방향을 양방향으로 만드는 것은 반대 방향으로 객체 그래프 탐색 기능이 추가된 것 뿐이다.

        * **JPA 모델링 시, 처음에는 단방향 매핑으로 설계를 완료하고 양방향은 필요할 때 추가하면 된다.**

        * 나중에 추가해도 되는 이유는 테이블에 영향을 주지 않기 때문이다.

        * **단방향은 항상 외래 키가 있는 곳(多)을 기준으로 매핑하면 된다.**

## 6. 다양한 연관관계 매핑

* 연관관계 매핑 시 고려사항 3가지

    * **다중성**
      
        * ① 다대일: `@ManyToOne`
          
        * ② 일대다: `@OneToMany`
          
        * ③ 일대일: `@OneToOne`
          
        * ④ 다대다: `@ManyToMany`
           
            * 사실, 다대다는 실무에서 사용하면 안 된다. 
            
    * **단방향, 양방향**
      
        * ① 테이블

            * 외래 키 하나로 양쪽으로 조인 할 수 있다.
    
            * 테이블은 사실 방향이라는 개념이 없다.

        * ② 객체

            * 참조용 필드가 있는 쪽으로만 참조 할 수 있다.

            * 한쪽만 참조하면 단방향

            * 양쪽이 서로 참조하면 양방향

                * 사실은 단방향이 2개인 것이다.
            
    * 양방향인 경우, **연관관계의 주인**를 정해야 한다.

        * 객체의 양방향 관계는 참조가 2개 있는데 둘 중 테이블의 외래 키를 관리할 참조를 지정해야 한다.

        * 외래 키를 관리하는 참조가 연관관계의 주인이 된다.

        * 주인의 반대편은 외래 키에 영향을 주지 않으며 단순 조회만 가능하다.

* `다중성`과 관련해서 자세히 살펴보기

    * 다대일 [N:1]
    
        * ① 다대일 단방향
        
            ![image 6](images/img6.png) 
    
            * 관계형 데이터베이스에서는 항상 다 쪽에 외래 키가 있어야 한다.
            
                * 그렇지 않다면 설계가 잘못된 것이다.
    
            * 다대일 단방향은 가장 많이 사용하는 연관관계다.
    
            * **외래 키가 있는 쪽에 참조를 추가하고 연관관계를 매핑한다.**
    
            * **"다대일"의 반대는 "일대다"가 된다.**
            
        * ② 다대일 양방향
        
            ![image 7](images/img7.png)

            * 외래 키가 있는 쪽이 연관관계의 주인이다.

            * 양쪽을 서로 참조하도록 할 때, 사용한다.
            
    * 일대다 [1:N]
    
        * ① 일대다 단방향
        
            ![image 8](images/img8.png)

            * 하나의 팀은 여러 회원을 참조 할 수 있는데 이런 관계를 일대다 관계라 한다. (Team이 연관관계의 주인이 된 구조)

            * 그리고 팀은 회원들을 참조하지만 반대로 회원은 팀을 참조하지 않으면 둘의 관계는 단방향이다.

            * **일대다 단방향 관계**는 **반대편 테이블의 외래 키를 관리하는 특이한 구조**가 된다.
            
                * 다 쪽인 Member 엔티티에는 외래 키를 매핑 할 수 있는 참조 필드가 없다.
    
                * 대신에 반대 쪽인 Team 엔티티에만 외래 키를 매핑 할 수 있는 참조 필드인 members가 있기 때문이다.
                
            * 예시 - 일대다 단방향으로 매핑한 팀(Team) 엔티티
            
                ```java
                @Entity
                public class Team{
                  @Id @GeneratedValue
                  @Column(name = "TEAM_ID")
                  private Long id;
              
                  private String name;
              
                  @OneToMany
                  @JoinColumn(name = "TEAM_ID")
                  private List<Member> members = new ArrayList<>();
                }   
                ```
              
                * **일대다 단방향은 일대다(1:N)에서 일(1)이 연관관계의 주인이다.**

                * **일대다 단방향은** `@JoinColumn`**을 꼭 사용해야 한다.**

                * **그렇지 않으면 조인 테이블 방식을 사용하여 중간에 테이블을 하나 추가한다.**
                
                * **일대다 단방향 매핑 보다는 다대일 양방향 매핑을 사용하자**
    
                    * 사실 다대일 단방향, 양방향만 알면 일대다에 대해서는 몰라도 된다.

        * ② 일대다 양방향
        
            ![image 9](images/img9.png)

            * 일대다 양방향 매핑은 공식적으로 존재하지는 않는다. 
            
            * 일대다 단방향 매핑 반대편에 같은 외래 키를 사용하는 다대일 단방향 매핑을 읽기 전용으로 하나 추가하면 된다.
            
                * `@JoinColumn(name = "TEAM_ID", insertable=false, updatable=false)`
                
            * **일대다 양방향 매핑보다는 다대일 양방향 매핑을 사용하자**
            
    * 일대일 [1:1]

        * 설명
    
            * 일대일 관계는 그 반대도 일대일 관계다.
        
            * 주 테이블이나 대상 테이블 중에 누가 외래 키를 가질지 선택 할 수 있다.
        
            * 외래 키에 데이터베이스 유니크(UNI) 제약조건을 추가해야 한다.
              
                * 제약 조건을 추가하지 않더라도 가능하지만 애플리케이션에서 관리를 잘 해주어야 함.
    
                * `@JoinColumn(name = "LOCKER_ID", unique = true)`
        
        * 예시

            * ① 주 테이블에 외래 키가 있는 일대일 단방향 관계
    
                ![image 10](images/img10.png)
    
                * `MEMBER`가 주 테이블이고 `LOCKER`는 대상 테이블이다.
                
                    * 여기서 말하는 `주 테이블`은 많이 접근(Access)하는 테이블을 말한다.
                
                * `@OneToOne`으로 일대일 단방향 관계를 매핑하고 `@JoinColumn`를 지정한다. (다대일 단방향과 거의 비슷함)
                
                    ```java
                    @Entity
                    public class Member{
                      @Id @GeneratedValue
                      @Column(name = "MEMBER_ID")
                      private Long id;
                  
                      @Column(name = "USER_NAME")
                      private String username;
                  
                      @OneToOne
                      @JoinColumn(name = "LOCKER_ID")
                      private Locker locker;
                    }   
                    ```
                  
                    ```java
                    @Entity
                    public class Locker{
                      @Id @GeneratedValue
                      private Long id;
                  
                      private String name;
                    }   
                    ```
                  
            * ② 주 테이블에 외래 키가 있는 일대일 양방향 관계
    
                ![image 11](images/img11.png)
    
                * 일대일 **양방향** 관계이므로 **연관관계의 주인**을 정해야 한다.
                
                * 다대일 양방향 매핑처럼 **외래 키가 있는 곳이 연관관계의 주인**이며 **반대편은 mappedBy 속성**을 사용한다.

                    ```java
                    @Entity
                    public class Locker{
                      @Id @GeneratedValue
                      private Long id;
                  
                      private String name;
                  
                      @OneToOne(mappedBy = "locker")
                      private Member member;
                    }   
                    ```

                    *  MEMBER 테이블이 외래 키를 가지고 있으므로 Member 엔티티에 있는 `Member.locker`가 연관관계의 주인이다. 
                    
                    *  따라서 반대 매핑인 `Locker.member`는 `mappedBy` 속성을 사용해서 연관관계의 주인이 아니라고 설정했다.

            * ③ 대상 테이블에 외래 키가 있는 일대일 단방향 관계
    
                ![image 12](images/img12.png)
    
                * 대상 테이블에 외래 키가 있는 일대일 단방향 관계는 JPA에서 지원하지 않는다. 
                
                    * 이러한 경우에는 양방향 관계로 만들고 Locker를 연관관계의 주인으로 설정해야 한다.
    
                    * 즉, **일대일 관계는 자신의 엔티티가 자신의 테이블에 있는 외래 키를 직접 관리해야 한다. (★★★)**  
                      
            * ④ 대상 테이블에 외래 키가 있는 일대일 양방향 관계
    
                ![image 13](images/img13.png)
    
                * 주 엔티티인 Member 엔티티 대신에 **대상 엔티티인 Locker를 연관관계의 주인으로 만들어서 LOCKER 테이블의 외래 키를 관리하도록 한다.**

                * 사실, 일대일 주 테이블에 외래 키 양방향과 매핑 방법은 같음
                    
                    ```java
                    @Entity
                    public class Member{
                      @Id @GeneratedValue
                      @Column(name = "MEMBER_ID")
                      private Long id;
                  
                      @Column(name = "USER_NAME")
                      private String username;
                  
                      @OneToOne(mappedBy = "member")
                      private Locker locker;
                    }   
                    ```
                  
                    ```java
                    @Entity
                    public class Locker{
                      @Id @GeneratedValue
                      private Long id;
                  
                      private String name;
                  
                      @OneToOne
                      @JoinColumn(name = "MEMBER_ID")
                      private Member member;
                    }   
                    ```
                  
    * 다대다 [N:M]
    
        * ① 다대다
          
            * 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현 할 수 없다.
              
                * **다대다 관계는 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야 한다.**

            * 객체는 컬렉션을 사용해서 객체 2개로 다대다 관계를 표현 할 수 있다.       

        * ② 다대다 매핑의 한계

            * 다대다 매핑은 편리해 보이지만 실무에서 사용할 수 없다.
            
            * 연결 테이블이 보통 연결만 하고 끝나지는 않는다.

                * 연결 테이블에 주문 시간, 수량 같은 컬럼이 더 필요한 경우가 많다.
    
                * 하지만 연결 테이블에 컬럼을 추가하면 더 이상 `@ManyToMany`를 사용 할 수 없다. 
                
        * ③ 다대다 매핑의 한계를 극복

            * **연결 테이블용 엔티티를 추가한다.** (연결 테이블을 엔티티로 승격)
    
            * 그리고 `@ManyToMany`를 `@OneToMany`, `@ManyToOne`로 변경한다.
            
                ![image 14](images/img14.png)
                
                * 연결 테이블(`Order`)에 새로운 기본 키(`ORDER_ID`)를 추가하고 다른 식별자는 외래 키로 사용한다.
                    
        * ④ 예시 - 다대다 매핑을 일대다 다대일로 변경

            * 회원과 회원상품을 양방향 관계로 만든다. 회원 상품 엔티티 쪽이 외래 키를 가지고 있으므로 연관관계의 주인이다.
                
                ```java
                @Entity
                public class Member{
                  @Id @GeneratedValue
                  @Column(name = "MEMBER_ID")
                  private Long id;
              
                  //...
              
                  @OneToMany(mappedBy = "member")
                  private List<MemberProduct> memberProducts = new ArrayList<>();
                }   
                ```
              
            * 회원 상품 엔티티는 기본 키를 매핑하는 `@Id`와 외래 키를 매핑하는 `@JoinColumn`을 동시에 사용한다.
            
                ```java
                @Entity
                public class MemberProduct{
                  @Id @GeneratedValue
                  private Long id;
              
                  @ManyToOne
                  @JoinColumn(name = "MEMBER_ID")
                  private Member member;
              
                  @ManyToOne
                  @JoinColumn(name = "PRODUCT_ID")
                  private Product product;
              
                  private int orderAmount;
                  
                  private LocalDateTime orderDateTime;
                }   
                ```
              
            * 상품 엔티티에서 회원 상품 엔티티로 연관관계를 만들었다.
            
                ```java
                @Entity
                public class Product{
                  @Id @GeneratedValue
                  private Long id;
              
                  private String name;
              
                  @OneToMany(mappedBy = "product")
                  private List<MemberProduct> memberProducts = new ArrayList<>();
                }   
                ```

* 다양한 연관관계 매핑 정리

    * 다대일 단방향, 양방향 매핑을 사용한다.
    
        * (일대다 단방향 매핑, 일대다 양방향 매핑 보다는 다대일 양방향 매핑을 사용하자.)
    
    * 또는 일대일 단방향, 양방향 매핑을 사용한다.
    
    * 그리고 다대다 관계는 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야 한다.

## 7. 고급 매핑

### 1. 상속관계 매핑

* (1) 개요
  
    * 관계형 데이터베이스는 상속 관계가 없다.
    
    * 관계형 데이터베이스의 슈퍼타입 서브타입 관계라는 모델링 기법이 객체의 상속 관계와 유사하다.
    
    * ORM에서 이야기 하는 `상속관계 매핑`은 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑하는 것이다.

* (2) 슈퍼타입 서브타입 논리 모델을 실제 물리 모델인 테이블로 구현하는 방법

    * ① `조인 전략` : 엔티티 각각을 모두 테이블로 만들고 조회할 때 조인을 사용한다.
    
    * ② `단일 테이블 전략` : 테이블을 하나만 사용해서 통합한다. (기본 값)
    
    * ③ `구현 클래스 마다 테이블 전략` : 서브 타입 마다 하나의 테이블을 만든다. (잘 사용하지 않음)
    
* (3) 상속관계 매핑을 자세하게 살펴보기

    * ① 조인 전략
    
        ![image 15](images/img15.png)
    
        * `조인 전략(Joined Strategy)`은 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 `기본 키`를 받아서 `기본 키 + 외래 키`로 사용하는 전략이다. 
        
            * 이 전략을 사용할 때 주의할 점이 있는데 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없다. 
            
            * 따라서 타입을 구분하는 컬럼(`DTYPE`)을 추가해야 한다.
        
        * 예시
        
            * 조인 전략을 사용한 예제 코드는 다음과 같다.
        
                ```java
                @Entity
                @Inheritance(strategy = InheritanceType.JOINED)
                @DiscriminatorColumn
                public abstract class Item{
                  @Id @GeneratedValue
                  private Long id;
              
                  private String name;
              
                  private int price;
                }   
                ```
              
                * 상속 관계 매핑은 부모 엔티티에 `@Inheritance`를 사용해야 한다. 그리고 매핑 전략을 지정해야 하는데 여기서는 조인 전략을 사용한다.
                
                * `@DiscriminatorColumn`는 부모 엔티티에 구분 컬럼을 지정한다. 해당 컬럼으로 저장된 자식 테이블을 구분 할 수 있다.
                
                    * 구분 컬럼의 기본 이름은 DTYPE이다.
    
                    * name 옵션으로 구분 컬럼의 이름을 변경 할 수 있다.
    
            * 자식 엔티티를 저장할 때, 구분 컬럼에 입력할 값을 지정 할 수도 있다.
            
                ```java
                @Entity
                @DiscriminatorValue("M")
                public class Movie extends Item{
                  private String director;
              
                  private int price;
                }   
                ```
          
                * `@DiscriminatorValue("M")` : 자식 엔티티를 저장할 때, 구분 컬럼에 입력할 값을 지정한다.
        
                    * 만약 영화(Movie) 엔티티를 저장하면 구분 컬럼인 DTYPE에 M이 저장된다.
    
    * ② 단일 테이블 전략
    
        ![image 16](images/img16.png)
    
        * `단일 테이블 전략(Single-Table Strategy)`은 하나의 테이블로 통합한다.
        
            * 그리고 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분한다.
        
            * 이 전략을 사용할 때 주의점은 자식 엔티티가 매핑한 컬럼은 모두 null을 허용 해야한다는 점이다.
            
                * 예를 들어, Book 엔티티를 저장하면 ITEM 테이블의 AUTHOR, ISBN만 사용하고 다른 엔티티와 매핑된 ARTIST, DIRECTOR, ACTOR 컬럼은 사용하지 않으므로 null이 입력되기 때문이다.
                
        * 예시
        
            ```java
            @Entity
            @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
            @DiscriminatorColumn
            public abstract class Item{
              @Id @GeneratedValue
              private Long id;
          
              private String name;
          
              private int price;
            }   
            ```
                 
            * 단일 테이블 전략은 `@DiscriminatorColumn`을 지정하지 않아도 `DTYPE`이 추가된다.
    
    * ③ 구현 클래스 마다 테이블 전략

        ![image 18](images/img18.png)
      
        * `구현 클래스 마다 테이블 전략(Table-per-Concrete-Class Strategy)`은 자식 엔티티 마다 테이블을 만든다.
          
            * `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`
          
        * 그리고 자식 테이블 각각에 필요한 컬럼이 모두 있다.
          
        * 여러 자식 테이블을 함께 조회할 때 성능이 느리다.
    
            * Ex) `Item item = em.find(Item.class, movie.getId());`
        
                * SQL에 UNION을 사용해야 되기 때문에 성능이 느리다.
               
                * 이 전략은 데이터베이스 설계자와 ORM 전문가 둘 다 추천하지 않는다.

### 2. @MappedSuperclass

* (1) @MappedSuperclass

    * `@MappedSuperclass`는 자식 클래스에게 공통 매핑 정보를 제공하는 부모 클래스를 작성할 때 사용한다.
    
    * `@MappedSuperclass`로 지정한 클래스는 엔티티가 아니므로 `em.find()` 나 JPQL에서 사용 할 수 없다.
    
    * 해당 클래스를 직접 생성해서 사용할 일은 거의 없으므로 추상 클래스로 만드는 것을 권장한다.
    
    * 주로 등록일, 수정일, 등록자, 수정자와 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용한다.
    
* (2) 예시

    ```java
    @MappedSuperclass
    public abstract class BaseEntity{
      // 없어도 됨. @Column를 사용 할 수 있다는 것을 보여주기 위함
      @Column(name = "INSERT_MEMBER") 
      private String createdBy;
  
      private LocalDateTime createdDate;
  
      @Column(name = "UPDATE_MEMBER")
      private String lastModifiedBy;
  
      private LocalDateTime lastModifiedDate;
    }   
    ```
  
    ```java
    @Entity
    public class Member extends BaseEntity{
      @Id @GeneratedValue
      @Column(name = "MEMBER_ID")
      private Long id;
  
      @Column(name = "USER_NAME")
      private String username;
    }   
    ```

## 8. 프록시와 연관관계 관리

### 1. 프록시(Proxy)

* `프록시 객체`는 실제 엔티티 객체를 대신하여 데이터베이스 조회를 지연 할 수 있는 가짜 객체를 말한다.

* `em.find()` vs `em.getReference()` 

    * `em.find()` : 데이터베이스를 통해서 실제 엔티티 객체를 조회한다.
    
    * `em.getReference()` : 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체를 반환한다.

* 프록시 특징

    * 프록시 객체는 실제 객체의 참조(target)를 보관한다.

    * 프록시 객체의 메소드를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.
    
* 프록시 객체의 초기화
    
    * `프록시 객체의 초기화`는 프록시 객체가 사용될 때, 데이터베이스를 조회해서 실제 엔티티 객체를 생성하는 것을 말한다.
    
        * 즉, 프록시 객체의 메소드를 호출하면 초기화가 진행된다.

            ```java
            Member findMember = em.getReference(Member.class, "id1");
          
            // [결과] findMember = class hellojpa.Member$HibernateProxy$ZIFtUTMC
            // System.out.println("findMember = " + findMember.getClass()); 
          
            findMember.getName(); // 프록시 객체의 메소드 호출
            ```

    * 프록시의 초기화 과정
    
        ![image 17](images/img17.png)
    
        * ① 프록시 객체(`member`)의 `getName()`을 호출해서 실제 데이터를 조회한다.
          
        * ② 프록시 객체는 실제 엔티티 객체가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티 객체 생성을 요청하는데 이것을 초기화라 한다.
          
        * ③ 영속성 컨텍스트는 데이터베이스를 조회해서 실제 엔티티 객체를 생성한다.
          
        * ④ 프록시 객체는 생성된 실제 엔티티 객체의 참조를 `Member` 타입의 `target` 멤버변수에 보관한다.
        
            * ② ~ ④의 과정은 `target`에 값이 없을 때만 진행된다. 
          
        * ⑤ 프록시 객체는 실제 엔티티 객체의 `getName()`을 호출해서 결과를 반환한다.
        
* 프록시 정리

    * 프록시 객체는 처음 사용할 때 한 번만 초기화된다.

    * 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것이 아니다. 

        * 프록시 객체가 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 할 수 있게 되는 것이다.

    * 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시, 주의해야 한다. 

        * `==` 비교 대신에 `instanceof`를 사용해야 한다.

    * 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 데이터베이스를 조회할 필요가 없으므로 `em.getReference()`를 호출해도 프록시가 아닌 실제 엔티티를 반환한다.

        ```java
        Member m1 = em.find(Member.class, member1.getId());
        System.out.println("m1 = " + m1.getClass()); // m1 = class hellojpa.Member
        
        Member reference = em.getReference(Member.class, member1.getId());
        System.out.println("reference = " + reference.getClass()); // reference = class hellojpa.Member
        ```

        * JPA는 같은 트랜잭션 안에서 같은 엔티티를 반환한다.
    
            * 즉, JPA에서 실제 엔티티, 프록시 객체 중 무엇이던간에 `==` 비교는 `true`를 반환해야 한다.
      
    * 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화 하면 문제가 발생한다.

        ```java
        try{
            Member member1 = new Member();
            member1.setUsername("member1");
            em.persist(member1);
            
            em.flush();
            em.clear();
            
            Member refMember= em.getReference(Member.class, member1.getId());
            System.out.println("refMember = " + refMember.getClass());
            
            em.detach(refMember); // refMember를 준영속 상태로 만들기
            
            refMember.getUsername(); // LazyInitializationException 예외 발생
            
            tx.commit();
        }catch (Exception e){
            tx.rollback();
            e.printStackTrace();
        }finally {
            em.close();
        }
        ```

        * 하이버네이트는 `org.hibernate.LazyInitializationException` 예외를 발생시킨다.

        * 문제가 발생하는 이유는 프록시의 초기화 요청은 영속성 컨텍스트를 통해 이루어지기 때문이다. 
        
### 2. 즉시 로딩과 지연 로딩

* JPA는 개발자가 연관된 엔티티의 조회 시점을 선택 할 수 있도록 두 가지 방법을 제공한다.

    * ① `즉시 로딩(EAGER LOADING)` : 연관된 엔티티를 함께 조회하는 방식이다.
    
        * 예시 : `em.find(Member.class, "member1")` 처럼 호출할 때, 회원 엔티티와 연관된 팀 엔티티도 함께 조회한다.
    
        * 설정 방법 : `@ManyToOne(fetch = FetchType.EAGER)`
    
    * ② `지연 로딩(LAZY LOADING)`: 연관된 엔티티를 실제 사용할 때까지 데이터베이스 조회를 지연하는 방식이다.
    
        * 연관된 엔티티를 프록시 객체로 가져온다. 프록시를 실제 사용할 때, 초기화하면서 데이터베이스를 조회한다.
    
        * 예시 : `em.find(Member.class, "member1")` 처럼 호출할 때, 회원(member1)을 조회하고 회원의 team 멤버 변수에 프록시 객체를 넣어둔다.
            
            * `member.getTeam().getName()`처럼 팀 엔티티를 실제 사용하는 시점에 데이터베이스를 조회해서 프록시 객체를 초기화한다.
        
                * 조회 대상이 영속성 컨텍스트에 이미 있으면 프록시 객체가 아닌 실제 객체를 사용한다.
    
        * 설정 방법 : `@ManyToOne(fetch = FetchType.LAZY)`
        
* 주의사항

    * 실무에서는 모든 연관관계에 `지연 로딩(LAZY LOADING)`을 사용해야 한다.

    * `즉시 로딩(EAGER LOADING)`을 적용하면 예상하지 못한 쿼리가 발생한다.

    * `즉시 로딩`은 JPQL에서 `N+1` 문제를 일으킨다.

        ```java
        Team teamA = new Team();
        teamA.setName("teamA");
        em.persist(teamA);
        
        Team teamB = new Team();
        teamB.setName("teamB");
        em.persist(teamB);
        
        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setTeam(teamA);
        em.persist(member1);
        
        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setTeam(teamB);
        em.persist(member2);
        
        em.flush();
        em.clear();
        
        List<Member> members = em.createQuery("select m from Member m", Member.class)
                .getResultList();
        ```

        * 소스코드 동작 과정

            * 일단, `select * from Member`로 Member의 데이터를 가져온다.
        
            * Member 클래스를 보면 연관된 엔티티인 Team이 EAGER로 설정 되어 있다. 
        
                * 이때, LAZE 였다면 프록시 객체를 넣게 된다.
        
            * 그러면 즉시, `select * from Team where TEAM_ID = xxx`로 Team의 데이터를 가져온다.  (여기서, N+1 문제 발생)      

        * `N+1 문제` : 처음에 쿼리 1개를 실행 했을 때, 얻은 결과의 개수(N개)만큼 추가 쿼리가 발생하는 문제를 말한다.
            
            * 지연 로딩을 사용하더라도 Loop를 이용하면 N+1 문제가 발생 할 수 있다.
            
            * 해결 방안은 다음과 같다.
    
                * ① JPQL의 fetch 조인을 이용한다.
    
                * ② `@EntityGraph`를 이용한다.
                
                * ③ `@BatchSize`를 이용한다.
                
    * `@ManyToOne`, `@OneToOne`의 기본 값은 즉시(EAGER) 로딩이므로 `LAZY`로 설정해야 한다.
    
        * `@XToOne` 시리즈 

### 3. 영속성 전이: CASCADE

* CASCADE 란?
    
    * `CASCADE`는 해당 엔티티의 상태 변화를 연관된 엔티티에 전파시킨다.
    
        * 예를 들어, 부모만 영속 상태로 만들면 연관된 자식 엔티티까지 함께 영속화해서 저장한다.
        
            * `@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)`

    * CASCADE의 종류
    
        * **`ALL` : 모두 적용**
    
        * **`PERSIST` : 영속**
    
        * **`REMOVE` : 삭제**
        
        * `MERGE` : 병합
    
        * `REFRESH` : REFRESH
    
        * `DETACH` : DETACH
        
    * 주의사항
    
        * 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없다.
        
        * 단지 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐이다.
        
        * 하나의 부모만 자식들을 관리 한다면 CASCADE 속성을 사용 할 수 있다.
        
* 실습하기
    
    * 부모 엔티티
    
        ```java
        @Entity
        public class Parent{
          @Id @GeneratedValue
          private Long id;
          
          @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
          private List<Child> children = new ArrayList<>();
      
          public void addChild(Child child){
              children.add(child);
              child.setParent(this);
          }
        }
        ```
        
    * 자식 엔티티
    
        ```java
        @Entity
        public class Child{
          @Id @GeneratedValue
          private Long id;
          
          @ManyToOne
          @JoinColumn(name = "parent_id")
          private Parent parent;
        }
        ```
      
    * 부모를 영속화할 때, 연관된 자식들도 함께 영속화한다.

        ```java
        Child child1 = new Child();
        Child child2 = new Child();
      
        Parent parent = new Parent();
        parent.addChild(child1); // 연관관계 설정
        parent.addChild(child2); // 연관관계 설정
      
        em.persist(parent); // 부모 저장, 연관된 자식들 저장
        ```
    
### 4. 고아 객체

* `고아 객체(ORPHAN) 제거`는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능이다.

    * `@OneToMany(orphanRemoval = true)`

* 실습하기
    
    * 고아 객체 제거 기능을 활성화한다. 
    
        ```java
        @Entity
        public class Parent{
          @Id @GeneratedValue
          private Long id;
          
          @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST, orphanRemoval = true)
          private List<Child> children = new ArrayList<>();
      
          public void addChild(Child child){
              children.add(child);
              child.setParent(this);
          }
        }
        ```
              
    * 이제 컬렉션에서 제거한 자식 엔티티는 자동으로 삭제된다.

        ```java
        Parent findParent = em.find(Parent.class, parent.getId());
        findParent.getChildren().remove(0);
        ```

* 주의사항

    * 특정 엔티티가 개인 소유하는 엔티티에만 고아 객체 제거 기능을 사용해야 한다.

    * `orphanRemoval`는 `@OneToOne`, `@OneToMany`에만 사용할 수 있다.

    * 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다. 
    
        * 이것은 `CascadeType.REMOVE` 처럼 동작한다.

* 영속성 전이 + 고아 객체, 생명주기

    * `CascadeType.ALL + orphanRemovel=true` : 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있다.

        * 도메인 주도 설계(DDD)의 `Aggregate Root` 개념을 구현할 때, 유용하다.

## 9. 값 타입

### 1. JPA의 데이터 타입 분류

* JPA의 데이터 타입은 크게 `엔티티 타입`과 `값 타입`으로 나눌 수 있다.

    * ① 엔티티 타입

        * `엔티티 타입` : `@Entity`로 정의하는 객체다.

        * 데이터가 변해도 식별자를 통해 지속해서 추적 할 수 있다.

            * Ex) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능하다. 
    
        * 공유 할 수 있다.

    * ② 값 타입(Value Type)

        * `값 타입` : int, Integer, String처럼 단순히 값으로 사용하는 자바의 기본 타입이나 객체다.

        * 식별자가 없고 값만 있으므로 변경 시 추적 할 수 없다.

            * Ex) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체된다.
    
        * 공유하지 않는 것이 안전하다. (복사해서 사용)
    
        * 불변 객체로 만드는 것이 안전하다.
        
* 값 타입 분류

    * `값 타입`은 크게 3가지로 나눌 수 있다.

        * ① `기본값 타입`
    
            * 자바 기본 타입(int, double ...)
        
            * 래퍼 클래스(Integer, Long ...)
        
            * String

        * ② `임베디드 타입(embedded type, 복합 값 타입)`

        * ③ `컬렉션 값 타입(collection value type)`
        
    * (1) 기본 값 타입
    
        * 예를 들어 `String name`, `int age`가 기본 값 타입이다.
    
        * 생명주기를 엔티티에 의존한다.
      
            * Ex) 회원 엔티티를 삭제하면 이름, 나이 필드도 함께 삭제된다.
    
        * 값 타입은 공유하면 안 된다.
    
            * Ex) 회원 이름 변경 시 다른 회원의 이름도 함께 변경되면 안 되기 때문이다.
    
        * [참고] 자바의 기본 타입(primitive type)은 절대 공유되지 않는다.
    
            * int, double과 같은 기본 타입은 절대 공유되지 않는다.
    
                * 기본 타입은 항상 값을 복사하기 때문이다.
            
                * 그래서 기본 타입을 값 타입으로 사용 했을 때, 안전하다.
    
            * Integer 같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경 불가능(Immutable)하다.
    
                * 부작용 (Side effect)이 발생하지 않음
                  
    * (2) 임베디드 타입 (복합 값 타입)
    
        * `임베디드 타입(embedded type)`은 다른 타입들을 포함하고 있는 타입을 말한다.
        
            * `복합 값 타입`이라고도 한다.
            
            * 임베디드 타입도 `int`, `String`과 같은 값 타입이다.

        * 임베디드 타입의 장점

            * 재사용이 가능하다.
            
            * 응집도가 높다.
            
                * `Period.isWork()`처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있다.

        * 임베디드 타입을 사용하는 경우
    
            * 회원 엔티티는 이름, 근무 시작일과 종료일, 도시, 번지, 우편번호를 가진다.

                ```java
                public class Member{
                    private Long id;
                    private String name;
                    
                    private LocalDateTime startDate;
                    private LocalDateTime endDate;
                
                    private String city;
                    private String street;
                    private String zipcode;
                }
                ```

            * 임베디드 타입을 이용하면 "회원 엔티티는 이름, 근무 기간, 집 주소를 가진다."로 변경 할 수 있다.

        * 임베디드 타입을 정의해서 사용하기

            * ① 임베디드 타입과 테이블을 매핑

                * `@Embeddable` : 임베디드 타입을 정의하는 곳에 붙여준다.
                
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
    
                    * 임베디드 타입은 기본 생성자가 필수다. 
    
                * `@Embedded` : 임베디드 타입을 사용하는 곳에 붙여준다.
                
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
                    
                        // Getter
                    }
                    ```
    
            * ② 엔티티를 등록
        
                ```java
                Member member = new Member();
                member.setUsername("hello");
                member.setHomeAddress(new Address("city", "street", "100"));
                
                em.persist(member);
                ```

            * 정리하기
    
                * 임베디드 타입은 엔티티의 값일 뿐이다.
    
                * 임베디드 타입을 사용하기 전과 후에 **매핑하는 테이블은 같다.**
    
                * 임베디드 타입 덕분에 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능하다.
    
                * 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수 보다 클래스의 수가 더 많다.
    
                * 임베디드 타입은 값 타입을 포함하거나 엔티티를 참조 할 수 있다.

                    ```java
                    @Embeddable
                    public class Address{
                        String street;
                        String city;
                        String state;
                        
                        @Embedded Zipcode zipcode; // 임베디드 타입 포함
                    }
                    
                    @Embeddable
                    public class PhoneNumber{
                        String areaCode;
                        String localNumber;
                        
                        @ManyToOne
                        PhoneServiceProvider provider; // 엔티티 참조
                    }
                    ```
                  
        * @AttributeOverride 속성 재정의 

            * `@AttributeOverride` : 임베디드 타입에 정의한 매핑 정보를 재정의한다. 

                * 한 엔티티에서 같은 값 타입을 사용하면 테이블에 매핑하는 컬럼명이 중복된다.
        
                * 이때는 `@AttributeOverrides`를 사용해서 매핑 정보(컬럼명 속성)를 재정의한다.
    
            * 예시

                ```java
                @Entity
                public class Member{
                
                    //...
                
                    @Embedded
                    private Address homeAddress;
                
                    @Embedded
                    @AttributeOverrides({
                            @AttributeOverride(name = "city",
                                    column = @Column(name = "COMPANY_CITY")),
                            @AttributeOverride(name = "street",
                                    column = @Column(name = "COMPANY_STREET")),
                            @AttributeOverride(name = "zipcode",
                                    column = @Column(name = "COMPANY_ZIPCODE"))
                    })
                    private Address companyAddress;
                
                }
                ```

        * 임베디드 타입과 null
    
            * 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null이 된다.

                ```java
                member.setAddress(null); // null 입력
                em.persist(member);
                ```
              
                * Address가 null이면 회원 테이블의 주소와 관련된 CITY, STREET, ZIPCODE 컬럼 값은 모두 null이 된다.

    * (3) 값 타입 컬렉션
    
        * 값 타입 컬렉션?
    
            * `값 타입 컬렉션`은 **컬렉션에 값 타입을 넣어서 사용하는 것**을 말한다.
    
                * 값 타입을 하나 이상 저장할 때, 값 타입 컬렉션을 사용한다.
                    
            * `@ElementCollection`, `@CollectionTable`를 사용해서 매핑한다.
            
            * 관계형 데이터베이스의 테이블은 컬럼 안에 컬렉션을 포함 할 수 없다. 
    
                * JSON을 지원하는 DB는 가능한 경우도 있다. 하지만 기본적으로는 가능하지 않다. 
            
            * 따라서 컬렉션을 저장하기 위한 별도의 테이블을 생성해야 한다.
              
            * 그리고 값 타입 컬렉션은 기본적으로 `지연 로딩(LAZY LOADING)` 전략을 사용한다. 
     
        * 값 타입 컬렉션을 매핑하고 사용하기

            * 값 타입 컬렉션을 매핑하기
  
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
                    @Column(name = "FOOD_NAME") // 새롭게 생성되는 테이블의 컬럼명을 지정
                    private Set<String> favoriteFoods = new HashSet<>();
                
                    @ElementCollection
                    @CollectionTable(name = "ADDRESS",
                                     joinColumns = @JoinColumn(name = "MEMBER_ID"))
                    private List<Address> addressHistory = new ArrayList<>();
                
                    // Getter, Setter
              
                }
                ```
    
                * `@ElementCollection` : 값 타입 컬렉션이라는 것을 JPA에게 알려준다.
                  
                * `@CollectionTable` : 새롭게 생성되는 테이블에 대한 매핑 정보를 입력한다.
        
                    * `joinColumns` : 해당 속성으로 지정한 컬럼을 새롭게 생성되는 테이블의 외래 키(FK)로 지정한다.
    
                * favoriteFoods처럼 값으로 사용되는 컬럼이 하나면 `@Column`을 사용해서 컬럼명을 지정 할 수 있다.
    
                * addressHistory에 대한 테이블 매핑 정보는 `@AttributeOverride`를 사용해서 재정의 할 수 있다.

            * 값 타입 컬렉션을 사용하기

                * 값 타입 저장

                    ```java
                    Member member = new Member();
                    member.setUsername("member1");
                    member.setHomeAddress(new Address("homeCity", "street", "10000"));
                    
                    member.getFavoriteFoods().add("치킨");
                    member.getFavoriteFoods().add("족발");
                    member.getFavoriteFoods().add("피자");
                    
                    member.getAddressHistory().add(new Address("old1", "street", "10000"));
                    member.getAddressHistory().add(new Address("old2", "street", "10000"));
                    
                    em.persist(member);
                    ```
        
                    * 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.
                    
                        * member만 영속 상태로 만들어도 값 타입 컬렉션은 자동으로 DB에 반영된다.

                * 값 타입 조회 및 수정

                    ```java
                    /*
                     * 값 타입 조회
                     * */
                    Member findMember = em.find(Member.class, member.getId());
                    
                    /*
                     * 값 타입 수정
                     * */
                    // findMember.getHomeAddress().setCity("newCity"); // 잘못된 방식 (X)
                    Address a = findMember.getHomeAddress();
                    findMember.setHomeAddress(new Address("newCity", a.getStreet(), a.getZipcode())); // 올바른 방식 (O)
                    
                    // 좋아하는 음식을 "치킨"에서 "한식"으로 변경하기
                    findMember.getFavoriteFoods().remove("치킨");
                    findMember.getFavoriteFoods().add("한식");
                    
                    // 주소를 변경하기 (eqauls()와 hashCode()가 오버라이딩 되어 있어야 함)
                    findMember.getAddressHistory().remove(new Address("old1", "street", "10000"));
                    findMember.getAddressHistory().add(new Address("newCity1", "street", "10000"));
                    ```
                  
                    * **값 타입을 수정할 때는 값 타입 인스턴스 자체를 대체해야 한다.**
    
                        * 값 타입 안의 필드 하나만 변경하는 것은 잘못된 방식이다.

        * 값 타입 컬렉션의 제약사항
    
            * 값 타입은 엔티티와 다르게 식별자 개념이 없다. 값은 변경하면 추적이 어렵다.
    
            * **값 타입 컬렉션에 변경사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.**

                * Ex) `delete from ADDRESS where MEMBER_ID=?`
    
            * 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키(PK)를 구성해야 한다.
    
                * 컬럼에 `null`을 입력 할 수 없고, 같은 값을 중복해서 저장 할 수 없다.

            * 값 타입 컬렉션을 사용하는 경우는 다음과 같다.
              
                * 값을 추적할 필요가 없고 정말 단순한 경우에만 값 타입 컬렉션을 사용한다.
    
                    * Ex) 셀렉트 박스에 내가 좋아하는 음식을 여러 개를 체크 할 수 있도록 하는 경우

                * 하지만 대부분은 엔티티를 사용한다.
    
                    * Ex) 주소 이력 관리

        * 값 타입 컬렉션 대안
    
            * 설명
    
                * 실무에서는 `값 타입 컬렉션` 대신에 **일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용**한다. (값 타입을 엔티티로 승급)
                
                    * 일대다 단방향 매핑은 다른 테이블에 외래 키가 있기 때문에 update 쿼리가 발생한다.
                
                    * 다대일 일대다 양방향 매핑을 하면 update 쿼리가 발생하지 않는다.
                
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
                    
                        // 일대다 단방향 매핑
                        @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
                        @JoinColumn(name = "MEMBER_ID") // FK로 사용
                        private List<AddressEntity> addressHistory = new ArrayList<>();
                    
                    }
                    ```

### 2. 값 타입과 불변 객체 

* (1) 개요

    * 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다.

    * 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

* (2) 값 타입 공유 참조

    * 임베디드 타입과 같은 "값 타입"을 여러 엔티티에서 공유하면 위험하다.
   
    * 그 이유는 부작용(side effect)이 발생하기 때문이다.
    
        ```java
        Address address = new Address("city", "street", "10000");
        
        // 회원 1
        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setHomeAddress(address);
        em.persist(member1);
        
        // 회원 2
        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setHomeAddress(address);
        em.persist(member2);
        
        // 회원 1의 주소만 "newCity"로 변경되길 기대했지만 회원 2의 주소도 같이 변경된다.
        member1.getHomeAddress().setCity("newCity");
        ```

        * 회원1, 회원2가 같은 Address 인스턴스를 참조하고 있는 상황에서 city 값을 newCity로 변경하면 회원 1, 2 모두 값이 변경된다.
        
        * 이러한 부작용을 막으려면 값(인스턴스)을 복사해서 사용하면 된다.

* (3) 값 타입 복사

    * 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험하다.
      
    * 대신 값(인스턴스)을 복사해서 사용해야 한다.
    
        ```java
        Address address = new Address("city", "street", "10000");
        
        // 회원 1
        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setHomeAddress(address);
        em.persist(member1);
        
        // 회원 1의 address 값을 복사해서 새로운 copyAddress 값을 생성한 다음, 그 주소로 변경한다.
        Address copyAddress = new Address(address.getCity(),
                address.getStreet(),
                address.getZipcode());
        
        // 회원 2
        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setHomeAddress(copyAddress);
        em.persist(member2);
        
        // 원래 의도한 대로 회원 1의 주소만 변경된다.
        member1.getHomeAddress().setCity("newCity");
        ```

* (4) 객체 타입의 한계

    * 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.

    * 임베디드 타입처럼 **직접 정의한 `값 타입`은 자바의 기본 타입(primitive)이 아니라 객체 타입이다.**
     
    * 자바의 기본 타입은 값을 복사해서 전달한다.

        ```java
        int a = 10;
        int b = a;
        b = 4;
        ```
      
    * 자바의 객체 타입은 항상 참조 값을 전달한다.

        ```java
        Address a = new Address("Old");
        Address b = a;
        b.setCity("New");
        ```
      
        * a와 b는 같은 Address 인스턴스를 가리킨다.

    * 객체 타입은 복사를 하지 않고 원본의 참조 값을 직접 대입하는 것을 막을 방법이 없다.
    
    * 즉, 객체의 공유 참조는 피할 수 없다.

* (5) 불변 객체

    * 값 타입은 부작용 걱정 없이 사용 할 수 있어야 한다. 부작용이 일어나면 값 타입이라 할 수 없다.

    * 객체 타입을 수정 할 수 없게 만들면 부작용을 원천 차단 할 수 있다.

    * 따라서 **값 타입은 불변 객체(immutable object)로 설계해야 한다.**

        * `불변 객체` : **생성 시점 이후 절대 값을 변경할 수 없는 객체**를 말한다.

            * 불변 객체를 구현하는 간단한 방법은 **생성자로만 값을 설정하고 수정자(Setter)를 만들지 않으면 된다.** 
    
                * 또는 수정자를 `private`으로 만들어도 된다.
    
            * [참고] `Integer`, `String`은 자바가 제공하는 대표적인 불변 객체다.

### 3. 값 타입의 비교

* **값 타입은 인스턴스가 달라도 그 안에 있는 값이 같으면 같은 것으로 봐야 한다.**

    ```java
    int a = 10;
    int b = 10;
    ```
  
    ```java
    Address a = new Address("서울시");
    Address b = new Address("서울시");
    ```

* 자바에서 제공하는 객체 비교는 2가지가 있다.

    * ① `동일성(identity) 비교` : 인스턴스의 참조 값을 비교, `==` 사용
    
    * ② `동등성(equivalence) 비교` : 인스턴스의 값을 비교, `equals()` 사용
    
        * **값 타입은 `a.equals(b)`를 사용해서 동등성 비교를 해야 한다.**
        
            * 즉, `equals()` 메소드를 적절하게 재정의해야 한다. (주로 모든 필드 사용)

## 10. 객체지향 쿼리 언어

### 1. 객체지향 쿼리 언어 소개 

* (1) JPA는 다양한 쿼리 방법을 지원

    * JPQL
    
    * JPA Criteria
    
    * QueryDSL
    
    * 네이티브 SQL

    * JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

* (2) JPQL

    * 애플리케이션이 필요한 데이터만 DB에서 가져오려면 결국 검색 조건이 포함된 SQL이 필요하다. 

    * JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.

    * SQL과 문법이 유사하다. (SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원)
    
    * JPQL은 엔티티 객체를 대상으로 쿼리한다.
    
    * SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
    
* (3) Criteria

    * JPQL을 생성하는 빌더 역할을 한다.
    
    * 문자가 아닌 자바 코드로 JPQL을 작성할 수 있다.

    * **단점: 너무 복잡하고 실용성이 없다.** 
    
        * `Criteria` 대신에 `QueryDSL` 사용 권장
    
* (4) QueryDSL

    * JPQL을 생성하는 빌더 역할을 한다.

    * 문자가 아닌 자바 코드로 JPQL을 작성할 수 있다.
   
    * 컴파일 시점에 문법 오류를 찾을 수 있음
    
    * 동적 쿼리를 작성하는데 편리하다.
    
        * `동적 쿼리`는 상황에 따라 분기 처리를 통해 SQL을 동적으로 만드는 것을 말한다.
    
    * 단순하고 쉽기 때문에 실무에서 사용하는 것을 권장한다.

    * [참고] 레퍼런스 문서
    
        * `영어` : http://www.querydsl.com/static/querydsl/4.4.0/reference/html_single/
          
        * `한글` : http://www.querydsl.com/static/querydsl/4.0.1/reference/ko-KR/html_single/

* (5) 네이티브 SQL

    * SQL을 직접 사용 할 수 있는 기능을 제공한다.
    
    * JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 사용할 때 이용한다.
     
    * 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

        ```java
        String sql =
            "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = ‘kim’";
        List<Member> resultList =
                    em.createNativeQuery(sql, Member.class).getResultList();
        ```

* (6) JDBC 직접 사용, SpringJdbcTemplate 등

    * JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스 등을 함께 사용 할 수 있다.
    
    * 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시를 할 필요가 있다.
    
        * Ex) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 `flush()` 해야 한다.

### 2. JPQL

* JPQL(Java Persistence Query Language, 자바 영속성 쿼리 언어)

    * `JPQL`는 테이블이 아닌 엔티티 객체를 대상으로 검색하는 객체지향 쿼리 언어이다.
    
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
    
        * `이름 기준 파라미터`는 이름을 기준으로 파라미터를 구분하는 방법이다.
        
            * 문법 : `:이름`
            
            * 예시
              
                ```java
                TypedQuery<Member> query = em.createQuery("select m from Member m where m.username = :username", Member.class);
                query.setParameter("username", "member1");
                Member singleResult = query.getSingleResult();
                
                System.out.println("singleResult = " + singleResult.getUsername());
                ```

                * `이름 기준 파라미터 바인딩 방식`을 사용하는 것이 좋다.

        * `위치 기준 파라미터`는 위치를 기준으로 파라미터를 구분하는 방법이다.

            * 문법 : `?위치 값`  (위치 값은 1 부터 시작함)        

            * 예시
              
                ```java
                Member singleResult = em.createQuery("select m from Member m where m.username = ?1", Member.class)
                        .setParameter(1, "member1")
                        .getSingleResult();
        
                System.out.println("singleResult = " + singleResult.getUsername());
                ```
              
* 프로젝션

    * `프로젝션(projection)`은 SELECT 절에 조회할 대상을 지정하는 것을 말한다.
    
        * 프로젝션 대상은 엔티티, 임베디드 타입, 스칼라 타입이 있다.
        
            * 스칼라 타입은 숫자, 문자 등 기본 데이터 타입을 의미한다.
            
        * DISTINCT로 중복을 제거 할 수 있다.
        
        * 엔티티 프로젝션으로 조회한 엔티티는 영속성 컨텍스트에서 관리된다.
    
    * 예시
    
        ```java
        SELECT m FROM Member m                    // Member 엔티티를 조회한다. [엔티티 프로젝션] 
        SELECT m.team FROM Member m               // Member 엔티티와 연관된 Team 엔티티를 조회한다. [엔티티 프로젝션]
        SELECT m.address FROM Member m            // Member 엔티티의 Address 임베디드 타입을 조회한다. [임베디드 타입 프로젝션]
        SELECT m.username, m.age FROM Member m    // [스칼라 타입 프로젝션]
        ```
      
        * [참고] `SELECT a FROM Address a`처럼 FROM 절에 임베디드 타입을 사용하는 것은 불가능하다.

    * 프로젝션에서 여러 값을 조회하는 방법
    
        * ① `Query` 타입으로 조회

            ```java
            Query query = em.createQuery("select m.username, m.age from Member m");
            ```

        * ② `Object[]` 타입으로 조회

            ```java
            List<Object[]> resultList = em.createQuery("select distinct m.username, m.age from Member m")
                    .getResultList();

            for (Object[] row : resultList) {
                System.out.println("username = " + row[0]); // username = member1
                System.out.println("age = " + row[1]);  // age = 10
            }
            ```
        
        * ③ `new` 명령어로 조회 (여러 개의 단순 값을 DTO로 바로 조회)
        
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

        * `setFirstResult(int startPosition)` : 조회 시작 위치를 지정한다. (0 부터 시작)
        
        * `setMaxResults(int maxResult)` : 조회할 데이터 수를 지정한다.
        
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

    * 내부 조인은 `[INNER] JOIN`을 사용한다. (`INNER`는 생략 가능)
    
        ```java          
        // 내부 조인
        String query = "select m from Member m inner join m.team t";
        List<Member> result = em.createQuery(query, Member.class)
                .getResultList();
        ```
      
        * JPQL 조인은 연관 필드를 사용한다.
        
            * `연관 필드`는 다른 엔티티와 연관관계를 가지기 위해 사용하는 필드를 말한다.
    
                * Ex) `Member m inner join m.team t` : 회원이 가지고 있는 연관 필드로 팀과 조인한다.
            
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
      
        * 카테시안 곱이라고도 한다.
      
    * ON 절
    
        * ON 절은 **조인 대상을 필터링**하고 조인할 때, 사용한다.
        
            * Ex) 회원과 팀을 조인하면서, 팀 이름이 A인 것만 조회한다.
            
                ```java          
                String query = "select m from Member m left join m.team t on t.name = 'teamA'";
                ```
        
        * **연관관계가 없는 엔티티를 외부 조인**할 때도, 사용한다. (하이버네이트 5.1 부터)
        
            * Ex) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
            
                ```java          
                String query = "select m from Member m left join Team t on m.username = t.name";
                ```
              
* 서브 쿼리

    * `서브 쿼리(Sub Query)`은 쿼리문 안에 포함되어 있는 또 다른 쿼리문을 말한다.
    
        * JPQL는 WHERE, HAVING 절에서만 서브 쿼리를 사용 할 수 있다.
        
            * 하이버네이트는 SELECT 절의 서브 쿼리를 지원한다.
            
            * FROM 절의 서브 쿼리 (인라인 뷰)는 지원하지 않는다.
            
                * 조인으로 해결 할 수 있다면 조인을 사용한다.
                
                * 또는 네이티브 쿼리로 해결 할 수도 있다.
            
        * 예시
          
            * 나이가 평균보다 많은 회원
            
                ```java          
                select m from Member m
                where m.age > (select avg(m2.age) from Member m2)
                ```

                * 위와 같이 메인 쿼리와 서브 쿼리가 전혀 관계가 없는 쿼리는 성능이 좋다.

            * 한 건이라도 주문한 고객
            
                ```java          
                select m from Member m
                where (select count(o) from Order o where m = o.member) > 0
                ```
              
                * 위와 같이 메인 쿼리와 서브 쿼리가 연관성이 있는 쿼리는 성능이 좋지 않다.

    * 서브 쿼리 지원 함수
 
        * EXISTS
        
            * `[NOT] EXISTS (subquery)` : 서브 쿼리에 결과가 존재하면 참이다. (NOT은 반대) 

            * 예시
            
                ```java          
                // 팀A에 소속인 회원
                select m from Member m 
                where exists ( select t from m.team t WHERE t.name = '팀A')
                ```

        * {ALL | ANY | SOME} 
        
            * `{ALL | ANY | SOME} (subquery)` : 비교 연산자와 같이 사용한다.
            
                * `ALL` : 조건을 모두 만족하면 참이다.
                
                * `ANY`, `SOME`: 조건을 하나라도 만족하면 참이다. (둘은 같은 의미)

            * 예시
            
                ```java          
                // 전체 상품 각각의 재고보다 주문량이 많은 주문들
                select o from Order o
                where o.orderAmount > ALL (select p.stockAmount from Product p)
              
                // 어떤 팀이든 팀에 소속된 회원 
                select m from Member m
                where m.team = ANY (select t from Team t)
                ```

        * IN
        
            * `[NOT] IN (subquery)` : 서브 쿼리의 결과 중 하나라도 같은 것이 있으면 참이다.
    
            * 예시

                ```java
                // 20세 이상을 보유한 팀      
                select t from Team t
                where t IN (select t2 from Team t2 JOIN t2.members m2 where m2.age >= 20)
                ```

* 조건식

    * 타입 표현 - 대소문자 구분하지 않음
    
        * 문자
        
            * 문자는 작은 따옴표 사이에 표현한다.
            
            * 작은 따옴표를 표현하고 싶으면 작은 따옴표 2개('')를 사용한다.
            
                * Ex) `'HELLO'`, `'She''s'`
            
        * 숫자
        
            * `L` : Long 타입으로 지정한다.
            
            * `D` : Double 타입으로 지정한다.
            
            * `F` : Float 타입으로 지정한다.
            
                * Ex) `10L`, `10D`, `10F`

        * 날짜
        
            * `{d 'yyyy-mm-dd'}` : DATE
            
            * `{t 'hh:mm:ss'}` : TIME 

            * {ts 'yyyy-mm-dd hh:mm:ss.f} : TIMESTAMP

                * Ex) `m.createDate = {d ‘2021-01-03’}`
           
        * Boolean
        
            * `TRUE` 또는 `FALSE`
            
        * ENUM
        
            * ENUM은 (하드 코딩 시) 패키지 명을 포함한 전체 이름을 사용해야 한다.
            
                * Ex) `jpql.MemberType.ADMIN`

                    * [참고] 엔티티에서 enum 타입을 선언할 때는 `@Enumerated(EnumType.STRING)`을 사용해야 한다.

                * Ex) `setParameter()`를 사용하면 패키지 명을 포함하지 않아도 된다.

                    ```java          
                    String query = "select m.username, 'HELLO', true from Member m " + 
                    "where m.type = :userType";
                    
                    em.createQuery(query)
                        .setParameter("userType", MediaType.ADMIN)
                        .getResultList();
                    ```

        * 엔티티 타입
        
            * TYPE은 엔티티의 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 사용한다. 

            * 주로 상속 관계에서 사용함. (잘 사용하지 않음)
            
                * Ex) `TYPE(i) = Book`

                    ```java          
                    em.createQuery("select i from Item i where type(i) = Book", Item.class);
                    ```

    * 연산자
    
        * 논리 연산
    
            * `AND` : 둘 다 만족하면 참
    
            * `OR` : 둘 중 하나만 만족하면 참
    
            * `NOT` : 조건식의 결과 반대
    
        * 비교식
    
            * `=`, `>`, `>=`, `<`, `<=`, `<>`
    
        * BETWEEN, IN, LIKE, IS NULL
    
            * `필드 [NOT] BETWEEN A AND B` : 필드는 A ~ B 사이의 값이면 참
    
            * `필드 [NOT] IN(목록)` : 필드와 같은 값이 목록에 하나라도 있으면 참이다.

            * `필드 [NOT] LIKE 패턴값` : 필드와 패턴 값을 비교한다.
    
                * `%` : 아무 값들이 입력되어도 된다. (값이 없어도 됨)
    
                * `_` : 한 글자는 아무 값이 입력되어도 되지만, 값이 있어야 한다.

            * `필드 IS [NOT] NULL` : NULL 인지 비교한다.
    
                * NULL은 =가 아닌 IS NULL로 비교해야 한다.

    * CASE 식
    
        * 기본 CASE 식
        
            * 문법
            
                ```java          
                CASE  
                    WHEN <조건식> THEN <스칼라식>  
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
                    WHEN <스칼라식1> THEN <스칼라식2>
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
            
            * 설명 : 두 값이 같으면 null을 반환하고 다르면 첫 번째 값을 반환한다.

            * 예시
            
                ```java
                // 사용자 이름이 '관리자'면 null을 반환하고 나머지는 본인의 이름을 반환한다. (관리자의 이름을 숨길 때 사용)
                select NULLIF(m.username, '관리자') from Member m
                ```

* JPQL 기본 함수

    * 문자 함수
    
        * `CONCAT(문자1, 문자2)` : 문자를 더한다.
        
            * Ex) `CONCAT('A', 'B')` = AB
            
        * `SUBSTRING(문자, 위치 [,길이])` : 위치부터 시작해서 길이만큼 문자를 구한다. 길이 값이 없으면 나머지 전체 길이를 의미한다.
        
            * Ex) `SUBSTRING(‘ABCDEF’, 2, 3)` = BCD
            
        * `TRIM(문자)` : TRIM 문자를 제거한다. (LEADING : 왼쪽만, TRAILING : 오른쪽만, BOTH : 양쪽, TRIM 문자의 기본 값은 공백)
        
            * Ex) `TRIM('ABC')` = 'ABC'
            
        * `LOWER(문자)` : 소문자로 변경한다.
        
            * Ex) `LOWER('ABC')` = abc

        * `UPPER(문자)` : 대문자로 변경한다.
        
            * Ex) `UPPER('abc')` = ABC

        * `LENGTH(문자)` : 문자 길이를 알려준다.
        
            * Ex) `LENGTH('ABC')` = 3
            
        * `LOCATE(찾을 문자, 원본 문자 [, 검색 시작 위치])` : 원본 문자에서 찾는 문자의 위치를 반환한다. 1 부터 시작하며 못 찾으면 0을 반환한다.
        
            * Ex) `LOCATE('DE', 'ABCDEFG')` = 4

    * 수학 함수
    
        * `ABS(수학식)` : 절대값을 구한다.
        
            * Ex) `ABS(-10)` = 10
            
        * `SQRT(수학식)` : 제곱근을 구한다
        
            * Ex) `SQRT(4)` = 2.0
            
        * `MOD(수학식, 나눌 수)` : 나머지를 구한다
        
            * Ex) `MOD(4, 3)` = 1
            
        * `SIZE(컬렉션 값 연관 경로식)` : 컬렉션의 크기를 구한다
        
            * Ex) `SIZE(t.members)`

    * 날짜 함수
    
        * `CURRENT_DATE` : 현재 날짜
        
        * `CURRENT_TIME` : 현재 시간 
        
        * `CURRENT_TIMESTAMP` : 현재 날짜 + 시간

        * 하이버네이트는 날짜 타입에서 년,월,일,시간,분,초 값을 구하는 기능을 지원한다.
        
            ```java
            select year(CURRENT_TIMESTAMP), month(CURRENT_TIMESTAMP), day(CURRENT_TIMESTAMP) from Member;
            ```
          
    * JPA 용도
    
        * `SIZE` : 컬렉션의 크기를 반환한다.

            ```java
            select size(t.members) from Team t;
            ```

        * `INDEX` : `@OrderColumn`을 사용할 때, 컬렉션의 위치 값을 반환한다. (`@OrderColumn` 자체를 사용하지 않는 것을 권장함)
        
            ```java
            select index(t.members) from Team t;
            ```

* 사용자 정의 함수 호출

    * JPA는 사용자 정의 함수를 지원한다.
    
        * ① 자신이 사용하는 DB 방언(Dialect) 클래스를 상속받은 다음, 사용자 정의 함수를 등록한다.

            ```java
            public class MyH2Dialect extends H2Dialect{
          
                public MyH2Dialect(){
                    registerFunction("group_concat", new StandardFunction("group_concat", StandardBasicTypes.STRING));
                }
          
            }
            ```
          
        * ② `persistence.xml`에서 기존의 방언 클래스를 상속받은 방언 클래스로 대체한다.

            ```html
            <property name="hibernate.dialect" value="dialect.MyH2Dialect" />
            ```

        * ③ 사용자 정의 함수는 다음과 같이 사용하면 된다.
        
            * JPA 문법

                ```java
                select function('group_concat', m.username) from Member m
                ```
              
            * 하이버네이트 문법

                ```java
                select group_concat(m.username) from Member m
                ```
              
* 경로 표현식

    * `경로 표현식`은 `.(점)`을 찍어 객체 그래프를 탐색하는 것을 말한다.
    
        ```java
        SELECT m.username     // 상태 필드
        FROM Member m
            JOIN m.team t     // 단일 값 연관 필드
            JOIN m.orders o   // 컬렉션 값 연관 필드
        WHERE t.name = '팀A';
        ```
      
    * 경로 표현식 용어 정리
    
        * `상태 필드(state field)` : 단순히 값을 저장하기 위한 필드 
        
            * Ex) m.username
        
        * `연관 필드(association field)` : 연관 관계를 위한 필드
        
            * `단일 값 연관 필드` : 대상이 엔티티인 필드를 말한다. (@ManyToOne, @OneToOne)
            
                * Ex) m.team
            
            * `컬렉션 값 연관 필드` : 대상이 컬렉션인 필드를 말한다. (@OneToMany, @ManyToMany)   
          
                * Ex) m.orders
                
    * 경로 표현식의 특징
    
        * `상태 필드 경로` : 경로 탐색의 끝이다. 더 이상 탐색 할 수 없다.
    
            * Ex) `select m.username from Member m`
    
                * 예를 들면 `m.username`에서 점(.)을 찍더라도 더 이상 탐색 할 수 없다. 
        
        * `단일 값 연관 경로` : **묵시적으로 내부 조인이 발생한다.** 계속 탐색 할 수 있다.

            * Ex) `select m.team from Member m`

                * 예를 들면 `m.team`에서 점(.)을 찍어서 `m.team.name`으로 더 탐색 할 수 있다.

        * `컬렉션 값 연관 경로`: **묵시적으로 내부 조인이 발생한다.** 더 이상 탐색 할 수 없다.

            * [묵시적 조인] `select t.members from Team t`
    
                * `t.members.size` 처럼 size만 사용 할 수 있음
                
            * [명시적 조인] `select m.username from Team t join t.members m`
    
                * 컬렉션은 경로 탐색의 끝이다. 컬렉션에서 경로 탐색을 하고 싶으면 명시적 조인을 통해 별칭을 얻어야 한다.
    
                    * `join t.members m`으로 컬렉션에 새로운 별칭을 얻었다. 이제 별칭 m 부터 다시 경로 탐색을 할 수 있다.
                
        * 정리하기
          
            * 위의 내용 보다는 아래 내용을 확실히 기억하자. 
              
            * **묵시적 조인 보다는 항상 명시적 조인을 사용하자.**
    
                * **묵시적 조인은 조인이 일어나는 상황을 파악하기 어렵다.**

    * 명시적 조인, 묵시적 조인
    
        * `명시적 조인` : JOIN 키워드를 직접 사용하는 것을 의미한다.
        
            * Ex) `select m from Member m join m.team t`
            
                * Member와 연관된 team과 조인을 하라는 의미다. 
        
        * `묵시적 조인` : 경로 표현식에 의해 묵시적으로 조인이 발생하는 것을 의미한다. (내부 조인만 가능하다.)
        
            * Ex) `select m.team from Member m`
            
* 페치 조인(fetch join)

    * `페치 조인`은 **연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능**이다.
    
        * SQL 조인의 종류가 아니다.
        
        * **JPQL에서 성능 최적화를 위해 제공하는 기능이다.**
    
        * `join fetch` 명령어를 사용한다.
        
            * `[ LEFT [OUTER] | INNER ] JOIN FETCH` 조인경로
            
        * `N+1 문제`는 즉시 로딩, 지연 로딩 둘 다에서 발생하는 문제다.
        
            * 해결 방법으로 페치 조인을 사용 할 수 있다.
            
    * 엔티티 페치 조인
    
        * 회원을 조회 하면서 연관된 팀도 SQL 한 번에 함께 조회한다.
        
            ```java
            // JPQL
            select m from Member m join fetch m.team
            ```

            ```sql
            // SQL
            SELECT M.*, T.* 
            FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID = T.ID
            ```

        * 위의 예제를 좀 더 자세히 살펴보자. (다대일 관계)
        
            ```java
            /*
            * 회원 1, 팀 A
            * 회원 2, 팀 A
            * 회원 3, 팀 B
            * */
            
            // 팀(Team)
            Team teamA = new Team();
            teamA.setName("팀A");
            em.persist(teamA);
            
            Team teamB = new Team();
            teamB.setName("팀B");
            em.persist(teamB);
            
            // 회원(Member)
            Member member1 = new Member();
            member1.setUsername("회원1");
            member1.setTeam(teamA);
            em.persist(member1);
            
            Member member2 = new Member();
            member2.setUsername("회원2");
            member2.setTeam(teamA);
            em.persist(member2);
            
            Member member3 = new Member();
            member3.setUsername("회원3");
            member3.setTeam(teamB);
            em.persist(member3);
            
            em.flush();
            em.clear();
            
            String query =  "select m from Member m join fetch m.team";
            
            List<Member> result = em.createQuery(query, Member.class)
                    .getResultList();
            
            for (Member member : result) {
                // 회원과 팀의 이름을 같이 출력한다. 페치 조인으로 지연 로딩이 아닌 연관된 엔티티를 함께 조회한다.
                System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());
            }
            ```
      
    * 컬렉션 페치 조인
    
        * 일대다 관계인 컬렉션에서도 페치 조인을 사용 할 수 있다.

            ```java
            // JPQL        
            select t from Team t join fetch t.members where t.name = '팀A'
            ```

            ```sql
            // SQL        
            SELECT T.*, M.*
            FROM TEAM T INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
            WHERE T.NAME = '팀A'
            ```

        * 위의 예제를 좀 더 자세히 살펴보자. (일대다 관계)

            ```java
            String query =  "select t from Team t join fetch t.members";
            
            List<Team> result = em.createQuery(query, Team.class).getResultList();
            
            for (Team team : result) {
                System.out.println("team = " + team.getName() + " | members = " + team.getMembers().size());
                
                for(Member member : team.getMembers()){
                    //페치 조인으로 팀과 회원을 함께 조회해서 지연 로딩이 발생하지 않는다.
                    System.out.println("-> member = " + member);
                }
            }
            ```
                      
            * 일대다 조인의 결과는 데이터가 더 많아 질 수 있다. **[주의!]** 

                * TEAM (1)
        
                    | ID (PK) | NAME |
                    |:-------:|:----:|
                    |    1    |  팀A |
                    |    2    |  팀B |
                    |    3    |  팀C |

                * TEAM (多)
                
                    | ID (PK) | TEAM_ID (FK) |  NAME |
                    |:-------:|:------------:|:-----:|
                    |    1    |       1      | 회원1 |
                    |    2    |       1      | 회원2 |
                    |    3    |       2      | 회원3 |
                    |    4    |     null     | 회원4 |

                * TEAM과 MEMBER를 조인 했을 때, 팀 A의 데이터는 다음과 같다.
                    
                    | ID (PK) | NAME | ID (PK) | TEAM_ID (FK) |  NAME |
                    |:-------:|:----:|:-------:|:------------:|:-----:|
                    |    1    |  팀A |    1    |       1      | 회원1 |
                    |    1    |  팀A |    2    |       1      | 회원2 |

            * 다대일 조인의 결과는 데이터가 같거나 줄어든다. 
            
    * 페치 조인과 DISTINCT
    
        * JPQL의 `DISTINCT`는 SQL에 DISTINCT를 추가하고 애플리케이션에서 엔티티의 중복을 제거한다.

            ```java        
            select distinct t
            from Team t join fetch t.members where t.name = '팀A'
            ```
          
            * SQL에 DISTINCT를 추가하더라도 데이터가 다르므로 SQL 결과에서는 중복 제거에 실패한다. (SQL은 완전히 같아야 중복이 제거 됨)
            
                * TEAM_ID: 1, TEAM_NAME : 팀A, **MEMBER_ID : 1**, TEAM_ID : 1, NAME : 회원1
                
                * TEAM_ID: 1, TEAM_NAME : 팀A, **MEMBER_ID : 2**, TEAM_ID : 1, NAME : 회원2
            
            * DISTINCT가 추가로 애플리케이션에서 중복 제거를 시도한다. 그리고 같은 식별자를 가진 Team 엔티티를 제거한다.
                        
    * 페치 조인과 일반 조인의 차이
    
        * ① 일반 조인을 실행하면 연관된 엔티티를 함께 조회하지 않는다.

            ```java
            // JPQL       
            select t
            from Team t join t.members m where t.name = '팀A'
            ```

            ```sql
            // SQL
            SELECT T.*
            FROM TEAM T INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
            WHERE T.NAME = '팀A'      
            ```

        * ② JPQL은 결과를 반환할 때 연관관계까지 고려하지 않는다. 
    
        * ③ 단지 SELECT 절에 지정한 엔티티만 조회할 뿐이다.
            
            * 팀 엔티티만 조회하고, 연관된 회원 컬렉션은 조회하지 않는다.
            
            * 만약 회원 컬렉션을 지연 로딩으로 설정하면 프록시나 아직 초기화하지 않은 컬렉션 래퍼를 반환한다.
    
        * ④ 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회한다. (즉시 로딩) 
        
            * 즉, **페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념이다.** 
        
    * 페치 조인의 특징과 한계
    
        * ① fetch join의 대상은 on, where 등에서 필터링 조건으로 사용하면 안된다.
    
            * 아래 예시를 살펴보며 좀 더 자세히 이해해보자.

                * 1번. 
                  
                    * `select t from Team t join fetch t.members m where m.age > 10` (X)
    
                        * 페치 조인 대상에 별칭을 주고 where 조건으로 사용하면 Collection 형태로 조회되는 데이터가 전부 조회되지 않고 일부만 나오기 때문에 문제가 생길 수 있다.

                * 2번.
    
                    * `select t from Team t join fetch t.members m where t.name=:teamName` (O)

                        * 해당 쿼리에서 `Team`은 fetch join 대상이 아니므로 where에서 마음껏 사용해도 된다.

                        * 하지만 `t.members m`은 페치 조인 대상이다. 따라서 on, where 등에서 필터링 조건으로 사용하면 위험하다.

                * 3번.
    
                    ```java
                    @Entity
                    public class Member {
                    
                        @Id @GeneratedValue
                        private Long id;
                        private String username;
                    
                        @ManyToOne
                        @JoinColumn(name = "team_id")
                        private Team team;
                    
                    }
                    ```

                    ```java
                    @Entity
                    public class Team {
                    
                        @Id @GeneratedValue
                        private Long id;
                    
                        private String name;
                    
                        @OneToMany(mappedBy = "team")
                        private List<Member> members = new ArrayList<>();
                        
                    }
                    ```
                    
                    ```java
                    @RunWith(SpringRunner.class)
                    @SpringBootTest
                    @Transactional
                    public class DemoApplicationTests {
                    
                        @Autowired
                        EntityManager em;
                    
                        @Test
                        public void contextLoads() {
                    
                            Team team = new Team();
                            team.setName("teamA");
                            em.persist(team);
                    
                            Member member1 = new Member();
                            member1.setUsername("m1");
                            member1.setTeam(team);
                            em.persist(member1);
                    
                            Member member2 = new Member();
                            member2.setUsername("m2");
                            member2.setTeam(team);
                            em.persist(member2);
                    
                            em.flush();
                            em.clear();
                    
                            List<Team> result = em.createQuery("select t from Team t join fetch t.members m where m.username = 'm1'", Team.class)
                                    .getResultList();
                    
                            for (Team team1 : result) {
                                System.out.println("team1 = " + team1.getName());
                                List<Member> members = team1.getMembers();
                                for (Member member : members) {
                                    System.out.println("member = " + member.getUsername());
                                }
                            }
                    
                        }
                    
                    }
                    ```
                  
                    * 실행 결과를 보면 member가 하나만 존재한다.

                        * `team1 = teamA`

                        * `member = m1`

                    * 애플리케이션에서 fetch join의 결과는 연관된 모든 엔티티가 있을 것이라 가정하고 사용해야 한다. 

                    * 이렇게 페치 조인에 별칭을 잘못 사용해서 컬렉션 결과를 필터링하면, 객체의 상태와 DB의 상태 일관성이 깨져버린다.

                    * [결론] **fetch join의 대상은 on, where 등에서 필터링 조건으로 사용하면 안된다.**

                * 4번. 
                  
                    * `select t from Team t join fetch t.members m join fetch m.team` (O)
    
                        * 페치 조인을 여러 단계로 하는 경우에는 페치 조인의 대상에 별칭을 줄 수 있다.
    
        * ② 둘 이상의 컬렉션은 페치 조인을 할 수 없다.

            * JPA 구현체에 따라 가능한 경우도 있는데 가급적 사용하면 안된다.

            * `컬렉션 * 컬렉션`의 카테시안 곱이 만들어지므로 주의해야 한다.

        * ③ 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용 할 수 없다.
        
            * 일대일, 다대일과 같은 단일 값 연관 필드들은 페치 조인을 사용해도 페이징 API를 사용 할 수 있다.
    
                * 일대다는 조심하자! 데이터가 더 많아 질 수 있다.
                 
            * 하이버네이트에서 컬렉션을 페치 조인하고 페이징 API를 사용하면 경고 로그를 남기면서 메모리에서 페이징 처리를 한다.

                ```java
                // [잘못된 방식] 일대다 페치 조인         
                select t from Team t join fetch t.members m
                
                List<Team> result = em.createQuery(query, Team.class)
                        .setFirstResult(0)
                        .setMaxResults(1)
                        .getResultList();
                ```

                * 성능 이슈와 메모리 초과 예외가 발생 할 수 있어서 매우 위험하다.

            * 해결 방법은 다음과 같다.
            
                * 일대다 페치 조인을 다대일 페치 조인으로 뒤집어서 해결 할 수 있다.
           
                    ```java 
                    // [올바른 방식] 다대일 페치 조인       
                    select m from Member m join fetch m.team
                    
                    List<Team> result = em.createQuery(query, Team.class)
                            .setFirstResult(0)
                            .setMaxResults(1)
                            .getResultList();
                    ```
                  
                * 또는 `@BatchSize`를 사용하여 해결 할 수도 있다. 

                    ```java 
                    @Entity
                    public class Team {
                    
                        //...
                    
                        @BatchSize(size = 100) // 1000 이하 값으로 설정
                        @OneToMany(mappedBy = "team")
                        private List<Member> members = new ArrayList<>();
                        
                    }
                    ```

                    ```java 
                    String query =  "select t from Team t";
                    
                    List<Team> result = em.createQuery(query, Team.class)
                            .setFirstResult(0)
                            .setMaxResults(2)
                            .getResultList();
                    
                    for (Team team : result) {
                        System.out.println("team = " + team.getName() + " | members = " + team.getMembers().size());
                    
                        for(Member member : team.getMembers()){
                            System.out.println("-> member = " + member);
                        }
                    }
                    ```

                    * `@BatchSize` : 연관된 엔티티를 조회할 때 지정된 size 만큼 SQL의 IN 절을 사용해서 조회한다.

                        * 위의 예시에서는 Team의 members는 지연 로딩(LAZY LOADING)으로 설정되어 있다.
    
                        * 지연 로딩을 한 엔티티인 members를 실제 사용할 때, 지정된 size 만큼 SQL의 IN 절을 사용해서 조회한다. 

                            * 조회한 Team의 id를 size 만큼 SQL의 IN 절에 설정해서 실행한다.

                    * `hibernate.default_batch_fetch_size`로 글로벌 설정이 가능하다.

    * 엔티티와 DB 데이터의 일관성
      
        * JPA의 엔티티 데이터는 DB의 데이터와 일관성을 유지해야 한다.
        
            * 임의로 데이터를 제외하고 조회하면 DB에 해당 데이터가 없다고 판단하는 것과 똑같다.
        
        * 예를 들어, 통계 쿼리를 작성해야 한다면, 엔티티가 아닌 그냥 값으로 조회하면 된다.
        
            * 엔티티는 객체 그래프를 유지하고 DB와 데이터 일관성을 유지한다. 하지만 엔티티가 아닌 일반 값들은 그럴 필요가 없다.
              
            * 필요한 값들만 조회해서 DTO로 반환하면 된다. 

    * 정리하기
    
        * ① 연관된 엔티티들을 SQL 한 번으로 조회한다. - 성능 최적화 
        
        * ② 페치 조인은 엔티티에 직접 적용하는 글로벌 로딩 전략 보다 우선적으로 적용된다.
        
            * `@OneToMany(fetch = FetchType.LAZY)` : 글로벌 로딩 전략

        * ③ 실무에서 글로벌 로딩 전략은 모두 지연 로딩으로 설정한다.

        * ④ 페치 조인은 객체 그래프를 유지 할 때 사용하면 효과적이다.
        
        * ⑤ **여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 만들어야 하면, 페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적이다.**
        
            * **Ex) 통계 쿼리**
    
* 다형성 쿼리

    * `TYPE`은 엔티티의 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 사용한다.

        * Ex) Item 중에 Book, Movie를 조회해라

            ```java
            // JPQL        
            select i from Item i
            where type(i) IN (Book, Movie)
            ```

            ```sql
            // SQL        
            select i from i
            where i.DTYPE in ('B', 'M')
            ```

    * `TREAT`은 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.

        * 자바의 타입 캐스팅과 유사하다.

        * JPA 표준은 FROM, WHERE 절에서 사용, 하이버네이트는 SELECT 절에서도 사용 가능

        * Ex) 부모인 Item과 자식 Book이 있다.

            ```java
            // JPQL        
            select i from Item i
            where treat(i as Book).author = 'kim'
            ```

            ```sql
            // SQL        
            select i.* from Item i
            where i.DTYPE = 'B' and i.author = 'kim'
            ```
          
* 엔티티 직접 사용

    * 기본 키 값

        * JPQL에서 엔티티를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.
        
            ```java        
            select count(m.id) from Member m //엔티티의 아이디를 사용
            select count(m) from Member m //엔티티를 직접 사용
            ```
          
            * 엔티티를 식별 할 수 있는 것은 기본 키 값이기 때문이다.
          
        * JPQL의 파라미터로 엔티티를 직접 사용하더라도 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.
    
            ```java        
            String query = "select m from Member m where m = :member"; 
            List resultList = em.createQuery(query)
                                .setParameter("member", member)
                                .getResultList();
            ```
          
    * 외래 키 값

        * 기본 키 값이 1L인 팀 엔티티를 파라미터로 사용하고 있다. m.team은 현재 team_id라는 외래 키와 매핑되어 있다.

        * JPQL의 파라미터로 엔티티를 직접 사용하더라도 SQL에서는 해당 엔티티의 외래 키 값을 사용한다.

            ```java        
            Team team = em.find(Team.class, 1L);
          
            String qlString = "select m from Member m where m.team = :team"; 
            List resultList = em.createQuery(qlString)
                                .setParameter("team", team) .getResultList();
            ```

* Named 쿼리: 정적 쿼리

    * JPQL 쿼리는 동적 쿼리와 정적 쿼리로 나눌 수 있다.

        * `동적 쿼리` : 검색 조건에 따라 실행 시점에 쿼리를 생성하는 것을 말한다. 
        
            * `em.createQuery("select ...")`

        * `정적 쿼리` : 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용하는 것을 말한다. `Named 쿼리`라고도 한다.
        
            * 애플리케이션 로딩 시점에 JPQL 문법을 체크하여 미리 SQL로 파싱해둔다.
    
                * `파싱(Parsing)` : 어떤 데이터를 원하는 형태로 가공하는 것을 말한다.
            
            * 따라서 오류를 빨리 확인할 수 있고 사용하는 시점에는 파싱된 결과를 재사용하므로 성능상 이점이 있다.
            
                ```java        
                @Entity
                @NamedQuery(
                    name = "Member.findByUsername",
                    query="select m from Member m where m.username = :username")
                public class Member {
                    //...
                }
              
                List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                                            .setParameter("username", "회원1")
                                            .getResultList();
                ```
              
                * 스프링 데이터 JPA에서는 `@Query`를 제공한다.
        
* 벌크 연산

    * `벌크 연산`은 JPQL로 한 번에 여러 데이터를 수정하거나 삭제하는 것을 말한다. (SQL의 UPDATE, DELETE 문)

        * UPDATE 벌크 연산
        
            ```java
            // 재고가 10개 미만인 모든 상품의 가격을 10% 상승한다.
            String query = "update Product p " +
                           "set p.prce = p.price * 1.1 " +
                           "where p.stockAmount < :stockAmount";
            
            // JPQL을 실행하기 직전에 flush()가 자동 호출된다.
            int resultCount = em.createQuery(query)
                                .setParameter("stockAmount", 10)
                                .executeUpdate();
          
            em.clear();
            ```
          
            * 벌크 연산(UPDATE, DELETE)은 `executeUpdate()`를 사용한다. 
              
                * 해당 메소드는 벌크 연산으로 영향을 받은 엔티티 건수를 반환한다.

        * DELETE 벌크 연산
        
            ```java
            // 가격이 100원 미만인 상품을 삭제한다.
            String query = "delete from Product p " +
                           "where p.price < :price";
            
            int resultCount = em.createQuery(query)
                                .setParameter("price", 100)
                                .executeUpdate();
          
            em.clear();
            ```
          
    * 벌크 연산의 주의사항

        * **벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다**는 점에 주의해야 한다.

        * 해결책은 다음 2가지가 있다.
        
            * ① 벌크 연산을 가장 먼저 실행한다. 
            
            * ② 벌크 연산을 수행한 후, 바로 영속성 컨텍스트를 초기화한다.
            
                * 영속성 컨텍스트를 초기화하면 이후 엔티티를 조회할 때, 벌크 연산이 적용된 데이터베이스에서 엔티티를 조회한다.
            
    * 스프링 데이터 JPA에서는 벌크 연산을 `@Modifying`으로 제공한다.