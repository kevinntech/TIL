# 백기선님의 스프링 부트 개념과 활용
> 아래 내용은 [스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8 "스프링 부트 개념과 활용") 강좌를 정리한 내용 입니다.

## 1. 스프링 부트 시작하기

#### 1) 스프링 부트 소개

* (1) 스프링 부트

    * Spring Boot는 제품 수준의 Spring 기반 애플리케이션을 쉽게 만들 수 있도록 도와준다.

    * 가장 널리 사용되는 설정을 기본적으로 제공함으로써 개발자가 일일이 직접 설정하지 않도록 한다. (opinionated view)

* (2) 스프링 부트 목적

    * 스프링 개발 시 더 빠르고 폭넓은 사용성을 제공한다.

    * 개발자가 일일이 설정하지 않아도 이미 컨벤션으로 정해진 설정을 제공하며 요구사항에 맞게 설정을 쉽고 빠르게 변경 할 수 있다.

    * 비즈니스 로직을 구현하는데 필요한 기능 뿐만 아니라, 다양한 기능을 제공한다.

    * XML 설정을 더 이상 사용하지 않으며, code generation도 하지 않는다.

* (3) 시스템 요구사항

    * 자바 8 버전 이상 필요하다.
    
#### 2) 스프링 부트 시작하기

* `인텔리제이` 또는 `start.spring.io`에서 프로젝트 생성하기
    * Spring Web 의존성을 추가

#### 3) 스프링 부트 프로젝트 구조
        
* 메이븐(Maven) 기본 프로젝트 구조와 동일하다.

    * `소스코드 (src\main\java)` : 자바 코드가 저장된다.

    * `소스 리소스 (src\main\resources)`
    
        * 자바 코드를 제외한 모든 파일이 저장된다. 
        
        * 즉, resources가 클래스패스 루트이다.
        
    * `테스트 코드 (src∖test∖java)`
    
    * `테스트 리소스 (src∖test∖resources)`

* 메인 애플리케이션의 위치는 다음과 같이 지정 해야 한다.

    * `src\main\java`에 디폴트 패키지(me.kevinntech)를 만들고 여기에 `@SpringBootApplication`이 붙어있는 클래스가 위치 하도록 해야한다.
    
    * 그 이유는 `src\main\java`에 메인 애플리케이션이 위치하면 모든 패키지에 대해서 컴포넌트 스캔을 하기 때문이다.
    
## 2. 스프링 부트 원리

#### 1) 의존성 관리 이해

* 의존성 관리

    * 앞서 생성한 프로젝트의 `pom.xml`을 보면 의존성을 정의할 때, 라이브러리 버전을 명시하지 않았지만 적절한 버전을 가져오고 있다.

    * 그 이유는 스프링 부트에서 제공하는 `의존성 관리(dependency management) 기능` 때문이다.
   
    * `spring-boot-starter`는 프로젝트에 설정 해야되는 많은 의존성들을 미리 정의하여 제공한다.
   
    * `spring-boot-dependencies`에서 의존성 버전을 관리하고 있다.
   
    * 스프링 부트가 제공하는 의존성 관리 기능은 개발자가 직접 관리해야하는 의존성의 개수를 줄여주는 장점이 있다.
    
    * (스프링 버전을 올렸을 때, 다른 라이브러리와의 호환성을 신경쓰지 않아도 됨)
    
#### 2) 의존성 관리 응용

* (1) 버전 관리를 해주는 의존성을 추가하는 방법

    ```html
        <dependencies>
  
            ...
    
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-jpa</artifactId>
            </dependency>
            
            ...
  
        </dependencies>
    ```

* (2) 버전 관리를 해주지 않는 의존성을 추가하는 방법

    * ① `Maven Repository`에서 사용 하고자 하는 의존성을 검색한다.

    * ② 의존성을 추가 해주는 스크립트를 복사한다.

    ```html
    <dependency>
        <groupId>org.modelmapper</groupId>
        <artifactId>modelmapper</artifactId>
        <version>2.1.0</version>
    </dependency>
    ```
  
    * ③ 자신의 프로젝트에 있는 `pom.xml`에 붙여넣는다.

* (3) 기존의 의존성 버전을 변경하는 방법

    * ① 기존 스프링 버전을 확인한다.
    
        * `spring-boot-dependencies`의 <properties>에는 의존성 버전을 일괄적으로 관리하고 있다. 

        * spring.version을 찾아보면 스프링의 버전을 확인 할 수 있는데, 다음과 같은 코드를 복사하자.

        * `<spring.version>5.1.17.RELEASE</spring.version>`

    * ② 원하는 스프링 버전으로 변경한다.
    
        * pom.xml에서 <properties></properties> 안에 복사한 코드를 붙여넣고 스프링 버전을 변경한다.

            ```html
            <properties>
                <java.version>1.8</java.version>
                <spring.version>5.1.16.RELEASE</spring.version>
            </properties>
            ```
          
    * ③ 스프링 버전이 변경된 것을 확인 할 수 있다.

#### 3) 자동 설정 이해

* (1) 자동 설정 개요

    * `@SpringBootApplication`은 다음 3개의 애노테이션을 합친 것이다.
    
        * `@SpringBootConfiguration`, `@ComponentScan`, `@EnableAutoConfiguration`
      
    * 스프링 부트 애플리케이션은 2 단계에 걸쳐 빈(Bean)을 등록한다.
    
        * `@ComponentScan`으로 빈을 읽어서 등록한 다음, `@EnableAutoConfiguration`으로 추가적인 Bean을 읽어서 등록한다.
        
* (2) 첫 번째 단계 (@ComponentScan)

    * @ComponentScan은 @Component 애노테이션이 붙어있는 클래스들을 스캔하여 스프링 IoC 컨테이너에 빈으로 등록한다.

    * 다음 애노테이션이 붙은 클래스는 컴포넌트 스캔 대상이다.

        * @Component, @Configuration , @Repository , @Service , @Controller , @RestController

* (3) 두 번째 단계 (@EnableAutoConfiguration)

    * `spring-boot-autoconfigure` 라는 프로젝트에 `spring.factories`라는 스프링 메타 파일이 있다.
    
    * 해당 파일에 설정된 `EnableAutoConfiguration` 키 값과 일치하는 클래스들을 빈으로 등록한다.  

    * 그런데 모두 빈으로 등록되는 것은 아니며 조건에 따라 등록된다. 
    
        * 예를 들어, `WebMvcAutoConfiguration` 클래스를 살펴보면 @ConditionalXXX 형식의 애노테이션들이 많이 사용되며 해당 애노테이션은 조건에 따라 빈 등록 여부를 결정할 때 사용된다. 

#### 4) 자동 설정 만들기 1부: Starter와 Autoconfigure

* (1) 자동 설정 구현

    * `Xxx-Spring-Boot-Autoconfigure` 모듈 : 자동 설정
    
    * `Xxx-Spring-Boot-Starter` 모듈 : 필요한 의존성 정의
    
        * 그냥 하나로 만들고 싶다면? `Xxx-Spring-Boot-Starter`
  
