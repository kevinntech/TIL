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

    * `src\main\java`에 디폴트 패키지(me.kevinntech)를 만들고 여기에 @SpringBootApplication이 붙어있는 클래스가 위치 하도록 해야한다.
    
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


    