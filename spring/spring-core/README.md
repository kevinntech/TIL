# 백기선님의 스프링 프레임워크 핵심 기술
> 아래 내용은 [스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core "스프링 프레임워크 핵심 기술") 강좌를 정리한 내용 입니다.

## 1. 스프링 소개

#### 1) 스프링이란?

* 스프링은 “소규모 애플리케이션 또는 기업용 애플리케이션을 자바로 개발하는데 있어 유용하고 편리한 기능을 제공하는 프레임워크"이다.

#### 2) 스프링의 역사

* 2003년 등장

* 최근까지 주로 서블릿 기반 애플리케이션을 만들 때 사용해 옴.

* 스프링 5 부터는 WebFlux 지원으로 서블릿 기반이 아닌 서버 애플리케이션도 개발할 수 있게 됨.

#### 3) 스프링의 디자인 철학

* 모든 선택은 개발자의 몫.

* 다양한 관점을 지향한다. 

* 하위 호환성을 지킨다. 

* API를 신중하게 설계한다.

* 높은 수준의 코드를 지향한다.

## 2. IoC 컨테이너와 빈

### 2-1. IoC 컨테이너 1부: 스프링 IoC 컨테이너와 빈

#### 1) 제어의 역전(IoC: Inversion of Control)

* `제어의 역전(IoC)`은`의존성 주입(Dependency Injection)`이라고도 하며,어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라,주입 받아 사용하는 방법을 말한다.

* 의존 객체를 직접 만들어 사용하는 예시는 다음과 같다.

```java
BookRepository bookRepository = new BookRepository();
 
BookService service = new BookService(bookRepository);
```

* IoC의 예시는 다음과 같다.

```java
// BookService 타입의 객체가 사용할 bookRepository라는 의존 객체를 
// 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말함

@Autowired
BookRepository bookRepository;
 
BookService service = new BookService(bookRepository);
```

#### 2) 빈(Bean)

* `빈(Bean)`은 스프링 IoC 컨테이너가 관리 하는 객체이다

* 스프링에서 빈으로 등록될 때의 장점은 다음과 같다.
  
    * 의존성 관리

        * 의존성 주입을 받으려면 빈이 되어야 한다.
  
    * 객체의 스코프 관리가 용이
  
        * 스프링 IoC 컨테이너에 등록되는 빈은 기본적으로 싱글톤 scope 으로 등록된다.
        * (메모리 측면에서 효율적이며 런타임 시 성능 최적화에 유리함.)
  
    * 라이프 사이클 인터페이스를 제공한다. `@PostConstruct`

#### 3) 스프링 IoC 컨테이너

* `스프링 IoC 컨테이너`는 빈 설정 파일로 부터 빈 정의를 읽어 들이고 빈을 생성한 다음, 제공하는 역할을 한다.

* Annotation을 사용하여 POJO 객체를 Bean으로 등록하고 Bean으로 등록된 객체를 주입 받아서 사용한다.

#### 4) 스프링 IoC 컨테이너 관련 인터페이스

##### (1) BeanFactory

* `BeanFactory`는 스프링 IoC 컨테이너의 최상위에 있는 인터페이스이다.

* IoC 컨테이너 기능을 수행한다. (빈을 생성하고 의존성을 관리)

##### (2) ApplicationContext

* `ApplicationContext`는 BeanFactory를 상속 받은 인터페이스이다.

* `ApplicationContext`는 BeanFactory의 IoC 컨테이너 기능을 가지고 있으면서도 다음과 같은 추가적인 기능을 가진다.

    * 국제화 기능 (i18n) `MessageSource`

    * 이벤트 발행 기능 `ApplicationEventPublisher`

    * 리소스 로딩 기능 `ResourceLoader`

    * 프로파일과 프로퍼티 `EnvironmentCapable`
    
* `ApplicationContext` 인터페이스를 구현한 대표적인 클래스

    * `ClassPathXmlApplicationContext` [XML]

    * `AnnotationConfigApplicationContext` [Java]

### 2-2. IoC 컨테이너 2부: ApplicationContext와 다양한 빈 설정 방법

#### 1) 빈 설정 파일

* 스프링 IoC 컨테이너는 `빈 설정 파일`이 필요하다.

#### 2) 빈을 등록(설정)하는 방법과 의존성 주입

##### 2-1) `application.xml`에 빈을 등록하는 방법