* (2) `Xxx-Spring-Boot-Starter`만 만드는 구현 방법

    * ① 먼저, maven 프로젝트를 xxx-spring-boot-starter 형식으로 생성한다.

    * ② pom.xml에 의존성을 추가한다.

        ```html
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-autoconfigure</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-autoconfigure-processor</artifactId>
                <optional>true</optional>
            </dependency>
        </dependencies>
        
        <dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-dependencies</artifactId>
                    <version>2.2.11.RELEASE</version>
                    <type>pom</type>
                    <scope>import</scope>
                </dependency>
            </dependencies>
        </dependencyManagement>
        ```
      
    * ③ 패키지(me.kevinntech)를 만든 다음, 자동 설정의 대상이 되는 클래스를 작성한다.
    
        ```java
        public class Holoman {
            
            String name;
            
            int howLong;
        
            public String getName() {
                return name;
            }
        
            public void setName(String name) {
                this.name = name;
            }
        
            public int getHowLong() {
                return howLong;
            }
        
            public void setHowLong(int howLong) {
                this.howLong = howLong;
            }
        
            @Override
            public String toString() {
                return "Holoman{" +
                        "name='" + name + '\'' +
                        ", howLong=" + howLong +
                        '}';
            }
        }
        ```
      
    * ④ 자동 설정 파일(@Configuration)을 만든다.
   
        ```java
        @Configuration
        public class HolomanConfiguration {
        
            @Bean
            public Holoman holoman(){
                Holoman holoman = new Holoman();
                holoman.setHowLong(5);
                holoman.setName("kevinntech");
                return holoman;
            }
            
        }
        ```   
      
    * ⑤ `src/main/resource/META-INF`에 `spring.factories` 파일을 만들고 여기에 자동 설정 파일을 추가한다.
    
        ```java
        org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
          me.kevinntech.HolomanConfiguration
        ```
      
    * ⑥ 터미널에서 `mvn install` 명령어를 실행 하거나 인텔리제이의 Maven 탭에서 install를 클릭한다.
    * 그러면 해당 프로젝트를 빌드하여 jar 파일을 생성하며 이 파일을 다른 maven 프로젝트에서도 사용 할 수 있도록 로컬 maven 저장소에 설치한다.
    
    * ⑦ 기존 프로젝트(spring-boot-second)의 pom.xml에 아래 코드를 추가한다.
    
        ```html
        <dependency>
        	<groupId>org.example</groupId>
        	<artifactId>kevin-spring-boot-starter</artifactId>
        	<version>1.0-SNAPSHOT</version>
        </dependency>
        ```    
      
    * ⑧ 의존성이 들어온 것을 확인 할 수 있다.

    * ⑨ 스프링 부트 메인 애플리케이션을 다음과 같이 변경 한다.

        ```java
        @SpringBootApplication
        public class Application {
        
        	public static void main(String[] args) {
        		SpringApplication application = new SpringApplication(Application.class);
        		application.setWebApplicationType(WebApplicationType.NONE);
        		application.run(args);
        	}
        
        }
        ```
      
    * 그리고 ApplicationRunner 인터페이스를 구현한 클래스를 작성하여 자동 설정에 의해 빈을 주입 받을 수 있다는 것을 확인 할 수 있다.    
    
        ```java
        @Component
        public class HolomanRunner implements ApplicationRunner {
        
            @Autowired
            Holoman holoman;
        
            @Override
            public void run(ApplicationArguments args) throws Exception {
                System.out.println(holoman);
            }
        }
        ```
      
    * 앞서 살펴본 구현 방식은 문제점이 존재한다.

        * 스프링 부트의 메인 애플리케이션에 명시적으로 빈을 등록하면 `@ComponentScan`에 의해서 빈으로 등록 된 다음, 
        
        * `@EnableAutoConfiguration`이 자동 설정 파일을 읽어 빈으로 등록하며 이때, 명시적으로 등록한 빈을 덮어쓰게 된다는 문제점이 있다.

            ```java
            @SpringBootApplication
            public class SpringBootSecondApplication {
            
                ...
            
                @Bean
                public Holoman holoman(){ // 명시적으로 빈을 등록
                    Holoman holoman = new Holoman();
                    holoman.setName("whiteship");
                    holoman.setHowLong(60);
                    return holoman;
                }
            
            }
            ```
          
#### 5) 자동 설정 만들기 2부: @ConfigurationProperties

이전에 발생한 문제점을 해결하기 위한 방법을 아래에서 하나씩 살펴본다.

* (1) @ConditionalOnMissingBean으로 덮어쓰기 방지하기 

    * ① `kevin-spring-boot-starter` 프로젝트의 `HolomanConfiguration`를 다음과 같이 수정한다.

        ```java
        @Configuration
        public class HolomanConfiguration {
        
            @Bean
            @ConditionalOnMissingBean
            public Holoman holoman(){
                Holoman holoman = new Holoman();
                holoman.setHowLong(5);
                holoman.setName("kevinntech");
                return holoman;
            }
        
        }
        ```

        * `@ConditionalOnMissingBean`
        
            * 해당 bean이 생성되어 있지 않은 경우에만 조건을 만족한다.
            
            * @Bean과 함께 사용되어 조건을 만족하는 경우에만 빈을 생성한다.
            
        * `@ConditionalOnClass`
        
            * 해당 class가 클래스패스에 존재할 때만 조건을 만족한다. 
            
    * ② 그리고 `Maven install`을 한다.
    
    * ③ 기존 프로젝트(spring-boot-second)에서 Maven을 refresh 한다.
    
    * ④ 애플리케이션을 다시 실행하면 내가 직접 등록한 빈이 출력되는 것을 확인 할 수 있다.
    
필요한 빈(Bean)을 일일이 재정의하는 것이 아닌 값만 변경하고 싶다면 다음과 같이 할 수 있다.
    
* (2) @ConfigurationProperties

    * ① 기존 프로젝트(spring-boot-second)의 `src\main\resources`에 `application.properties` 파일을 만든다.

    * ② `key = value` 형식으로 Bean에 설정할 값을 작성한다.    

        ```java
        holoman.name = gildong
        holoman.how-long = 9
        # howLong도 가능(camel case)
        ```
      
    * ③ kevin-spring-boot-starter 프로젝트에 `@ConfigurationProperties("prefix")`를 붙인 Properties 클래스를 작성한다.

        ```java
        @ConfigurationProperties("holoman")
        public class HolomanProperties {
        
            private String name;
        
            private int howLong;
        
            public String getName() {
                return name;
            }
        
            public void setName(String name) {
                this.name = name;
            }
        
            public int getHowLong() {
                return howLong;
            }
        
            public void setHowLong(int howLong) {
                this.howLong = howLong;
            }
        }
        ```
      
    * ④ 인텔리제이 얼티메이트 버전에서는 알림이 나타나는데 해당 링크의 의존성을 pom.xml에 추가한다.

        ```html
        <dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-configuration-processor</artifactId>
        	<optional>true</optional>
        </dependency>
        ```
      
    * ⑤ Bean 설정을 담당하고 있는 HolomanConfiguration 클래스가 properties 파일을 읽어와 값을 사용할 수 있도록 @EnableConfigurationProperties 애노테이션을 붙인다.

        ```java
        @Configuration
        @EnableConfigurationProperties(HolomanProperties.class)
        public class HolomanConfiguration {
        
            @Bean
            @ConditionalOnMissingBean
            public Holoman holoman(HolomanProperties properties){
                Holoman holoman = new Holoman();
                holoman.setHowLong(properties.getHowLong());
                holoman.setName(properties.getName());
                return holoman;
            }
        
        }
        ```
      
    * ⑥ kevin-spring-boot-starter 프로젝트에서 mvn install를 하고 spring-boot-second 프로젝트에서 refresh를 한 다음, 실행한다.

#### 6) 내장 서블릿 컨테이너

* 스프링 부트는 웹 서버가 아니다.

    * 스프링 부트는 내장 서블릿 컨테이너를 쉽게 사용 할 수 있도록 해주는 도구(Tool)일 뿐, 스프링 부트 자체가 웹 서버는 아니다.
    
    * 자바 코드로 톰캣을 만들 수 있다.
    
        ```java
        @SpringBootApplication
        public class Application {
        
        	public static void main(String[] args) throws LifecycleException {
        		Tomcat tomcat = new Tomcat(); // 톰캣 객체 생성
        		tomcat.setPort(8080); // 포트 설정
        		tomcat.getConnector(); // 해당 코드가 추가되어야 정상적으로 동작한다.
        
        		Context context = tomcat.addContext("/", "/"); // 톰캣에 컨텍스트 추가
        
                // 서블릿 만들기
        		HttpServlet servlet = new HttpServlet() {
        			@Override
        			protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        				PrintWriter writer = resp.getWriter();
        				writer.println("<html><head><title>");
        				writer.println("Hey, Tomcat");
        				writer.println("</title></head>");
        				writer.println("<body><h1>Hello Tomcat</h1></body>");
        				writer.println("</html>");
        			}
        		};
        
                // 톰캣에 서블릿 추가              
        		String servletName = "helloServlet";
        		tomcat.addServlet("/", servletName, servlet);
      
                // 컨텍스트에 서블릿 맵핑 
        		context.addServletMappingDecoded("/hello", servletName);

                // 톰캣 실행 및 대기             
        		tomcat.start();
        		tomcat.getServer().await();
        	}
        
        }
      
        ```
      
* 앞서 살펴본 모든 과정을 보다 상세하게 설정하고 실행 해주는 것이 바로 스프링 부트의 자동 설정이다.

    * `spring-boot-autoconfigure`에서 `spring.factories`를 열어보면 자동 설정 파일들이 있다.
    
    * `ServletWebServerFactoryAutoConfiguration`는 서블릿 웹 서버를 생성하는 자동 설정 파일이다.
    
        * `TomcatServletWebServerFactoryCustomizer`는 웹 서버를 커스터마이징한다.
        
    * `DispatcherServletAutoConfiguration`
    
        * 서블릿을 만들고 서블릿 컨테이너에 등록한다.
        
    * `DispatcherServlet`는 `HttpServlet`를 상속해서 만든 Spring MVC의 핵심 클래스이다
        
* 즉, 스프링 부트의 자동 설정이 내장 서블릿 컨테이너(Tomcat, Jetty, Undertow)를 설정하고 실행 시킨다.

#### 7) 내장 서블릿 컨테이너 응용 1부 : 컨테이너와 서버 포트

