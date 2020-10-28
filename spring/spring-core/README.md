# 백기선님의 스프링 프레임워크 핵심 기술
> 아래 내용은 [스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core "스프링 프레임워크 핵심 기술") 강좌를 정리한 내용 입니다.

## 1. 스프링 소개

#### 1) 스프링이란?

* 스프링은 “소규모 애플리케이션 또는 기업용 애플리케이션을 자바로 개발하는데 있어 유용하고 편리한 기능을 제공하는 프레임워크"이다.

#### 2) 스프링의 역사

* 2003년 등장

* 최근까지 주로 서블릿 기반 애플리케이션을 만들 때 사용해 옴.

* 스프링 5 부터는 WebFlux 지원으로 서블릿 기반이 아닌 서버 애플리케이션도 개발할 수 있게 됨.

#### 3) 스프링의 디자인 철학 

* 모든 선택은 개발자의 몫.

* 다양한 관점을 지향한다. 

* 하위 호환성을 지킨다. 

* API를 신중하게 설계한다.

* 높은 수준의 코드를 지향한다.

## 2. IoC 컨테이너와 빈

### 2-1. IoC 컨테이너 1부: 스프링 IoC 컨테이너와 빈

#### 1) 제어의 역전 (IoC : Inversion of Control)

* `제어의 역전(IoC)`은 `의존성 주입(Dependency Injection)`이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라,주입 받아 사용하는 방법을 말한다.

* 의존 객체를 직접 만들어 사용하는 예시는 다음과 같다.

```java
BookRepository bookRepository = new BookRepository();
 
BookService service = new BookService(bookRepository);
```

* IoC의 예시는 다음과 같다.

```java
// BookService 타입의 객체가 사용할 bookRepository라는 의존 객체를 
// 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말함

@Autowired
BookRepository bookRepository;
 
BookService service = new BookService(bookRepository);
```

#### 2) 빈(Bean)

* `빈(Bean)`은 스프링 IoC 컨테이너가 관리 하는 객체이다

* 스프링에서 빈으로 등록될 때의 장점은 다음과 같다.
  
    * 의존성 관리 
    
        * 의존성 주입을 받으려면 빈이 되어야 한다. 
  
    * 객체의 스코프 관리가 용이 
  
        * 스프링 IoC 컨테이너에 등록되는 빈은 기본적으로 싱글톤 scope 으로 등록된다.  
        * (메모리 측면에서 효율적이며 런타임 시 성능 최적화에 유리함.) 
  
    * 라이프 사이클 인터페이스를 제공한다.  `@PostConstruct`

#### 3) 스프링 IoC 컨테이너

* `스프링 IoC 컨테이너`는 빈 설정 파일로 부터 빈 정의를 읽어 들이고 빈을 생성한 다음, 제공하는 역할을 한다.

* Annotation을 사용하여 POJO 객체를 Bean으로 등록하고 Bean으로 등록된 객체를 주입 받아서 사용한다.

#### 4) 스프링 IoC 컨테이너 관련 인터페이스

##### (1) BeanFactory

* `BeanFactory`는 스프링 IoC 컨테이너의 최상위에 있는 인터페이스이다.

* IoC 컨테이너 기능을 수행한다. (빈을 생성하고 의존성을 관리)

##### (2) ApplicationContext

* `ApplicationContext`는 BeanFactory를 상속 받은 인터페이스이다.

* `ApplicationContext`는 BeanFactory의 IoC 컨테이너 기능을 가지고 있으면서도 다음과 같은 추가적인 기능을 가진다. 

    * 국제화 기능 (i18n) `MessageSource`

    * 이벤트 발행 기능 `ApplicationEventPublisher`

    * 리소스 로딩 기능 `ResourceLoader`

    * 프로파일과 프로퍼티 `EnvironmentCapable`
    
* `ApplicationContext` 인터페이스를 구현한 대표적인 클래스

    * `ClassPathXmlApplicationContext` [XML]  

    * `AnnotationConfigApplicationContext` [Java]  

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

* `Converter`는 S 타입을 T 타입으로 변환 할 수 있는 변환기이다.

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