* 빈(Bean) 등록 및 의존성 주입하기

    * 다음과 같이 빈(Bean)으로 등록할 클래스를 작성한다.
    
        ```java
        public class BookService {
        
            // BookService가 BookRepository를 사용한다고 가정
            BookRepository bookRepository;
        
            public void setBookRepository(BookRepository bookRepository) {
                this.bookRepository = bookRepository;
            }
        }
        ```
      
    * resources 디렉토리 아래에 application.xml 파일을 다음과 같이 작성 한다.
    
        ```html
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        
            <bean id="bookService"
                  class="me.whiteship.springapplicationcontext.BookService">
                <property name="bookRepository" ref="bookRepository"></property>
            </bean>
        
            <bean id="bookRepository"
                  class="me.whiteship.springapplicationcontext.BookRepository">
            </bean>
        
        </beans>
        ```
      
        * `<bean>` 태그는 빈을 등록하는 태그이다.
        
        * `<bean>` 태그의 id는 빈을 구분할 때 사용하는 이름을 의미하며 camel-case로 작성한다.
        
        * 그리고 class는 빈의 타입을 의미한다.
        
        * `<property>` 태그는 setter를 이용하여 의존성을 주입 받을 때 사용한다.
        
        * `<property>` 태그의 name은 setter의 이름에서 가져온 것이며 ref는 (해당 setter의 매개변수로 전달 될 수 있는) 다른 bean 의 id를 참조한다.
        
* 빈(Bean)을 사용하기

    * 빈 설정 파일을 사용하는 ApplicationContext를 ClassPathXmlApplicationContext 클래스를 이용하여 만든 다음, Bean을 사용 할 수 있다.

        ```java
        Applicationcontext context = new ClassPathXmlApplicationContext("application.xml");
        ```

    * 해당 방법은 빈을 일일이 등록 해야 하기 때문에 번거롭다. 이러한 이유로 등장한 것이 `<context:component-scan>` 이다.
    
##### 2-2) `application.xml` 과 `<context:component-scan>` 태그를 활용한 빈 등록 

* 빈 설정 파일(application.xml)에 `<context:component-scan>` 태그를 이용하면 base-package에서 부터
* @Component와 이를 확장한 애노테이션(@Service, @Repository, @Controller)을 스캔하여 해당 클래스를 빈으로 등록한다.

    ```html
    <context:component-scan base-package="me.whiteship.springapplicationcontext"/>
    ```
  
* 그리고 @Autowired 애노테이션을 사용하여 빈을 주입 받을 수 있다.  즉, 컨테이너에서 빈을 꺼낼 수 있다.

    ```java
    @Service
    public class BookService {
    
        @Autowired
        BookRepository bookRepository;
    
        public void setBookRepository(BookRepository bookRepository) {
            this.bookRepository = bookRepository;
        }
    }
    ```

##### 2-3) Java 설정 파일

* 빈 설정 파일을 xml이 아닌 Java로 만들 수 없을까? 라는 생각에 등장한 것이 바로 Java 설정 파일이다.

* 방금 전까지 선언 했던 애노테이션(@Repository, @Service, @Autowird)을 모두 제거한다.

* 그리고 @Configuration을 붙인 ApplicationConfig 클래스를 작성한다.

    * 빈(Bean) 등록 및 의존성 주입하기
    
        * Java 설정 파일을 이용하여 빈 등록 및 의존성 주입을 한다.
    
            ```java
            @Configuration
            public class ApplicationConfig {
                
                @Bean
                public [등록할 빈의 타입][빈의 id](){
                    return new [등록할 빈의 타입]();
                }
                
            }
            ```
          
            * @Configuration : 스프링 IoC 컨테이너에게 해당 클래스가 빈 설정 파일이라는 것을 알려준다.
       
            * @Bean : 메서드의 실행 결과로 반환되는 객체를 Bean으로 등록한다. (클래스에 @Bean 선언 불가능)
            
                * @Configuration이 선언된 클래스 내에 있는 메서드에 사용된다.
                
                * 보통 외부 라이브러리들을 Bean으로 등록하고 싶은 경우에 사용된다.
                
            ```java
            @Configuration
            public class ApplicationConfig {
            
                @Bean
                public BookRepositry bookRespository() {
                  return new BookRepository();
                }
            
                @Bean
                public BookService bookService() {
                  BookService bookService = new BookService();
                  bookService.setBookRepositry(bookRepository());  // setter를 사용한 의존성 주입
                   return bookService;
                }
            }
            ```        
      
    * 빈(Bean)을 사용하기
    
        * 빈 설정 파일을 사용하는 ApplicationContext를 AnnotionConfigApplicationContext 클래스를 이용하여 만든 다음, Bean을 사용 할 수 있다.
        
            ```java
            ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class)
            ```  