* 내장 웹 서버(서블릿 컨테이너) 변경

    * 서블릿 기반의 웹 애플리케이션을 개발할 때, 기본적으로 톰캣을 사용하게 된다. 이를 변경해보자.
    
        * ① `spring-boot-starter-web`는 다음과 같이 `spring-boot-starter-tomcat`을 포함하고 있다.
        
        * ② `pom.xml`에 톰캣의 의존성을 제외하기 위해 `<exclusions>` 엘리먼트를 추가하고 jetty 의존성을 추가한다.
            
            ```html
            <dependencies>
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-web</artifactId>
                    <exclusions>
                        <exclusion>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-starter-tomcat</artifactId>
                        </exclusion>
                    </exclusions>
                </dependency>
            
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-jetty</artifactId>
                </dependency>
                
                ...
          
            </dependencies>
            ```
          
        * ③ 톰캣 의존성은 사라지고 jetty 의존성이 추가 되었음을 확인 할 수 있으며 실행 또한 jetty로 실행된다. 

* 웹 서버 사용 하지 않기

    * 어떤 웹 서버도 사용하고 싶지 않으면 `application.properties`에 `spring.main.web-application-type=none`를 추가하면 된다.
    
* 포트 변경하기

    * ① 원하는 포트로 변경
    
        ```java
        #application.properties
        server.port=7070
        ```
      
    * ② 랜덤 포트로 사용
    
        ```java
        #application.properties      
        server.port=0
        ```
      
    * ③ 런타임 시 HTTP 포트 검색
    
        * `ApplicationListener<ServletWebServerInitializedEvent>`를 이용한다.
    
            ```java
            @Component
            public class PortListener implements ApplicationListener<ServletWebServerInitializedEvent> {
                // 웹 서버가 생성(초기화)되면 해당 이벤트의 리스너가 호출된다.
                @Override
                public void onApplicationEvent(ServletWebServerInitializedEvent event) {
                    // 이벤트에서 WebServerApplicationContext를 꺼낸다.
                    ServletWebServerApplicationContext applicationContext = event.getApplicationContext();
                    System.out.println(applicationContext.getWebServer().getPort()); // 웹 서버를 얻은 다음, 포트 정보를 출력
                }
            }
            ```

#### 8) 내장 웹 서버 응용 2부: HTTPS와 HTTP2

* KeyStore는 암호화 / 복호화 및 디지털 서명에 사용되는 키(Key)와 인증서(Certificate)를 추상화한 Java의 인터페이스이다.

* keytool은 KeyStore 기반으로 인증서와 key를 관리할 수 있는 명령어 방식의 유틸리티로 JDK에 포함되어 있다. 

* (1) HTTPS 설정하기

    * 인텔리제이의 터미널에서 다음과 같은 keytool 명령어로 keystore를 만든다.

        ```
        keytool -genkey -alias spring -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 4000
        
        키 저장소 비밀번호 입력: 123456
        새 비밀번호 다시 입력: 123456
        이름과 성을 입력하십시오.
          [Unknown]:  KEVIN KIM
        조직 단위 이름을 입력하십시오.
          [Unknown]:  KEVINNTECH
        조직 이름을 입력하십시오.
          [Unknown]:  CEO
        구/군/시 이름을 입력하십시오?
          [Unknown]:  Seoul
        시/도 이름을 입력하십시오.
          [Unknown]:  Seoul
        이 조직의 두 자리 국가 코드를 입력하십시오.
          [Unknown]:  KO
        CN=KEVIN KIM, OU=KEVINNTECH, O=CEO, L=Seoul, ST=Seoul, C=KO이(가) 맞습니까?
          [아니오]:  y
        ```
      
    * 그러면 다음과 같이 keystore가 생성된 것을 확인 할 수 있다.
    
        ![image 1](images/img1.png)
        
    * 그리고 `application.properties`를 다음과 같이 수정한 다음, 스프링 부트 애플리케이션을 실행한다.
    
        ```
        # 만약 keystore가 애플리케이션 루트가 아닌 resources에 있었다면 classpath:를 붙여주면 된다.
        server.ssl.key-store = keystore.p12
        server.ssl.key-store-type = PKCS12
        server.ssl.key-store-password = 123456
        server.ssl.keyAlias = spring
        ```
      
    * 이렇게 지정하면 스프링 부트에서 톰캣이 사용하는 커넥터는 하나만 등록되는데 해당 커넥터에 SSL이 적용된다.
    
    * 즉, 앞으로는 모든 요청에 https를 붙여야 한다. (http로 접근하면 Bad Request가 결과로 나타난다.)
    
* (2) HTTP 커넥터를 코딩으로 설정하기

    * https를 적용하면 http 요청을 받을 수 있는 커넥터가 없다. 

    * 그래서 http와 https를 모두 사용하고 싶다면 멀티 커넥터를 설정해야 한다.

    ```java
    @SpringBootApplication
    @RestController
    public class SpringBootWebserverApplication {
    
        // GET 요청 시 처리
        @GetMapping("/hello")
        public String hello() {
            return "Hello Spring";
        }
    
        public static void main(String[] args) {
            SpringApplication.run(SpringBootWebserverApplication.class, args);
        }
    
        // 멀티 커넥터 설정 
        @Bean
        public ServletWebServerFactory serverFactory(){
            TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
            tomcat.addAdditionalTomcatConnectors(createStandardConntor()); // 커넥터 추가
            return tomcat;
        }
    
        private Connector createStandardConntor() {
            Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
            connector.setPort(8080);
            return connector; // 커넥터를 리턴
        }
    
    }
    ```
  
    * http는 8080, https는 8443으로 포트 번호를 설정한다.
    
    ```
    server.ssl.key-store = keystore.p12
    server.ssl.key-store-type = PKCS12
    server.ssl.key-store-password = 123456
    server.ssl.keyAlias = spring
    server.port = 8443
    ```
  
    * 터미널에서 다음과 같은 요청으로 확인 할 수 있다.

    ```
    curl -I -k --http2 http://localhost:8080/hello
    curl -I -k --http2 https://localhost:8443/hello
    ```
  
* (3) HTTP2 설정

    * HTTP2를 활성화 하려면 `application.properties`에 `server.http2.enabled = true`를 추가한다.
    
    * 그리고 웹 서버 마다 제약 사항이 다른데, Undertow의 경우, HTTPS만 적용 되어 있다면 추가 설정이 필요 없다.

    * 하지만 톰캣의 경우, Tomcat 9.0.x 과 JDK 9 이상의 버전을 사용하지 않으면 설정이 매우 복잡해진다. 
    
    * 그러므로 Undertow를 사용하기 위해 다음과 같이 pom.xml을 설정한다.
    
    ```html
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-web</artifactId>
    	<exclusions>
    		<exclusion>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-tomcat</artifactId>
    		</exclusion>
    	</exclusions>
    </dependency>
    
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-undertow</artifactId>
    </dependency>
    ```
  
    * http2로 요청을 보내면 http2를 사용하고 있는 것을 확인 할 수 있다.
        
        ```html
        curl -I -k --http2 https://localhost:8443/hello
        ```
      
    * 명심해야 될 것은 HTTP2를 사용하려면 SSL이 적용되어 있어야 한다.
    
#### 9) 톰캣 HTTP2 설정

* `pom.xml`에서 Java와 Tomcat의 버전을 변경한다.

    * 스프링 부트 버전에 따라 아래 설정은 필요 없을 수도 있다. 

    ```html
    <properties>
        <java.version>11</java.version>
        <tomcat.version>9.0.10</tomcat.version>
    </properties>
    ```
        
    * 만약, 기존에 Java 9 이전 버전을 사용 하고 있었다면 JDK 버전을 변경해야 한다.
    * 그래서 인텔리제이의 [Project Structure]-[Project]에서 Project SDK 그리고 [Project Structure] - [Modules] - [Dependencies]에서 Module SDK를 변경한다.

* http2로 요청을 보내면 정상적인 응답을 확인 할 수 있다.
    
    ```html
    curl -I -k --http2 https://localhost:8443/hello
    ```
  
#### 10) 독립적으로 실행 가능한 JAR 

* (1) JAR 만들기

    * 스프링 부트 메인 애플리케이션을 다음과 같이 작성한다.
    
        ```java
        @SpringBootApplication
        @RestController
        public class SpringinitApplication {
        
        	@GetMapping("/")
        	public String hello(){
        		return "Hello Spring";
        	}
        
        	public static void main(String[] args) {
        		SpringApplication.run(SpringinitApplication.class, args);
        	}
        
        }
        ```
      
    * 인텔리제이 터미널에서 `mvn clean`를 이용하여 maven으로 빌드 시 생성 되었던 target 디렉토리의 내용을 모두 삭제한다.
    
    * `mvn package`를 이용하여 컴파일된 결과물을 패키지 파일로 생성한다. (JAR, WAR ...)
    
        * 위의 과정을 `mvn clean package`로 한번에 처리 할 수도 있다.

        * 그리고 `mvn package -DskipTests`와 같은 방식으로 테스트를 생략하고 패키징을 할 수도 있다.

    * 그리고 다음 명령어로 target 디렉토리로 이동한 다음, JAR 파일이 생성 되었는지 확인한다.
    
        ```java
        cd target
        ls
        ```
      
        ![image 2](images/img2.png)

