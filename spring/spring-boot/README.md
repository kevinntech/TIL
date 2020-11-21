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
    
    * `Xxx-Spring-Boot-Start` 모듈 : 필요한 의존성 정의
    
        * 그냥 하나로 만들고 싶다면? `Xxx-Spring-Boot-Starter`
  
* (2) `Xxx-Spring-Boot-Starter`만 만드는 구현 방법

    * ① 먼저, maven 프로젝트를 xxx-spring-boot-starter 형식으로 생성한다.

    * ② pom.xml에 의존성을 추가한다. me.kevinntech

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
      
    * ④ 앞서 만든 클래스의 자동 설정 파일(@Configuration)을 만든다.
   
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
    
    * ⑧ 의존성이 들어온 것을 확인 할 수 있다.

        ```java
        <dependency>
        	<groupId>org.example</groupId>
        	<artifactId>kevin-spring-boot-starter</artifactId>
        	<version>1.0-SNAPSHOT</version>
        </dependency>
        ```

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
      
    * 앞서 살펴본 구현 방식은 발생할 수 있는 문제가 있다.

        * 스프링 부트의 메인 애플리케이션에 명시적으로 빈을 등록하면 @ComponentScan에 의해서 빈으로 등록 된 다음, 
        
        * @EnableAutoConfiguration는 자동 설정 파일을 읽어 빈으로 등록하며 이때, 명시적으로 등록한 빈을 덮어쓰게 된다는 문제점이 있다.

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

앞서 살펴본 자동 설정을 만드는 방식에는 문제점이 있었다. 문제점을 해결하기 위한 방법을 아래에서 하나씩 살펴본다.

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
        
            * 해당 bean이 생성되어 있지 않은 경우에만 조건이 만족된다.
            
            * @Bean과 함께 사용되어 조건을 만족하는 경우에만 빈을 생성한다.
            
        * `@ConditionalOnClass`
        
            * 클래스패스에 해당 class가 존재할 때만 조건이 만족된다. 
            
    * ② 그리고 Maven install을 한다.
    
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
      
    * ⑥ kevin-spring-boot-starter 프로젝트에서 install 한 다음, spring-boot-second 프로젝트에서 refresh를 한 다음, 실행한다.

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
        
* 즉, 스프링 부트의 자동 설정이 내장 서블릿 컨테이너(Tomcat, Jetty, Undertow)를 설정하고 실행 시켜준다.

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
    
    * 즉, 앞으로는 모든 요청에 https를 붙여야 한다. (http로 접근하면 Bad Request가 나타남)
    
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
  
    * http2로 요청을 보내면 http2를 사용하고 있는 것을 알 수 있다.
        
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
    