##### 2-4) Java 설정 파일에 @ComponentScan를 활용한 빈 등록  ★★★

* 앞서 살펴본 Java 설정도 Bean을 일일이 등록해야 하는 번거로움이 있어서 xml에서 처럼 컴포넌트 스캔을 사용 할 수 있다.

* @ComponentScan 애노테이션을 이용하여 지정한 Class가 위치한 곳부터 컴포넌트 스캔을 하여 Bean으로 등록한다.

    * 빈(Bean) 등록 및 의존성 주입하기
    
        ```java
        @Configuration
        // 해당 class가 위치한 곳부터 Component Scan을 한다.
        @ComponentScan(basePackageClasses = DemoApplication.class) 
        public class ApplicationConfig { 
        
        }
        ```
        * basePackages로 class path를 사용 할 수도 있지만 basePackageClasses가 더 type-safe 하다.
        
    * 빈(Bean)을 사용하기
    
        * 빈 설정 파일을 사용하는 ApplicationContext를 AnnotionConfigApplicationContext 클래스를 이용하여 만든 다음, Bean을 사용 할 수 있다.

##### 2-5) @SpringBootApplication  ★★★

* @SpringBootApplication는 내부적으로 @Configuration과 @ComponentScan 애노테이션이 포함되어 있기 때문에

* @SpringBootApplication이 붙어 있는 클래스 자체가 Bean 설정 파일이다.

* 그래서 SpringBoot의 경우, ApplicationConfig 파일이 필요 없다.
           
```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 2-3. IoC 컨테이너 3부: @Autowired

#### 1) @Autowired

* `@Autowired`는 필요한 의존 객체의 “타입"에 해당하는 빈을 찾아 주입한다.

#### 2) @Autowired의 required

* @Autowired의 required는 true가 기본 값이다.
 
* 필요한 의존 객체의 타입에 해당하는 빈을 찾지 못하면 애플리케이션 구동에 실패한다.

```java
@Service
public class BookService {

    BookRepository bookRepository;

    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

* BookRepository가 빈으로 등록되어 있지 않다고 가정 할 때, 아래 코드에서 required를 false로 지정 하면, 의존성 주입을 할 수 없더라도 애플리케이션을 구동한다. (필드, 세터만 가능)

* 즉, BookRepository가 의존성 주입이 되지 않은 상태로 BookService가 빈으로 등록된다.

```java
@Service
public class BookService {

    BookRepository bookRepository;

    @Autowired(required = false)
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

* 해당 내용은 필드에도 적용 가능 하다.

```java
@Service
public class BookService {

    @Autowired(required = false)
    BookRepository bookRepository;
}
```

#### 3) @Autowired를 사용 할 수 있는 위치

* @Autowired는 생성자, 세터(setter), 필드(field)에 사용 할 수 있다.

    * 생성자
    
        ```java
        @Service
        public class BookService {
        
            BookRepository bookRepository;
        
            @Autowired
            public BookService(BookRepository bookRepository) {
                this.bookRepository = bookRepository;
            }
        }
        ```
      
    * 세터(setter)
  
      ```java
      @Service
      public class BookService {
      
        BookRepository bookRepository;
      
        @Autowired
         public void setBookRepository(BookRepository bookRepository) {
                this.bookRepository = bookRepository;
        }
      
      }
      ```
   
    * 필드(field)

        ```java
        @Service
        public class BookService {
        