* (2) JAR 실행

    * JAR 파일 하나를 실행하면 스프링 부트 애플리케이션이 동작한다.
    
    * `java -jar springinit-0.0.1-SNAPSHOT.jar` 
    
* (3) JAR 파일 하나로 애플리케이션이 실행되는 이유는 무엇일까?

    * 하나의 JAR 파일 안에 프로젝트에서 사용되는 JAVA 클래스, 리소스, 외부 라이브러리 파일 등이 포함 되어 있기 때문이다.
    
    * JAVA에는 내장 JAR를 읽을 수 있는 표준 방법이 없다.
    
    * 하지만 스프링 부트는 내장 JAR 파일을 읽을 수 있는 로더가 있으며 실행 할 수 있다.
    
    * 독립적으로 실행 가능한 JAR 관련 용어
    
        * 내장 JAR : JAR 파일 안에 있는 JAR 파일을 말한다.
        
        * `spring-boot-maven-plugin` : 패키징을 처리한다.
        
        * `org.springframework.boot.loader.jar.JarFile` : 내장 JAR를 읽는다.
        
        * `org.springframework.boot.loader.Launcher` : JAR 파일을 실행한다.

* 스프링 부트의 주요 목표 중 하나가 `독립적으로 실행 가능한 애플리케이션(stand-alone)`을 만들 수 있도록 하는 것이다.
    
## 3. 스프링 부트 활용

#### 1) SpringApplication : 1부 - `핵심 기능`

* (1) SpringApplication 실행 방법

    * 이전까지 실행 했던 방법은 다음과 같다.
    
        ```java
        @SpringBootApplication
        public class SpringinitApplication {
        
        	public static void main(String[] args) {
        		SpringApplication.run(SpringinitApplication.class, args);
        	}
        
        }
        ```
      
    * 하지만 이렇게 사용하게 되면 스프링 애플리케이션이 제공하는 다양한 커스터마이징 기능을 사용하기가 어렵다.
    
    * 그래서 SpringApplication 인스턴스를 만든 다음, run()를 호출하는 방법을 사용한다.
          
        ```java
        @SpringBootApplication
        public class SpringinitApplication {
        
        	public static void main(String[] args) {
        		SpringApplication app = new SpringApplication(SpringinitApplication.class);
        		app.run(args);
        	}
        
        }
        ```
      
    * 그리고 SpringApplicationBuilder로 애플리케이션을 실행하고 커스터마이징 기능을 사용하는 것도 가능하다.
          
        ```java
        @SpringBootApplication
        public class SpringinitApplication {
        
        	public static void main(String[] args) {
        		SpringApplication app = new SpringApplication(SpringinitApplication.class);
        		app.run(args);
        	}
        
        }
        ```

* (2) 기본 로그 레벨 INFO

    * 어떠한 옵션도 변경하지 않고 실행하면 기본 로그 레벨은 INFO다.
    
    * 로그 관련 설정 - 실습

        * ① Edit Configurations... 를 클릭한다.
        
            ![image 3](images/img3.png)
            
        * ② `VM option`에 -Ddebug를 작성하거나 `Program arguments`에 —debug를 작성한 다음, 실행하면 디버그 모드로 애플리케이션이 동작되며 애플리케이션 로그도 디버그 레벨까지 출력한다.

        * ③ 다음 그림처럼 디버그 레벨로 출력된 로그를 확인 할 수 있다.

            ![image 4](images/img4.png)

        * 디버그 레벨로 출력된 로그를 보면 어떠한 자동 설정이 적용 되었는지, 적용되지 않았다면 그 이유를 확인 할 수 있다.

* (3) FailureAnalyzer

    * FailureAnalyzer는 애플리케이션 에러가 발생 했을 때 에러 메시지를 좀 더 예쁘게 출력 해주는 기능이다.

    * 스프링 부트 애플리케이션은 기본적으로 여러 가지 FailureAnalyzer들이 등록 되어 있다.

* (4) 배너

    * 스프링 애플리케이션 실행 시 처음에 나타나는 로고를 "배너"라고 한다.
    
        ![image 5](images/img5.png)
        
    * 배너를 변경하는 방법
    
        * `src/main/resources`에 배너 파일(banner.txt)를 만든 다음, 배너로 사용 할 텍스트를 입력한다.
        
        * banner는 gif, jpg, png로 만들 수도 있다.

    * 여러가지 변수를 사용 할 수 있다.
    
        * ${spring-boot.version}를 사용하면 스프링 부트 버전이 출력된다.

        * MANIFEST 파일이 존재해야 스프링 부트 버전이 출력 된다.

            * JAR 파일로 패키징할 때, MANIFEST 파일도 만들어 주기 때문에 패키징을 한 다음, 해당 JAR 파일을 실행하면 된다.

    * 배너 파일의 위치를 변경하는 방법

        * `application.properties`에서 `spring.banner.location`를 지정하면 된다.
        
    * 배너가 나타나지 않도록 하는 방법
    
        ```java
        @SpringBootApplication
        public class SpringinitApplication {
        
        	public static void main(String[] args) {
        		SpringApplication app = new SpringApplication(SpringinitApplication.class);
        		app.setBannerMode(Banner.Mode.OFF);
        		app.run(args);
        	}
        
        }
        ```

    * 배너를 코딩으로 구현하는 방법
    
        * Banner 클래스를 구현하고 `SpringApplication.setBanner()`로 설정하면 된다.
    
        ```java
        @SpringBootApplication
        public class SpringinitApplication {
        
        	public static void main(String[] args) {
        		SpringApplication app = new SpringApplication(SpringinitApplication.class);
        		app.setBanner(new Banner() {
        			@Override
        			public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
        				out.println("=================");
        				out.println("KEVINNTECH");
        				out.println("=================");
        			}
        		});
        		app.run(args);
        	}
        
        }
        ```
  
#### 2) SpringApplication : 2부 - `핵심 기능`

* (1) ApplicationEvent와 Listener

    * ① 이벤트 리스너 만들기
    
        * ApplicationListener 인터페이스를 구현하는 클래스를 정의한다.
        
            ```java
            @Component
            public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {
            
                @Override
                public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
                    System.out.println("=======================");
                    System.out.println("Application is Starting");
                    System.out.println("=======================");
                }
            }
            ```
          
    * ② 이벤트 리스너를 빈(Bean)으로 등록하기
    
        * 이벤트 리스너를 빈(Bean)으로 등록하기 위해 @Component 애노테이션을 붙여준다.
        
        * 해당 이벤트가 발생하면 리스너가 실행 되는데, 리스너를 빈으로 등록하면 등록되어 있는 빈 중에 해당하는 이벤트의 리스너를 알아서 실행시켜준다.

    * ③ 여기서 발생 할 수 있는 문제점
    
        * `ApplicationContext`가 만들어진 다음, 발생하는 이벤트들은 해당 이벤트의 리스너가 빈(Bean)일 경우, 알아서 호출 할 수 있는데, 문제는 `ApplicationContext`가 만들어 지기 전에 발생하는 이벤트다.

        * `ApplicationStartingEvent`는 `ApplicationContext`가 만들어 지기 전에 발생하는 이벤트이며 해당 이벤트에 대한 리스너를 빈으로 등록해도 실행 되지 않는다.

        * 문제점을 해결하려면 `addListeners()`를 사용하면 된다. (리스너에 붙인 @Component 애노테이션은 의미가 없기 때문에 제거한다.)
    
            ```java
            @SpringBootApplication
            public class SpringinitApplication {
            
                public static void main(String[] args) {
                    SpringApplication app = new SpringApplication(SpringinitApplication.class);
                    app.addListeners(new SampleListener());
                    app.run(args);
                }
            
            }
            ```

* (2) WebApplicationType 설정

    * 웹 애플리케이션 타입은 다음 세 종류가 있다.
    
        * `WebApplicationType.SERVLET`는 Spring MVC가 있을 때 사용되는 타입이다.
    
        * `WebApplicationType.REACTIVE`는 Spring Webflux가 있을 때 사용되는 타입이다.
    
        * `WebApplicationType.NONE`는 둘 다 없을 때 사용되는 타입이다. (내장 웹 서버가 실행 되지 않음)
    
    * Spring MVC, Spring Webflux 둘 다 있다면 WebApplicationType.SERVLET으로 설정된다.
   
        * 이러한 상황에서 Spring Webflux를 선택하고 싶다면 다음과 같이 메인 애플리케이션에 명시적으로 지정하면 된다.
    
        * `app.setWebApplicationType(WebApplicationType.REACTIVE);`
        
