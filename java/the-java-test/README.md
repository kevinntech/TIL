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
    
        * 첫 번째 매개변수(`duration`) : 얼마 만에 끝나야 하는지를 지정한다.
        
        * 두 번째 매개변수(`executable`) : 실행 할 문장을 람다식으로 지정한다.
           
* (2) 실습하기

    * 실습 1 - Study를 처음 만들었을 때, 상태가 DRAFT 인지 확인하는 테스트 코드를 작성한다.
    
        * ① 테스트 코드를 작성한다.
        
            ```java
            @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
            class StudyTest {
            
                @Test
                @DisplayName("스터디 만들기")
                void create_new_study(){
                    // 스터디를 처음 만들었을 때, 상태가 DRAFT인지 확인한다.
                    Study study = new Study();
                    assertNotNull(study);
                    assertEquals(StudyStatus.DRAFT, study.getStatus()); // 테스트에 실패하게 됨
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
          
        * ④ 테스트를 실행하면 기대한 값과 실제 값이 다르며 처음 상태가 `null`인 것을 알 수 있다.
        
        * ⑤ 테스트 실패 시, 메시지(message)를 출력 하도록 할 수도 있다.

            ```java
            assertEquals(StudyStatus.DRAFT, study.getStatus(), "스터디를 처음 만들면 상태 값이 DRAFT여야 한다.");
            ```
         
            * 문자열 연산을 해서 복잡한 메시지를 생성해야 하는 경우, 다음과 같이 람다식을 사용하면 테스트를 실패 했을 때만 해당 메시지를 만들도록 할 수 있다. (성능 향상)

                ```java
                assertEquals(StudyStatus.DRAFT, study.getStatus(), () -> "스터디를 처음 만들면 " + StudyStatus.DRAFT + "상태여야 한다." );
                ```
            
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
    
        * ① Study 클래스에 limit 필드를 추가한 다음, 생성자와 getter를 만든다.

            ```java
            public class Study {
            
                private StudyStatus status;
            
                private int limit;
            
                public Study(int limit) {
                    this.limit = limit;
                }
            
                public StudyStatus getStatus() {
                    return this.status;
                }
            
                public int getLimit() {
                    return limit;
                }
            }
            ```

        * ② 테스트 메서드 내에 다음과 같이 assertTrue()를 호출한다.

            ```java
            @Test
            @DisplayName("스터디 만들기")
            void create_new_study(){
                Study study = new Study(-10);
                assertNotNull(study);
                assertEquals(StudyStatus.DRAFT, study.getStatus(),
                () -> "스터디를 처음 만들면 " + StudyStatus.DRAFT + "상태여야 한다."); // 여기에서 테스트가 실패한다.
            
                assertTrue(study.getLimit() > 0 , "스터디 최대 참석 가능 인원은 0보다 커야 한다.");
            }
            ```

        * ③ 테스트를 실행하면 `assertEquals()`에서 테스트가 실패하기 때문에 `assertTrue()`는 실행 되지 않는다.

        * ④ Study 클래스의 status를 변경한 다음, 테스트를 실행해야 `assertTrue()`에서도 테스트가 실패한다는 것을 그제서야 알 수 있다. 

            * `private StudyStatus status = StudyStatus.DRAFT;`

        * ⑤ 모든 검증을 실행해서 결과를 한 번에 알고 싶다면 `assertAll()`를 사용하면 된다.

            ```java
            @Test
            @DisplayName("스터디 만들기")
            void create_new_study(){
                Study study = new Study(-10);
                assertAll(
                    () -> assertNotNull(study),
                    () -> assertEquals(StudyStatus.DRAFT, study.getStatus(),
                          () -> "스터디를 처음 만들면 " + StudyStatus.DRAFT + "상태여야 한다."),
                    () -> assertTrue(study.getLimit() > 0 , "스터디 최대 참석 가능 인원은 0보다 커야 한다.")
                );
            }
            ```

    * 실습 3
    
        * ① 다음과 같이 Study 클래스의 생성자를 변경한다.

            ```java
            public class Study {
            
                private StudyStatus status;
            
                private int limit;
            
                public Study(int limit) {
                    if(limit < 0){
                        throw new IllegalArgumentException("limit은 0 보다 커야 한다.");
                    }
          
                    this.limit = limit;
                }
            
                public StudyStatus getStatus() {
                    return this.status;
                }
            
                public int getLimit() {
                    return limit;
                }
            }
            ```

        * ② 테스트 메서드를 다음과 같이 변경한다.
        
            * `assertThrows()`를 호출하는데 executable을 실행한 결과로 지정한 타입의 예외가 발생하는지 확인한다.
            
                ```java
                @Test
                @DisplayName("스터디 만들기")
                void create_new_study(){
                    // 두 번째 파라미터의 코드를 실행 했을 때, 해당 예외가 발생하는지 확인한다.
                    IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> new Study(-10) );
                    String message = exception.getMessage(); // 발생한 예외 메시지를 String 타입의 변수에 저장
                    assertEquals("limit은 0 보다 커야 한다.", message); // 기대했던 메시지와 같은지 확인한다.
                }
                ```
              
    * 실습 4
    
        * 테스트 코드를 다음과 같이 작성한다.

            ```java
            @Test
            @DisplayName("스터디 만들기")
            void create_new_study(){
              assertTimeout(Duration.ofSeconds(1), () -> {
                  new Study(10);
                  Thread.sleep(2000);
              }); // 1초 안에 끝나야 한다. Study를 만드는 것은
                  // Thread.sleep(2000)으로 일부러 테스트를 실패 하도록 만든다.
            }
            ```
          
            * `assertTimeout()`는 지정한 timeout를 넘어서도 테스트가 끝날 때까지 기다려야 된다는 단점이 있다. 
              
            * `assertTimeoutPreemptively()`는 지정한 timeout을 지나면 더 이상 기다리지 않고 테스트를 종료 시킨다.
              
                * 단, 해당 메서드는 executable에 ThreadLocal를 사용하는 코드가 있다면 예상치 못한 결과가 발생 할 수 있으므로 주의 해야 한다.

* (3) `AssertJ`, `Hamcrest`, `Truth` 등의 라이브러리를 사용할 수도 있다.

    ```java
    @Test
    @DisplayName("스터디 만들기")
    void create_new_study(){
      Study actual = new Study(10);
      assertThat(actual.getLimit()).isGreaterThan(0); // [AssertJ] 0 보다 큰지 확인
    }
    ```
  
#### 5) 조건에 따라 테스트 실행 하기

* (1) 개요

    * 특정한 조건을 만족하는 경우에 테스트를 실행하는 방법에 대해서 알아본다.
    
    * 즉, 어떤 테스트 코드를 특정 OS, 자바 버전, 환경 변수에 따라 실행 여부를 결정 해야 할 때 사용 할 수 있다.

* (2) org.junit.jupiter.api.Assumptions.*

    * ① `assumeTrue(조건)` : 조건이 true이면 이후 테스트를 진행하고 그렇지 않으면 테스트를 생략한다.
    
        ```java
        @Test
        @DisplayName("스터디 만들기")
        void create_new_study(){
          String test_env = System.getenv("TEST_ENV"); // TEST_ENV의 환경 변수 값을 가져온다.
          System.out.println(test_env);
          assumeTrue("LOCAL".equalsIgnoreCase(test_env));
        
          Study actual = new Study(10);
          assertThat(actual.getLimit()).isGreaterThan(0); // 0보다 큰지 확인
        }
        ```
      
    * ② `assumingThat(조건, 테스트)` : 조건이 true이면 두 번째 인자로 받은 테스트를 수행한다.
      
        ```java
        @Test
        @DisplayName("스터디 만들기")
        void create_new_study(){
            String test_env = System.getenv("TEST_ENV"); // TEST_ENV의 환경 변수 값을 가져온다.
        
            assumingThat("LOCAL".equalsIgnoreCase(test_env), () -> {
                System.out.println("local");
                Study actual = new Study(100);
                assertThat(actual.getLimit()).isGreaterThan(0); // 0보다 큰지 확인
            });
        
            assumingThat("keesun".equalsIgnoreCase(test_env), () -> {
                System.out.println("keesun");
                Study actual = new Study(10);
                assertThat(actual.getLimit()).isGreaterThan(0); // 0보다 큰지 확인
            });
        }
        ```
      
* (3) @Enabled___ 와 @Disabled___

    * 종류
    
        * ① OnOS
        
        * ② OnJre
        
        * ③ IfSystemProperty
        
        * ④ IfEnvironmentVariable
        
        * ⑤ If

    * 예시

        * ① 운영체제
        
            ```java
            @Test
            @DisplayName("스터디 만들기")
            @EnabledOnOs({OS.MAC, OS.LINUX}) // 운영체제(OS)가 MAC, LINUX일 때 테스트 코드를 활성화
            void create_new_study(){
                String test_env = System.getenv("TEST_ENV");
                System.out.println("local");
                Study actual = new Study(10);
                assertThat(actual.getLimit()).isGreaterThan(0);
            }
            
            @Test
            @Disabled
            @DisabledOnOs(OS.MAC) // 운영체제(OS)가 MAC일 때 테스트 코드를 비활성화
            void create_new_study_again(){
                System.out.println("create1");
            }
            ```
          
        * ② 자바 버전
        
            ```java
            @Test
            @DisplayName("스터디 만들기")
            @EnabledOnOs({OS.MAC, OS.LINUX}) // 운영체제(OS)가 MAC, LINUX일 때 테스트 코드를 활성화
            void create_new_study(){
                String test_env = System.getenv("TEST_ENV");
                System.out.println("local");
                Study actual = new Study(10);
                assertThat(actual.getLimit()).isGreaterThan(0);
            }
            
            @Test
            @Disabled
            @DisabledOnOs(OS.MAC) // 운영체제(OS)가 MAC일 때 테스트 코드를 비활성화
            void create_new_study_again(){
                System.out.println("create1");
            }
            ```
          
        * ③ 환경변수
        
            * TEST_ENV라는 환경 변수의 값이 LOCAL과 일치 한다면 테스트 코드를 실행한다.
        
                ```java
                @Test
                @DisplayName("스터디 만들기")
                @EnabledIfEnvironmentVariable(named = "TEST_ENV", matches = "LOCAL")
                void create_new_study(){
                    Study actual = new Study(10);
                    assertThat(actual.getLimit()).isGreaterThan(0);
                }
                ```
              
#### 6) 태깅과 필터링

* (1) 개요

    * 테스트 그룹을 만들고 원하는 테스트 그룹만 테스트를 실행할 수 있는 기능

* (2) @Tag

    * `@Tag`는 테스트 메소드에 태그를 추가 할 수 있다.

    * 하나의 테스트 메소드에 여러 태그를 사용 할 수 있다.
    
* (3) Intellij에서 특정 태그로 테스트를 필터링 하는 방법

    * ① @Tag 애노테이션을 이용하여 테스트 코드를 변경한다.
    
        ```java
        class StudyTest {
        
            @Test
            @DisplayName("스터디 만들기 fast")
            @Tag("fast")
            void create_new_study(){
                Study actual = new Study(10);
                assertThat(actual.getLimit()).isGreaterThan(0);
            }
        
            @Test
            @DisplayName("스터디 만들기 slow")
            @Tag("slow")
            void create_new_study_again(){
                System.out.println("create1");
            }
        
        }
        ```
      
    * ② 인텔리제이 우측 상단 메뉴에서 `[Edit Configurations...]`를 클릭한다.

    * ③ `Test kind`를 "Tags"로, `Tag expression`을 "fast"로 변경한다.
    
    * ④ `[apply]` - `[OK]` 버튼을 클릭한다.
    
        * 그러면 fast라는 `@Tag("fast")`가 붙어 있는 테스트 메소드만 실행 된다.
           
* (4) 메이븐에서 테스트를 필터링 하는 방법

    ```html
    <profiles>
        <profile>
            <id>ci</id>
            <build>
                <plugins>
                    <plugin>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <configuration>
                            <groups>fast | slow</groups>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
    ```
        
#### 7) 커스텀 태그

* (1) 개요

    * JUnit 5 애노테이션을 조합하여 커스텀 태그를 만들 수 있다.

* (2) 실습

    * ① 다음과 같이 FastTest 애노테이션을 작성한다.

    * ② 테스트 코드를 다음과 같이 변경 할 수 있다.
    
        ```java
        class StudyTest {
        
            @FastTest
            @DisplayName("스터디 만들기 fast")
            void create_new_study(){
                Study actual = new Study(10);
                assertThat(actual.getLimit()).isGreaterThan(0);
            }
        
            @SlowTest
            @DisplayName("스터디 만들기 slow")
            void create_new_study_again(){
                System.out.println("create1");
            }
        
        }
        ```
      
        * 즉, @Test, @Tag("fast")를 제거하고 @FastTest 애노테이션을 적용한다.