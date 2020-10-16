# 백기선님의 스프링 프레임워크 핵심 기술

## 1. 스프링 소개
## 2. IoC 컨테이너와 빈
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