* (3) 애플리케이션 아규먼트(Argument) 사용하기

    * 아규먼트(Argument)
    
        * JVM option는 `-D`로 시작하며 JVM 설정과 관련된 Argument이다.
    
        * 애플리케이션 아규먼트는 `--`로 시작하며 애플리케이션 설정과 관련된 Argument이다. `Program arguments`

    * ① Edit Configurations...에서 다음과 같이 설정한다.

        ![image 6](images/img6.png)

    * ② 다음과 같이 `SampleListener`를 변경한 다음, 애플리케이션을 실행한다.

        ```java
        @Component
        public class SampleListener{
        
            /* 어떤 빈(Bean)에 생성자가 1개이고 그 생성자의 파라미터가
                빈(Bean)인 경우, 스프링이 해당 빈을 자동으로 주입한다. */
            public SampleListener(ApplicationArguments arguments){
                System.out.println("foo: " + arguments.containsOption("foo"));
                System.out.println("bar: " + arguments.containsOption("bar"));
            }
        
        }
        ```
      
    * ③ 실행 결과를 보면 foo는 출력되지 않고 bar는 출력된다.
        
    * 즉, ApplicationArguments 객체에는 JVM option 값이 전달 되지 않는다는 것을 알 수 있다.
    
* (4) 애플리케이션이 실행된 이후, 뭔가 추가적으로 실행하고 싶을 때, 사용 할 수 있는 방법

    * ① ApplicationRunner `추천`

        ```java
        @Component
        public class SampleListener implements ApplicationRunner {
        
            @Override
            public void run(ApplicationArguments args) throws Exception {
                System.out.println("foo: " + args.containsOption("foo"));
                System.out.println("bar: " + args.containsOption("bar"));
            }
        
        }
        ```
      
    * ② CommandLineRunner
    
    * `@Order`로 순서를 지정 할 수도 있다. (값이 낮을수록 우선 순위가 높다)

#### 3) 외부 설정 : 1부 - `핵심 기능`

* (1) 사용 할 수 있는 외부 설정

    * `외부 설정`은 애플리케이션에서 사용하는 여러 가지 설정 값들을 정의한다.
    
    * application.properties
   
        * `application.properties`는 스프링 부트가 애플리케이션을 구동할 때, 자동으로 로딩하는 외부 설정 파일이다. (해당 이름은 컨벤션으로 정해진 것이다.)

        * ① 설정 값 정의하기
        
            * `키 = 값` 형태로 설정 값을 저장한다.

                ```java
                #application.properties
                kevin.name = kevin
                ```
              
        * ② 애플리케이션에서 설정 값 참조하기
        
            * 애플리케이션에서 설정 값을 참조하려면 @Value를 사용하면 된다.

                ```java
                @Component
                public class SampleRunner implements ApplicationRunner {
                
                    @Value("${kevin.name}")
                    private String name;
                
                    @Override
                    public void run(ApplicationArguments args) throws Exception {
                        System.out.println("================");
                        System.out.println(name);
                        System.out.println("================");
                    }
                }
                ```

    * YAML, 환경변수, 커맨드 라인 아규먼트를 사용하는 방법도 있다.
    
* (2) 프로퍼티 우선 순위

    * ① 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties

    * ② 테스트에 있는 @TestPropertySource

    * ③ @SpringBootTest 애노테이션의 properties 애트리뷰트

    * ④ 커맨드 라인 아규먼트

    * ⑤ SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로퍼티)에 들어있는 프로퍼티

    * ⑥ ServletConfig 파라미터

    * ⑦ ServletContext 파라미터

    * ⑧ java:comp/env JNDI 애트리뷰트

    * ⑨ System.getProperties() 자바 시스템 프로퍼티

    * ⑩ OS 환경 변수

    * ⑪ RandomValuePropertySource

    * ⑫ JAR 밖에 있는 특정 프로파일용 application properties

    * ⑬ JAR 안에 있는 특정 프로파일용 application properties

    * ⑭ JAR 밖에 있는 application properties

    * ⑮ JAR 안에 있는 application properties

    * ⑯ @PropertySource

    * ⑰ 기본 프로퍼티 (SpringApplication.setDefaultProperties)
    
* (3) 테스트 코드에서 외부 설정 - 문제가 발생 할 수 있는 방법

    * ① 다음과 같은 테스트 코드를 작성한다.
    
        ```java
        @RunWith(SpringRunner.class)
        @SpringBootTest
        public class SpringinitApplicationTests {
        
            @Autowired
            Environment environment; // Environment 빈을 주입 받은 다음, 설정 값을 가져 올 수 있다.
        
            @Test
            public void contextLoads() {
                assertThat(environment.getProperty("kevin.name"))
                        .isEqualTo("kevin");
            }
        
        }
        ```
      
    * ② 테스트에서만 사용되는 application.properties 파일이 필요하다 `test/resources`에 application.properties 파일을 생성한다.
    
        * 해당 경로에 resources 디렉토리가 없다면 생성한다.
        
            ![image 7](images/img7.png)
            
        * 인텔리제이에서 [File] - [Project Structure]를 클릭한 다음, 아래 그림처럼 [src]-[test]에 있는 `resources`를 선택하고 `Test Resources` 아이콘을 클릭한다. 그리고 [Apply] , [OK] 버튼을 클릭한다.
          
            ![image 8](images/img8.png)
            
        * `test/resources`에 생성한 application.properties 파일의 내용은 다음과 같이 작성한다.
        
            ```
            kevin.name=kevinntech
            ```
                
    * ③ 그리고 테스트 코드를 다음과 같이 변경한 다음, 실행한다.
    
        ```java
        @RunWith(SpringRunner.class)
                @SpringBootTest
                public class SpringinitApplicationTests {
                
                    @Autowired
                    Environment environment; // Environment 빈을 주입 받은 다음, 설정 값을 가져 올 수 있다.
                
                    @Test
                    public void contextLoads() {
                        assertThat(environment.getProperty("kevin.name"))
                                .isEqualTo("kevinntech");
                    }
                
                }
        ```

* (4) 테스트 코드에서 외부 설정 - 문제가 발생하는 이유

    * 문제가 발생하는 이유에 대해서 하나씩 살펴보자.
    
    * ① `src/main/resources`에 있는 `application.properties`를 다음과 같이 수정한다.

        ```
        kevin.name=kevin
        kevin.age=${random.int}
        ```
      
    * ② SampleRunner 코드를 수정한다. 

        ```java
        @Component
        public class SampleRunner implements ApplicationRunner {
        
            @Value("${kevin.name}")
            private String name;
        
            @Value("${kevin.age}")
            private int age;
        
            @Override
            public void run(ApplicationArguments args) throws Exception {
                System.out.println("================");
                System.out.println(name);
                System.out.println(age);
                System.out.println("================");
            }
        }
        ```
      
    * ③ `src/main`에 있는 SampleRunner를 실행하면 문제가 없지만 테스트 코드를 실행하면 에러가 발생한다.
    
        * 에러가 발생한 이유는 다음과 같다.
        
        * 테스트 코드를 실행하면 main 디렉토리를 빌드한 다음, test 디렉토리를 빌드한다.
          
        * 이 과정에서 test 디렉토리에 있는 application.properties가 main 디렉토리에 있는 것을 덮어쓰게 된다.
          
        * 그리고 test 디렉토리에 있는 application.properties 파일은 kevin.age가 정의되어 있지 않으므로 에러가 발생하게 되는 것이다.
        
        * 그래서 에러가 발생하지 않도록 하려면 test 디렉토리에도 `kevin.age = ${random.int}`를 추가해야 한다. 이러한 방식으로 관리 하기에는 힘들다.
    
* (5) 테스트 코드에서 외부 설정 - 해결방법

    * ① @SpringBootTest의 properties 속성을 이용하는 방법
    
        ```java
        @RunWith(SpringRunner.class)
        @SpringBootTest(properties = "kevin.name = kevin2")
        public class SpringinitApplicationTests {
        
        	@Autowired
        	Environment environment;
        
        	@Test
        	public void contextLoads() {
        		assertThat(environment.getProperty("kevin.name"))
        				.isEqualTo("kevin2");
        	}
        
        }
        ```
    
    * ② @TestPropertySource를 사용하여 직접 설정 값을 지정하는 방법
    
        ```java
        @RunWith(SpringRunner.class)
        @TestPropertySource(properties = "kevin.name=kevin3")
        // @TestPropertySource(properties = {"kevin.name=kevin2", "kevin.age=20"})
        @SpringBootTest(properties = "kevin.name = kevin2")
        public class SpringinitApplicationTests {
        
        	@Autowired
        	Environment environment;
        
        	@Test
        	public void contextLoads() {
        		assertThat(environment.getProperty("kevin.name"))
        				.isEqualTo("kevin3");
        	}
        
        }
        ```

    * ③ 테스트 용도의 외부 설정 파일을 다른 이름(test.properties)으로 만든 다음, @TestPropertySource로 지정하는 방법

        * 외부 설정 파일 이름이 다르기 때문에 `application.properties` 파일을 덮어 쓰지 않는다.
    
        ```java
        @RunWith(SpringRunner.class)
        @TestPropertySource(locations = "classpath:/test.properties") // 테스트에서는 클래스 패스를 기준으로 test.properties를 사용한다.
        @SpringBootTest
        public class SpringinitApplicationTests {
        
        	@Autowired
        	Environment environment; // Environment 빈을 주입 받은 다음, 설정 값을 가져 올 수 있다.
        
        	@Test
        	public void contextLoads() {
        		assertThat(environment.getProperty("kevin.name"))
        				.isEqualTo("kevinTest");
        	}
        
        }
        ```
      
        ```
        #test.properties
        kevin.name=kevin2
        ```
      