            @Autowired
            BookRepository bookRepository;
        }
        ```
      
#### 4) @Autowired에서 발생 할 수 있는 경우의 수

* 해당 타입의 빈이 없는 경우 

* 해당 타입의 빈이 한 개인 경우  

* 해당 타입의 빈이 여러 개인 경우

    * 빈 이름으로 시도하여 같은 이름의 빈을 찾으면 해당 빈 사용 

    * 빈 이름으로 시도하여 같은 이름의 빈을 찾지 못하면 실패

#### 5) @Autowired에서 발생 할 수 있는 경우의 수

* 아래의 BookRepository 인터페이스를 구현한 클래스(MyBookRepository, KeesunBookRepository)가 2개 있다.

```java
public interface BookRepository { }

@Repository
public class MyBookRepository implements BookRepository { }

@Repository
public class KeesunBookRepository implements BookRepository{ }
```
      
* BookService 클래스에서는 @Autowired로 BookRepository를 주입 받으려고 하는데 에러가 발생한다.    

* 에러가 발생한 이유는 두 개의 클래스 중 어떤 클래스를 주입해야 하는지 알 수 없기 때문이다. 

* 해결책으로는 다음 3가지가 존재한다.

    * `@Primary` : 같은 타입의 빈이 여러 개 일 때, 우선 순위를 가지는 빈으로 지정하여 해당 빈이 주입 되도록 한다.

        ```java
         @Repository
         @Primary
         public class KeesunBookRepository implements BookRepository{}
        ```

    * `@Qualifier`
    
        * `@Qualifier`는 @Autowired와 함께 사용하며 빈의 이름이 같은 객체를 찾는다.
          
        * `@Qualifier`에 전달하는 빈의 이름은 클래스 명에서 첫 글자만 소문자로 변경하여 지정한다.
        
            ```java
            @Autowired
            @Qualifier("keesunBookRepository")
            BookRepository bookRepository;
            ```
                 
        * @Qualifier 보다 좀 더 type-safe 한 @Primary를 사용하는 것을 권장한다. 

    * `해당 타입의 모든 빈을 주입 받기`
    
        * 다음과 같이 List를 사용하면 해당 타입의 (BookRepository) 타입의 모든 빈을 주입 받을 수 있다.  

            ```java
            @Autowired
            List<BookRepository> bookRepository;
            ```

#### 6) @Autowired 동작 원리

* BeanPostProcessor의 구현체인 AutowiredAnnotationBeanPostProcessor가 Bean의 초기화 라이프 사이클 이전에 @Autowired 애노테이션이 붙어 있는 빈(Bean)을 찾아 주입 한다.

### 2-4. IoC 컨테이너 4부: @Component와 컴포넌트 스캔

#### 1) @ComponentScan

* `@ComponentScan`는 @Component 애노테이션이 붙어 있는 클래스들을 스캔하여 빈으로 등록한다.

    * `BasePackages`는 입력된 패키지의 경로를 기준으로 스캔을 시작한다. 
        * 패키지의 경로를 문자열로 입력 받기 때문에 type-safe 하지 않음

    * `BasePackageClasses`는 입력된 클래스가 위치한 패키지를 기준으로 스캔을 시작한다.
    
* 스프링 부트에서는 `@SpringBootApplication`이 붙어 있는 클래스가 컴포넌트 스캔의 시작 지점이다.

#### 2) @ComponentScan의 주요 기능

* 스캔 위치를 설정하고, 어떤 애노테이션을 스캔 할지 또는 하지 않을지 결정하는 필터(Filter) 기능을 가지고 있다.

#### 3) 컴포넌트 스캔 대상 : @Component를 확장한 애노테이션

* @Repository

* @Service

* @Controller

* @Configuration

### 2-5. IoC 컨테이너 5부: 빈의 스코프(Scope)

#### 1) 빈의 범위(Scope)

* `싱글톤`은 해당 Bean의 인스턴스를 단 한번만 생성한다. (기본 값)

* `프로토타입`은 해당 Bean의 인스턴스를 매번 새롭게 생성한다. 

* 프로토타입과 유사한 Scope는 Request, Session, WebSocket 등이 있다.

#### 2) 빈의 범위(Scope) 관련 실습 

* 빈의 Scope에 대해서 알아보기 위해 다음과 같은 실습을 진행한다.

* Proto라는 클래스를 작성하고 Bean으로 등록한다. 그리고 @Scope를 붙여서 prototype으로 지정한다.

    ```java
    @Component
    @Scope("prototype")
    public class Proto {
    }
    ```
  
* Single이라는 클래스를 작성한 다음, Bean으로 등록하고 Proto 타입의 필드에 의존성을 주입 받는다.

* 그리고 Single 클래스의 proto 값을 가져 올 수 있도록 getter를 작성한다.

    ```java
    @Component
    public class Single {      
        @Autowired
        private Proto proto;
    
        public Proto getProto() { // Single이 참조하고 있는 Proto를 반환
            return proto;
        }
    }
    ```
  
* AppRunner 클래스에서 ApplicationContext를 이용하여 getBean()의 내용을 출력 해보면

* 싱글톤의 스코프인 Single 빈은 항상 같은 인스턴스이며 프로토타입의 스코프인 Proto 빈은 매번 새로운 인스턴스를 생성하는 것을 확인 할 수 있다.
  
  ```java
  @Component
  public class AppRunner implements ApplicationRunner {
  
