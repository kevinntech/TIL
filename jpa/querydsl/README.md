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