* (6) 랜덤 값 설정 및 플레이스 홀더

    ```
    kevin.name = kevin
    kevin.fullName = ${kevin.name} Kim  // 플레이스 홀더
    
    kevin.age = ${random.int} // 랜덤 값
    ```

* (7) application.properties 우선 순위

    * 우선 순위가 높은 설정 파일이 낮은 설정 파일을 덮어쓴다.
    
        ```
        ① file:./config/
        ② file:./
        ③ classpath:/config/      
        ④ classpath:/
        ```
      
#### 4) 외부 설정 : 2부 - `핵심 기능`

* (1) 외부 설정을 Bean으로 등록하기

    * 같은 key로 시작하는 외부 설정이 많은 경우, 하나의 클래스로 정의한 다음, 빈으로 등록 할 수 있다. (자동 완성, 타입 컨버전 등 가능해짐)
    
        ```
        kevin.name = kevin
        kevin.age = ${random.int(0,100)}
        kevin.fullName = ${kevin.name} Kim
        ```

    * ① 클래스를 다음과 같이 작성한다.
    
        * `@Component`는 빈으로 등록되도록 하며 `@ConfigurationProperties`는 application.properties와 바인딩 되도록 한다.

            ```java
            @Component
            @ConfigurationProperties("kevin")
            public class KevinProperties {
            
                String name;
            
                int age;
            
                String fullName;
            
                // Getter, Setter
                public String getName() {
                    return name;
                }
            
                public void setName(String name) {
                    this.name = name;
                }
            
                public int getAge() {
                    return age;
                }
            
                public void setAge(int age) {
                    this.age = age;
                }
            
                public String getFullName() {
                    return fullName;
                }
            
                public void setFullName(String fullName) {
                    this.fullName = fullName;
                }
            }
            ```
     
    * ② pom.xml에 의존성을 추가한다.
    
        ```html
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
        ```
      
    * ③ SampleRunner 코드를 다음과 같이 변경한다. 외부 설정 클래스의 Bean을 주입 받아서 사용한다.
    
        ```java
        @Component
        public class SampleRunner implements ApplicationRunner {
            
            /*  // 기존 코드
                @Value("${kevin.fullName}")
                private String name;
            
                @Value("${kevin.age}")
                private int age;    */
            
            @Autowired
            KevinProperties kevinProperties;
            
            @Override
            public void run(ApplicationArguments args) throws Exception {
                System.out.println("================");
                System.out.println(kevinProperties.getName());
                System.out.println(kevinProperties.getAge());
                System.out.println("================");
            }
        }
        ```
      
* (2) 융통성 있는 바인딩 (Relaxed Binding)

    * `application.properties`에서 사용되는 키(key)에  `_ (under score)` 또는 `- (kebab)`이 포함되어 있더라도 적절하게 camelcase로 변환하여 바인딩 된다.
  
        ```
        kevin.name = kevin
        kevin.age = ${random.int(0,100)} // ${random.int(0, 100)}와 같이 공백이 있으면 에러가 발생함
        kevin.full_name = ${kevin.name} Kim
        ```
      
* (3) 프로퍼티 타입 컨버전

    * `application.properties`에 있는 설정 값이 외부 설정 Bean 클래스와 바인딩 될 때, 해당 클래스의 필드에 적절한 타입(int, String, Duration 등)으로 타입 컨버전이 된다.

        ```java
        @Component
        @ConfigurationProperties("kevin")
        public class KevinProperties {
        
            String name;
        
            int age;
        
            String fullName;
        
            @DurationUnit(ChronoUnit.SECONDS)
            private Duration sessionTimeout = Duration.ofSeconds(30);
        
            // getter, setter가 있다고 가정
        }
        ```
      
    * application.properties에서 외부 설정을 지정할 때 s, m, h, d 와 같은 suffix를 사용하면 @DurationUnit을 사용하지 않더라도 Duration으로 타입 컨버전이 된다.

        ```
        kevin.name = kevin
        kevin.age = ${random.int(0,100)} // ${random.int(0, 100)}와 같이 공백이 있으면 에러가 발생함
        kevin.full_name = ${kevin.name} Kim
        kevin.sessionTimeout=25s
        ```

* (4) 프로퍼티 값 검증

    * @Validated 와 JSR-303 Validation API의 구현체 (@NotNull ...)를 사용하여 프로퍼티 값을 검증한다.
    
        ```java
        @Component
        @ConfigurationProperties("kevin")
        @Validated
        public class KevinProperties {
        
            @NotEmpty
            String name;
        
            ...
        }
        ```

* (5) @Value과 @ConfigurationProperties 비교

    * @Value 보다는 @ConfigurationProperties를 사용하는 것이 좋다.
      
    * 그 이유는 @Value는 SpEL을 사용 할 수 있지만 위에서 언급된 기능들은 전부 사용 할 수 없기 때문이다.   

#### 5) 프로파일 - `핵심 기능`

* (1) Profile 이란

    * Profile은 환경(테스트, 운영 등)마다 다른 설정을 할 수 있도록 하는 기능이다.

    * 예를 들어, 테스트와 운영 환경에서 서로 다른 빈이 사용 되도록 할 수 있다.

* (2) Profile 설정

    * ① Profile 정의
    
        * 다음과 같이 `@Profile`과 `@Configuration`를 이용하여 두 개의 클래스를 정의한다.
        
            ```java
            /* 프로파일이 prod일 때만 사용되는 설정 파일 */
            @Profile("prod")
            @Configuration
            public class BaseConfiguration {
            
                @Bean
                public String hello(){
                    return "hello";
                }
            
            }
            ```
          
            ```java
            @Profile("test")
            @Configuration
            public class TestConfiguration {
            
                @Bean
                public String hello(){
                    return "hello test";
                }
            
            }
            ```

    * ② Profile 사용

        * SampleRunner에서 앞서 정의한 빈(Bean)을 사용하려고 하면 해당 빈(Bean)을 찾을 수 없다는 에러가 발생한다.
        
        * 그 이유는 프로파일을 정의만 하고 활성화 하지 않았기 때문이다.

            ```java
            @Component
            public class SampleRunner implements ApplicationRunner {
            
                @Autowired
                private String hello;
            
                @Override
                public void run(ApplicationArguments args) throws Exception {
                    System.out.println("================");
                    System.out.println(hello);
                    System.out.println("================");
                }
            }
            ```
          
    * ③ Profile 활성화

        * `application.properties`에서 `spring.profiles.active`로 활성화 할 프로파일을 지정 할 수 있다.
        
* (3) Profile용 프로퍼티 파일 만들기

    * ① Profile용 프로퍼티 파일 만들기
    
        * Profile용 프로퍼티 파일을 `application-{profile}.properties`로 만든다.
        
            ![image 9](images/img9.png)
            
        * 그리고 각 파일에 맞는 설정을 지정한다.
        
            ```
            # application-prod.properties
            kevin.name = kevin prod
            
            # application-test.properties
            kevin.name = kevin test
            ```
          
    * ② Profile 사용
    
        * SampleRunner의 코드를 변경한다.
        
            ```java
            @Component
            public class SampleRunner implements ApplicationRunner {
            
                @Autowired
                private String hello;
            
                @Autowired
                private KevinProperties kevinProperties;
            
                @Override
                public void run(ApplicationArguments args) throws Exception {
                    System.out.println("================");
                    System.out.println(hello);
                    System.out.println(kevinProperties.getName());
                    System.out.println("================");
                }
            }
            ```
          
        * Profile용 프로퍼티 파일이 기본적인 `application.properties` 보다 우선순위가 더 높다.
          
        * 그렇기 때문에 `application-prod.properties`의 값(kevin.name)이 `application.properties`에 있는 값(kevin.name)을 오버라이딩 한다.        

* (4) 다른 프로파일을 포함하기

    * 프로파일용 프로퍼티 파일을 `spring.profiles.include`로 포함 할 수 있다.  
    
    * ① Profile용 프로퍼티 파일 만들기

        ```
        # application-proddb.properties
        kevin.full-name = dbdbdb
        ```    

    * ② 프로파일 용 프로퍼티 파일을 포함한다.
    
        ```
        /# application-prod.properties
        kevin.name = kevin prod
        spring.profiles.include=proddb
        ```
      
#### 6) 로깅 1부 : 스프링 부트 기본 로거 설정 - `핵심 기능`

* (1) 로깅 퍼사드 VS 로거

    * 스프링 부트는 기본적으로 Commons Logging을 사용하며 결국에는 SLF4j를 사용하게 된다.

    * Commons Logging과 SLF4j는 실제 로깅을 하는 것이 아닌 로거(Logger) API 들을 추상화 해놓은 인터페이스들이다. 그리고 "로깅 퍼사드" 라고도 부른다.

    * 로깅 퍼사드를 사용하면 원하는 로거로 바꿔서 사용 할 수 있다.

    * 로거에는 JUL, Log4J2 , LogBack이 있다.
    