      @Autowired
      ApplicationContext ctx;
  
      @Override
      public void run(ApplicationArguments args) throws Exception {
          System.out.println("proto");
  
          System.out.println(ctx.getBean(Proto.class));
          System.out.println(ctx.getBean(Proto.class));
          System.out.println(ctx.getBean(Proto.class));
  
          System.out.println("single");
          System.out.println(ctx.getBean(Single.class));
          System.out.println(ctx.getBean(Single.class));
          System.out.println(ctx.getBean(Single.class));
      }
  }
  ```
  
* ApplicationRunner 인터페이스를 구현한 클래스에 @Component 애노테이션을 선언하면 컴포넌트 스캔이 되고 스프링 부트 구동 시점에 run 메서드의 코드가 실행된다.

#### 3) 싱글톤과 프로토타입을 같이 사용 하는 경우, 발생할 수 있는 문제점 

* 프로토타입 빈이 싱글톤 빈을 참조하는 경우

    * 아무 문제가 없다.
    
        * 그 이유는 프로토타입의 Bean은 Ioc 컨테이너에서 매번 꺼낼 때 마다 항상 새로운 인스턴스를 생성하지만
          
        * 그 새로운 인스턴스가 참조하고 있는 싱글톤 Bean은 항상 동일하며 원래 의도한대로 사용되는 것이므로 문제가 없다.  

* 싱글톤 빈이 프로토타입 빈을 참조하는 경우 

    * 문제가 발생함
    
        * 싱글톤 Bean은 인스턴스가 단 한번만 생성 된다. 그리고 생성될 때 프로토타입 Bean의 프로퍼티도 이미 설정 되었기 때문에 
          
        * 매번 싱글톤 Bean을 사용할 때, 프로토타입 빈의 프로퍼티가 변경되지 않는 문제점이 있다.  

      ```java
      @Component
      public class AppRunner implements ApplicationRunner {
      
          @Autowired
          ApplicationContext ctx;
      
          @Override
          public void run(ApplicationArguments args) throws Exception {
      
              System.out.println("proto by single");
      
              /* 여기서 getProto()가 매번 다른 인스턴스를 반환하는 것을 의도 하였는데
                 결과는 그렇지 않음 */
              System.out.println(ctx.getBean(Single.class).getProto());
              System.out.println(ctx.getBean(Single.class).getProto());
              System.out.println(ctx.getBean(Single.class).getProto());
          }
      }
      ``` 
  
  * 앞서 살펴본 문제점을 해결하는 방법은 @Scope 애노테이션을 사용할 때 proxyMode를 설정하는 것이다.
  
    * scopedProxyMode.DEFAULT가 기본 값이며  이는 Proxy를 사용하지 않는다는 것을 의미한다.
    
    * `proxyMode = scopedProxyMode.TARGET_CLASS` → 해당 Bean을 클래스 기반의 Proxy로 감싸도록 한다. 

    * Proxy로 감싸는 이유는 무엇일까?
    
        * 싱글톤 Bean이 프로토타입 Bean을 직접 참조하는 것이 아닌 Proxy를 거쳐서 참조 하도록 해야 프로토타입 Bean을 매번 새로운 인스턴스로 바꿔 줄 수 있다.

          ```java
          @Component 
          @Scope(value = "prototype", proxyMode = scopedProxyMode.TARGET_CLASS)
          public class proto {
          
          }
          ```
              
#### 4) 싱글톤 객체 사용 시 주의할 점

* 프로퍼티(필드)가 공유 되므로 thread-safe 할 것이라고 보장 받을 수 없음
    * 그러므로 thread-safe한 방법으로 코딩을 해야 한다.
    
* 싱글톤 객체는 ApplicationContext 초기 구동 시 생성하게 된다.
                      
## 3. Resource / Validation

## 4. 데이터 바인딩

### 4-1. 데이터 바인딩 추상화 : PropertyEditor

#### 1) 데이터 바인딩

* 데이터 바인딩은 사용자가 입력한 값을 애플리케이션 도메인 객체에 동적으로 변환해 넣어주는 기능이다.

* 즉, 사용자가 입력한 값은 주로 문자열이며 이 문자열을 객체가 가지고 있는 다양한 프로퍼티 타입으로 변환해서 넣어주는 기능을 말한다.

* Spring은 데이터 바인딩 기능을 여러 인터페이스(PropertyEditor, Converter, Formatter)로 추상화하여 제공한다.

* PropertyEditor는 DataBinder 인터페이스를 통해 사용되며 Converter와 Formatter는 ConversionService를 통해 사용된다.

#### 2) PropertyEditor

* `PropertyEditor`는 스프링 3.0 이전까지 DataBinder가 변환 작업에 사용한 인터페이스이다.

* 상태 정보을 저장 하고 있어서 쓰레드-세이프 하지 않음

* 일반적인 싱글톤 scope 빈으로 등록해서 사용 할 수 없음 

* Object와 String 간의 변환만 할 수 있어 사용 범위가 제한적임

### 4-2. 데이터 바인딩 추상화: Converter와 Formatter

#### 1) Converter

* `Converter`는 S 타입을 T 타입으로 변환 할 수 있는 변환기이다.

* 상태 정보 없으므로(Stateless) 쓰레드 세이프하다.

* ConverterRegistry에 등록해서 사용한다.

```java
public class EventConverter {
    
