# 더 자바, 애플리케이션을 테스트하는 다양한 방법
> 아래 내용은 [더 자바, 애플리케이션을 테스트하는 다양한 방법](https://www.inflearn.com/course/the-java-application-test# "더 자바, 애플리케이션을 테스트하는 다양한 방법")을 참고 하였습니다.

## 1. JUnit 5

#### 1) JUnit 5 소개

* (1) JUnit이란

    * `JUnit`은 Java의 테스팅 프레임워크이다.

        * `JUnit5`는 자바 8 이상을 필요로 함.
    
        * 대체재: TestNG, Spock ...

* (2) JUnit5

    * `JUnit5`는 3개의 모듈로 구성되어 있다.
    
        ![image 1](images/img1.png)
    
        * ① `JUnit Platform` : JUnit으로 작성한 테스트를 실행 해주는 런처를 제공한다. 그리고 TestEngine API를 제공한다.
    
            * IDE가 JUnit Platform을 사용해서 `@Test`이 붙어있는 메서드를 실행해준다.
    
        * ② `Jupiter` : JUnit5를 지원하는 TestEngine API의 구현체이다.
    
        * ③ `Vintage` : JUnit 4와 3을 지원하는 TestEngine API의 구현체이다.
        
#### 2) JUnit 5 시작하기

* (1) 프로젝트 만들기

    * JUnit5를 시작하는 방법은 다음 2가지가 존재한다.

        * ① 스프링 부트 프로젝트 만들기

            * 스프링 부트 2.2 이상 버전으로 프로젝트를 만든다면 기본적으로 JUnit 5 의존성이 추가된다.

        * ② Gradle 또는 Maven 프로젝트 만들기

            * 프로젝트를 생성한 다음, Maven을 사용한다면 `pom.xml`에 다음 의존성을 추가한다.
            
                ```html
                <dependency>
                    <groupId>org.junit.jupiter</groupId>
                    <artifactId>junit-jupiter-engine</artifactId>
                    <version>5.5.2</version>
                    <scope>test</scope>
                </dependency>
                ```
              
* (2) 테스트 코드 작성하기

    * ① Study 클래스를 생성한다.

        ```java
        public class Study {
        }
        ```

    * ② Mac을 사용한다면 단축키(Cmd + Shift + t)를 사용해서 테스트 코드를 작성한다.

    * ③ 테스트 메소드를 작성한다.

        ```java
        class StudyTest {
        
            @Test
            void create(){
                Study study = new Study();
                assertNotNull(study);
                System.out.println("create");
            }
        
            @Test
            @Disabled
            void create1(){
                System.out.println("create1");
            }
        
            @BeforeAll
            static void beforeAll(){
                System.out.println("before all");
            }
        
            @AfterAll
            static void afterAll(){
                System.out.println("after all");
            }
        
            @BeforeEach
            void beforeEach(){
                System.out.println("Before each");
            }
        
            @AfterEach
            void afterEach(){
                System.out.println("After each");
            }
        
        }
        ```

        * `JUnit4`에서는 테스트 클래스와 메소드 모두 public 이어야 한다.
        
        * 위에서 사용된 테스트 관련 애노테이션은 아래에서 살펴본다.
        
* (3) 테스트 관련 기본 애노테이션 (JUnit 5 기준)

    * `@Test` : 테스트 메소드로 선언한다.

    * `@BeforeAll` : 모든 테스트가 시작 되기 전에 단 한번 실행된다.
    
        * 메서드가 static void인 경우만 사용 가능하다.
        
        * private 이어서는 안되며 return 타입은 void여야 한다.

    * `@AfterAll` : 모든 테스트가 끝난 다음에 단 한번 실행된다.
    
        * `@BeforeAll`과 조건 동일
    
    * `@BeforeEach` : 하나의 테스트가 시작되기 전에 매번 실행된다.
    
        * 메소드가 static일 필요는 없다.

    * `@AfterEach` : 하나의 테스트가 끝날 때 마다 매번 실행된다.

    * `@Disabled` : 특정 테스트를 실행 하고 싶지 않은 경우에 사용한다.

* (4) 테스트 관련 기본 애노테이션 (JUnit 4 기준)

    * `@Test` : 테스트 메소드로 선언한다.

    * `@BeforeClass` : 모든 테스트가 시작 되기 전에 단 한번 실행된다.

    * `@AfterClass` : 모든 테스트가 끝난 다음에 단 한번 실행된다.
    
    * `@Before` : 하나의 테스트가 시작되기 전에 매번 실행된다.
    
    * `@After` : 하나의 테스트가 끝날 때 마다 매번 실행된다.

    * `@Ignore` : 특정 테스트를 실행 하고 싶지 않은 경우에 사용한다.

#### 3) 테스트 이름 표기하기

* (1) 기본적인 테스트 이름 표기 방식

    * 테스트를 실행하면 그 결과는 기본적으로 테스트 메소드명으로 표시된다.

    * 아래와 같은 방식으로 테스트 이름을 원하는 이름으로 표시할 수 있다.

* (2) `@DisplayNameGeneration`

    * `@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)`

        * 테스트 이름을 _(언더 스코어)에서 공백 문자로 변경한 이름으로 만든다.

        * 클래스에 사용 가능하다. 

            ```java
            @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
            class StudyTest {
            	...
            }
            ```

    * `@DisplayName`

        * 테스트 이름을 사용자가 정의한 이름으로 만든다.  

        * @Test이 붙어있는 메서드에 사용한다.

        * `@DisplayNameGeneration` 보다 우선 순위가 높다.

            ```java
            @Test
            @DisplayName("스터디 만들기")
            void create_new_study(){
              Study study = new Study();
              assertNotNull(study);
              System.out.println("create");
            }
            ```

* (3) 테스트 실행 방법

    * 테스트 메소드에 포커스를 두고 테스트를 실행하면 해당 메소드만 실행한다.

    * 중립 영역에 포커스를 두고 테스트를 실행하면 테스트 클래스 안에 있는 모든 테스트를 실행한다.
    
#### 4) Assertion

* (1) org.junit.jupiter.api.Assertions.*

    * `assertEquals(expected, actual)` : 기대한 값(expected)이 실제 값(actual)과 같은지 확인한다.
    
    * `assertNotNull(actual)` : 값이 null이 아닌지 확인한다.
    
    * `assertTrue(boolean)` : 다음 조건이 참(true)인지 확인한다.
    
    * `assertAll(executables...)` : 모든 검증을 실행하고 그 중에서 실패한 검증에 대해서만 에러 메시지로 보여준다.
    
        * 여러 개의 assert 문을 각각 람다식으로 작성하여 매개변수로 전달함.
        
    * `assertThrows(expectedType, executable)` : executable을 실행한 결과로 지정한 타입의 예외가 발생하는지 확인한다.
    
    * `assertTimeout(duration, executable)` : 특정 시간 안에 실행이 완료되는지 확인한다.
    
        * 첫 번째 매개변수 : 얼마 만에 끝나야 하는지를 지정한다.
        
        * 두 번째 매개변수 : 실행 할 문장을 람다식으로 지정한다.
           
* (2) 실습하기

    * 실습 1 - Study를 처음 만들면 상태가 DRAFT 인지 확인하는 테스트 코드를 작성한다.
    
        * ① 테스트 코드를 작성한다.
        
            ```java
            @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
            class StudyTest {
            
                @Test
                @DisplayName("스터디 만들기")
                void create_new_study(){
                    Study study = new Study();
                    assertNotNull(study);
                    assertEquals(StudyStatus.DRAFT, study.getStatus());
                }
                
            }
            ```
          
        * ② 열거형 StudyStatus를 작성한다. 
        
            ```java
            public enum StudyStatus {
                DRAFT, STARTED, ENDED
            }
            ```
          
        * ③ Study 클래스에 StudyStatus 타입 변수를 선언하고 getStatus()를 만든다.
        
            ```java
            public class Study {
            
                private StudyStatus status;
            
                public StudyStatus getStatus() {
                    return this.status;
                }
            }
            ```
          
        * ④ 테스트를 실행하면 기대한 값과 실제 값이 다르며 처음 상태가 null인 것을 알 수 있다.
        
        * ⑤ 테스트 실패 시, 메시지(message)를 출력 하도록 할 수 있다.

            * `assertEquals(StudyStatus.DRAFT, study.getStatus(), "스터디를 처음 만들면 상태 값이 DRAFT여야 한다.");`
            
            * 추가적인 내용
            
                * 복잡한 메시지를 생성해야 하는 경우, 다음과 같이 람다식을 사용하면 테스트를 실패한 경우에만 해당 메시지를 만들게 할 수 있다. (성능 향상)
                 
                * `assertEquals(StudyStatus.DRAFT ,  study.getStatus(), () -> "스터디를 처음 만들면 " + StudyStatus.DRAFT + "상태여야 한다." );`
            
        * ⑥ 테스트가 정상적으로 통과 되도록 하려면 기본 값(DRAFT)을 설정하면 된다.
        
            ```java
            public class Study {
            
                private StudyStatus status = StudyStatus.DRAFT;
            
                public StudyStatus getStatus() {
                    return this.status;
                }
            }
            ```
          
    * 실습 2 
    
        * ① 테스트 코드를 작성한다.