* (2) 스프링 5에 로거 관련 변경 사항

    * 스프링 5 부터 Commons Logging 를 pom.xml에서 exclusion 하지 않더라도 컴파일 시점에 SLF4j 또는 Log4j2로 변경 할 수 있는 기능을 가진 Spring-JCL이라는 모듈을 만들었다.
    
    * `Commons Logging` → `SLF4j` 또는 `Log4j2` → `Logback`
    
        * Commons Logging을 SLF4j 또는 Log4j2로 변환해주며 SLF4j는 SLF4j의 구현체인 Logback으로 보낸다.
      
        * 그리고 Log4j2를 사용하더라도 SLF4j로 보내기 때문에 결국에는 Logback으로 로그를 남기게 된다.
        
    * 즉, 스프링 부트는 최종적으로 `Logback`을 사용하게 된다.
    
* (3) 스프링 부트 로깅

    * 로깅 기본 포맷은 다음과 같다.
     
        ![image 10](images/img10.png)
        
    * 더 많은 내용의 로그를 출력하고 싶다면 [Edit Configurations...]에서 `Program arguments`에 --debug를 지정하거나 `VM options`에 -Ddebug를 지정한다.  

        * ① --debug : 일부 핵심 라이브러리(내장 컨테이너, 하이버네이트, 스프링 부트)만 디버깅 모드로 출력한다.
        
        * ② --trace : 전부 다 디버깅 모드로 출력한다.
 
    * 로그를 컬러로 출력한다.
    
        * `application.properties`에서 `spring.output.ansi.enabled = always`를 지정하면 로그가 컬러로 출력된다.
 
    * 로그를 파일로 출력한다.
    
        * `logging.file` : 지정한 로그 파일로 기록한다.
        
        * `logging.path` : 지정한 경로에 spring.log 파일로 기록한다. 
        
        * 예를 들어, `logging.path=logs`를 지정하면 logs 디렉토리 안에 `spring.log` 파일로 기록된다.
        
    * 로그 레벨 조정
    
        * `logging.level.패키지경로 = 로그 레벨`과 같이 패키지 마다 로그 레벨을 지정 할 수 있다.
        
        * `logging.level.me.kevinntech = DEBUG`로 지정하면 로그 레벨을 디버그로 지정한다.
        
    * 로거 만들기
    
        * `LoggerFactory` 클래스의 `getLogger()` 메소드로 `Logger` 객체를 생성한다. (SLF4j 사용)

        * 로그 레벨 마다 로그를 출력하는 메소드를 제공한다.

            * logger.trace()
              
            * logger.debug()
              
            * logger.info()
              
            * logger.warn()
              
            * logger.error()
            
            ```java
            @Component
            public class SampleRunner implements ApplicationRunner {
            
                private Logger logger = LoggerFactory.getLogger(SampleRunner.class);
            
                @Autowired
                private String hello;
            
                @Autowired
                private KevinProperties kevinProperties;
            
                @Override
                public void run(ApplicationArguments args) throws Exception {
                    logger.info("=====================");
                    logger.info(hello);
                    logger.info(kevinProperties.getName());
                    logger.info(kevinProperties.getFullName());
                    logger.info("=====================");
                }
            }
            ```
          
#### 7) 로깅 2부 : 커스터마이징 - `핵심 기능`

* (1) 실습 전에 환경설정

    * ① 다음과 같이 info에서 debug 레벨로 로그를 출력 하도록 변경 하였다.

        ```java
        @Component
        public class SampleRunner implements ApplicationRunner {
        
            private Logger logger = LoggerFactory.getLogger(SampleRunner.class);
        
            @Autowired
            private String hello;
        
            @Autowired
            private KevinProperties kevinProperties;
        
            @Override
            public void run(ApplicationArguments args) throws Exception {
                logger.debug("=====================");
                logger.debug(hello);
                logger.debug(kevinProperties.getName());
                logger.debug(kevinProperties.getFullName());
                logger.debug("=====================");
            }
        }
        ```
      
    * ② [Edit Configurations...]에서 VM Options 과 Program arguments에 설정한 내용을 제거한다.
    
    * ③ 애플리케이션을 실행하여 결과를 확인한 다음, 다음 실습을 위해 `application.properties`에서 `logging.level.me.kevinntech = DEBUG`를 제거한다.
        
* (2) 커스텀 로그 설정 파일 사용하기

    * ① 로거에 맞는 커스텀 로그 설정 파일을 다음과 같이 생성한다.
      
        * Logback: logback-spring.xml
      
        * Log4J2: log4j2-spring.xml
      
        * JUL (추천X): logging.properties

        ```html
        <?xml version="1.0" encoding="UTF-8"?>
        <configuration>
        <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
        <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
        <root level="INFO">
            <appender-ref ref="CONSOLE" />
        </root>
        <logger name="me.kevinntech" level="DEBUG"/>
        </configuration>
        ```
      
    * ② 애플리케이션을 실행하면 다음과 같이 DEBUG 레벨로 출력되는 것을 확인 할 수 있다.
    
        ![image 11](images/img11.png)
        
* (3) 로거를 Log4j2로 변경하기

    * 기본적으로는 Logback을 사용하게 되는데 Log4j2로 변경하는 방법은 다음과 같다.
   
    * 일단, 이전 실습에서 만들었던 `logback-spring.xml`를 삭제한다.
    
    * ① pom.xml에서 기본적으로 들어오는 로깅을 exclustion으로 제외한다.
    
    * ② 그리고 log4j2 의존성을 추가한다.
    
        ```html
        <?xml version="1.0" encoding="UTF-8"?>
        <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        
            ...
        
            <dependencies>
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-web</artifactId>
                    <exclusions>
                        <exclusion>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-starter-logging</artifactId>
                        </exclusion>
                    </exclusions>
                </dependency>
        
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-log4j2</artifactId>
                </dependency>
        
            </dependencies>
        
            ...
            
        </project>
        
        ```

    * ③ `application.properties`에 `logging.level.me.kevinntech = debug`를 추가한 다음, 애플리케이션을 실행한다.

    * 이제, 최종적으로 로그 메시지를 출력하는 것은 Log4J2가 된다.
    
#### 8) 테스트 - `핵심 기능`

* (1) 실습 전에 환경설정

    * ① 새로운 프로젝트를 생성한다.
    
    * ② 다음과 같이 Controller와 Service 클래스를 작성한다. 그리고 컨트롤러가 서비스를 호출하고 hello kevin를 리턴한다.
    
        ```java
        @RestController
        public class SampleController {
        
            @Autowired
            private SampleService sampleService;
        
            @GetMapping("/hello")
            public String hello() {
                return "hello " + sampleService.getName();
            }
        
        }
        ```
        
        ```java
        @Service
        public class SampleService {
        
            public String getName() {
                return "kevin";
            }
        }
        ```
      
    * ③ 테스트 코드를 작성 하기 전에 pom.xml에 다음 의존성이 있는지 확인한다.
    
        ```html
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        ```
      