    public static class StringToEventConverter implements Converter<String, Event>{
        @Override
        public Event convert(String source) {
            return new Event(Integer.parseInt(source));
        }
    }

    public static class EventToStringConverter implements Converter<Event, String>{
        @Override
        public String convert(Event source) {
            return source.getId().toString();
        }
    }
}
```

#### 2) Formatter

* `Formatter`는 PropertyEditor 대체재이며 Object와 String 간의 변환을 담당한다.

* Locale에 따라 문자열을 다국화 하는 기능도 제공한다.

* FormatterRegistry에 등록해서 사용

* thread-safe 하므로 빈으로 등록해서 사용 할 수도 있다.

```java
public class EventFormatter implements Formatter<Event> {

    @Override
    public Event parse(String text, Locale locale) throws ParseException {
        return new Event(Integer.parseInt(text));
    }

    @Override
    public String print(Event object, Locale locale) {
        return object.getId().toString();
    }
}
```

#### 3) ConversionService

* `Converter`와 `Formatter`는 `ConversionService`에 등록 되어 실제 변환 작업을 하게 된다.

* Spring이 제공하는 여러 가지 ConversionService 구현체 중 DefaultFormattingConversionService가 자주 사용된다.

* DefaultFormattingConversionService는 ConversionService와 FormatterRegistry를 구현하고 있다.

#### 4) 스프링 부트에서의 Converter와 Formatter

* 스프링 부트의 경우, 자동으로 DefaultFormattingConversionSerivce를 상속하여 만든 WebConversionService를 빈으로 등록해 준다.  

* 스프링 부트는 자동으로 Formatter와 Converter 빈을 찾아 ConversionService에 등록해 준다. 

## 5. SpEL (Spring Expression Language)

#### 1) 스프링 EL 이란?

* `SpEL`은 객체 그래프를 조회하고 조작하는 기능을 제공한다.

* Unified EL과 비슷하지만, 메서드 호출을 지원하며, 문자열 템플릿 기능도 제공한다. 

#### 2) SpEL 문법

##### ① 표현식

* `#{"표현식"}`는 표현식을 평가(실행)한다.

* 표현식 안에는 프로퍼티를 사용 할 수 있다. `#{${my.data} + 1}`는 가능

##### ② 프로퍼티 참조

* `${"프로퍼티"}`는 프로퍼티를 참조한다.

#### 3) SpEL 예시

* `application.properties`에 `my.value = 100`가 저장 되어 있다고 가정한다.

* `@Value`에 SpEL로 작성 되어 있다면 SpEL를 평가해서 결과 값을 변수에 할당한다.

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Value("#{1 + 1}")
    int value;

    @Value("#{'hello ' + 'world'}")
    String greeting;

    @Value("#{1 eq 1}")
    boolean trueOrFalse;

    @Value("hello")
    String hello;

    @Value("${my.value}")
    int myValue;

    @Value("#{${my.value} eq 100}")
    boolean isMyValue100;

    @Value("#{'spring'}")
    String spring;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=================");
        System.out.println(value);
        System.out.println(greeting);
        System.out.println(trueOrFalse);
        System.out.println(hello);
        System.out.println(myValue);
        System.out.println(isMyValue100);
        System.out.println(spring);
    }
}
```

## 6. 스프링 AOP

## 7. Null-Safety