* (2) 테스트 코드 작성

    * @SpringBootTest
    
        * 통합 테스트를 제공하는 스프링 부트 테스트 애노테이션이며 테스트에 필요한 모든 의존성을 제공한다.

        * JUnit 4에서는 @RunWith(SpringRunner.class)와 같이 사용해야 한다.
        
        * JUnit 5에서는 @ExtendWith(SpringExtension.class)가 이미 안에 존재하기 때문에 생략 가능하다.
        
        * webEnvironment의 타입
        
            * MOCK : 내장 톰캣 구동 안함
            * RANDOM_PORT, DEFINED_PORT : 내장 톰캣 사용 함
            * NONE : 서블릿 환경 제공 안함

            * webEnvironment가 RANDOM_PORT 또는 DEFINED_PORT 타입이면 해당 포트로 서블릿 컨테이너(내장 톰캣)를 구동한다.
            * 이때 부터는 MockMvc가 아닌 TestRestTemplate 이나 WebTestClient를 사용해야 한다.
            
    * MockMvc 사용
    
        * webEnvironment가 Mock 타입이면 서블릿 컨테이너를 구동 하지 않고 서블릿을 Mocking 한 것을 띄워준다.
          
        * 이 때, Mockup 된 서블릿과 상호 작용을 하려면 `MockMVC`라는 클라이언트를 사용해야 한다.
          
        * MockMVC 라는 클라이언트를 사용하려면 @AutoConfigureMockMvc 애노테이션을 붙여주고, MockMVC를 주입 받으면 된다.          
    
            ```java
            @RunWith(SpringRunner.class)
            @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
            @AutoConfigureMockMvc
            public class SampleControllerTest {
            
                @Autowired
                MockMvc mockMvc; // MockMvc를 주입 받는다.
            
                @Test
                public void hello() throws Exception {
                    mockMvc.perform(get("/hello"))      // MockMvc를 통해 "/hello"로 HTTP GET 요청을 한다.
                            .andExpect(status().isOk())         // status 코드가 200 OK
                            .andExpect(content().string("hello kevin")) // 컨텐츠가 hello kevin 이길 바란다.
                            .andDo(print());                    // 요청온 것을 출력한다.
                }
            
            }
            ```
          
            * ① @RunWith(SpringRunner.class)
            
                * JUnit에서 기본적으로 내장된 Runner가 아닌 스프링 Runner를 사용 하도록 한다.
        
            * ② mockMvc.perform(get("/hello"))
            
                * MockMvc를 통해 해당 주소로 HTTP GET 요청을 한다.

            * ③ .andExpect(status().isOk())
            
                * mockMvc.perform()의 결과를 검증한다.
                
                * HTTP Header의 상태(Status)가 200 OK 인지 검증한다.
        
            * ④ .andExpect(content().string("hello kevin"))
            
                * mockMvc.perform()의 결과를 검증한다.
                
                * 응답 본문의 내용을 검증한다.
        
    * TestRestTemplate 사용
     
        * 다음과 같이 webEnvironment를 RANDOM_PORT 타입으로 지정한 다음, TestRestTemplate를 사용해야 한다.
        
            ```java
            @RunWith(SpringRunner.class)
            @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
            @AutoConfigureMockMvc
            public class SampleControllerTest {
            
                @Autowired
                TestRestTemplate testRestTemplate;
            
                @Test
                public void hello() throws Exception {
                    String result = testRestTemplate.getForObject("/hello", String.class);
                    assertThat(result).isEqualTo("hello kevin");
            
                }
            
            }
            ```
          
        * 만약 테스트를 Service 단 까지 가지말고 Controller 만 테스트 하고 싶다면 @MockBean 애노테이션을 사용하여 SampleService 타입의 Mock 객체를 정의한다. (Mocking)
          
        * 그러면 SampleController가 사용하는 ApplicationContext에 있는 SampleService 빈을 Mock 객체(mockSampleService)로 교체한다.
          
        * 그래서 원본이 아닌 Mock bean을 사용해서 테스트할 수 있다.
        
            ```java
            @RunWith(SpringRunner.class)
            @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
            @AutoConfigureMockMvc
            public class SampleControllerTest {
            
                @Autowired
                TestRestTemplate testRestTemplate;
            
                @MockBean
                SampleService mockSampleService;
            
                @Test
                public void hello() throws Exception {
                    when(mockSampleService.getName()).thenReturn("kevin");
                    String result = testRestTemplate.getForObject("/hello", String.class);
                    assertThat(result).isEqualTo("hello kevin");
            
                }
            
            }
            ```
          
    * WebTestClient 사용
     
        * WebTestClient를 사용하면 비동기적으로 클라이언트 요청을 할 수 있다.
        
        * ① 먼저, pom.xml에 WebFlux 의존성을 추가한다.
        
            ```html
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-webflux</artifactId>
            </dependency>
            ```
          
        * ② 코드를 다음과 같이 수정한다.
        
            ```java
            @RunWith(SpringRunner.class)
            @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
            @AutoConfigureMockMvc
            public class SampleControllerTest {
            
                @Autowired
                WebTestClient webTestClient;
            
                @MockBean
                SampleService mockSampleService;
            
                @Test
                public void hello() throws Exception {
                    when(mockSampleService.getName()).thenReturn("kevin");
            
                    webTestClient.get().uri("/hello").exchange()
                            .expectStatus().isOk()
                            .expectBody(String.class).isEqualTo("hello kevin");
                }
            
            }
            ```
          
            * @SpringBootTest 애노테이션은 스프링 메인 애플리케이션(@SpringBootApplication)을 찾아가서 해당 패키지를 기준으로
              
            * 하위의 모든 Bean을 Scan 한 다음, Test용 ApplicationContext를 만들면서 빈으로 등록해주고, @MockBean을 찾아서 그 빈만 Mock 객체로 교체한다.
              
            * 그리고 MockBean는 @Test 마다 자동으로 리셋된다.
            
    * 슬라이싱 테스트 (단위 테스트)
     
        * 단위 테스트를 위한 @JsonTest, @WebMvcTest, @WebFluxTest, @DataJpaTest 애노테이션을 제공한다.
        
        * 레이어 별로 잘라서 테스트 하고 싶을 때 사용된다. (레이어 별로 빈이 등록 됨)
        
        * 다음과 같이, 하나의 컨트롤러만 테스트 할 수 있다.
          
        * Controller만 빈(Bean)으로 등록되고 Service는 빈으로 등록되지 않기 때문에 @MockBean 으로 주입 받아야 한다.
          
        * 그리고 @WebMvcTest는 MockMvc로 테스트 해야 한다.
        
            ```java
            @RunWith(SpringRunner.class)
            @WebMvcTest(SampleController.class)
            public class SampleControllerTest {
            
                @MockBean
                SampleService mockSampleService;
            
                @Autowired
                MockMvc mockMvc;
            
                @Test
                public void hello() throws Exception {
                    when(mockSampleService.getName()).thenReturn("kevin");
            
                    mockMvc.perform(get("/hello"))
                            .andExpect(content().string("hello kevin"));
                }
            
            }
            ```
          
        * SampleController 하나만 Bean으로 등록 되기 때문에, 훨씬 더 가벼운 테스트가 된다.
        
* (3) 테스트 유틸

    * 스프링 테스트가 제공하는 유틸은 다음과 같다.
    
        * ① OutputCapture
          
        * ② TestPropertyValues
          
        * ③ TestRestTemplate
          
        * ④ ConfigFileApplicationContextInitializer
        
    * 이 중 제일 유용할 것 같은 OutputCapture에 대해서 알아 보겠다.
    
    * OutputCapture는 JUnit의 Rule을 확장해서 만든 것이며 로그를 비롯한 console에 출력되는 모든 것을 캡처한다.
    
    * OutputCapture - 실습
    
        * ① SampleController에서 로거를 만든 다음, 로그를 출력한다.
        
            ```java
            @RestController
            public class SampleController {
            
                Logger logger = LoggerFactory.getLogger(SampleController.class);
            
                @Autowired
                private SampleService sampleService;
            
                @GetMapping("/hello")
                public String hello() {
                    logger.info("kevin");
                    System.out.println("skip"); // 이렇게 하면 안되지만 이것 또한 캡처된다는 것을 보여주기 위함
                    return "hello " + sampleService.getName();
                }
            
            }
            ```
          
        * ② outputCapture를 통해 holoman과 skip이 출력되는지 확인 할 수 있다.

            ```java
            @RunWith(SpringRunner.class)
            @WebMvcTest(SampleController.class)
            public class SampleControllerTest {
            
                @Rule
                public OutputCapture outputCapture = new OutputCapture();
            
                @MockBean
                SampleService mockSampleService;
            
                @Autowired
                MockMvc mockMvc;
            
                @Test
                public void hello() throws Exception {
                    when(mockSampleService.getName()).thenReturn("kevin");
            
                    mockMvc.perform(get("/hello"))
                            .andExpect(content().string("hello kevin"));
            
                    assertThat(outputCapture.toString())
                            .contains("kevin")
                            .contains("skip");
                }
            
            }
            ```
          
#### 9) Spring-Boot-Devtools - `핵심 기능`

* (1) Spring-Boot-Devtools

    * `Spring-Boot-Devtools`는 스프링 부트가 제공하는 optional한 tool이다.
    
    * 사용 하려면 pom.xml에 의존성을 추가 해야 한다.
    
        ```html
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        ```
      
    * Devtools의 의존성을 추가하는 순간, properties의 설정들이 자동으로 변경된다. 캐시 관련 설정을 개발 환경에 맞게 꺼준다.

    * 클래스패스에 있는 파일이 변경 될 때마다 애플리케이션을 자동으로 재 시작한다. 우리가 직접 톰캣을 종료 했다가 구동하는 것 보다는 빠르다. 왜냐하면 스프링 부트는 2개의 클래스 로더를 사용하기 때문이다.
    
        * ① base classloader: 라이브러리들, 우리가 바꾸지 않는 의존성을 읽어들이는 클래스 로더이다.
        
        * ② restart classloader: 애플리케이션을 읽어 들이는 클래스 로더이다.
        
    * 이를 테스트 하기 위해서는 소스코드를 변경한 다음, [Build Project]를 클릭하고 웹 브라우저를 새로고침하면 확인 할 수 있다. (해당 기능이 되지 않을 때도 있음...)
    
    * 리스타트 했을 때, 브라우저를 자동 새로고침(refresh)하는 기능을 라이브 리로드라고 한다. 해당 기능을 적용 하려면 웹 브라우저의 플러그인을 설치해야 한다.
       
    * 글로벌 설정
    
        * spring-boot-devtools가 의존성에 있으면 ~/.spring-boot-devtools.properties가 프로퍼티 우선 순위 중에 우선 순위가 가장 높다.
      

        
        
    