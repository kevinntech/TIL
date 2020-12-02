# 백기선님의 스프링 웹 MVC
> 아래 내용은 [스프링 웹 MVC](https://www.inflearn.com/course/%EC%9B%B9-mvc# "스프링 웹 MVC") 강좌를 정리한 내용 입니다.

## 1. 스프링 MVC 동작 원리

#### 1) 스프링 MVC 소개

* (1) 롬복(Lombok)

    * 롬복(Lombok)은 자바로 개발할 때, 자주 사용하는 코드(Getter, Setter, 기본 생성자, toString 등)를 애노테이션으로 자동 생성 해준다.
    
    * 롬복 설치 과정
    
        * ① 롬복 의존성 추가하기
    
            ```html
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.16.20</version>
                <scope>provided</scope>
            </dependency>
            ```
          
         * ② 롬복 플러그인 설치하기
         
            * 롬복(Lombok) 플러그인을 Install 한 다음, 인텔리제이를 재 시작 해야 한다.
                
            ![image 1](images/img1.png)
                
         * ③ `Enable annotation processing`을 체크한다.       
    
            ![image 2](images/img2.png)
     
          * ④ 지금부터 해당 프로젝트에서 롬복을 사용 할 수 있다.
          
    * 자주 사용하는 롬복 애노테이션
    
        * `@Getter` : getter 메서드를 자동 생성한다.
        * `@Setter` : setter 메서드를 자동 생성한다.
        * `@NoArgsConstructor` : 매개변수가 없는 생성자(기본 생성자)를 자동 생성한다.
        * `@AllArgsConstructor` : 모든 필드 값을 매개변수로 받는 생성자를 자동 생성한다.
        * `@RequiredArgsConstructor` : `final`이나 `@NonNull`인 필드 값만 매개변수로 받는 생성자를 자동 생성한다.
        * `@ToString` : toString() 메서드를 자동 생성한다. exclude 속성을 사용하여 특정 필드를 결과에서 제외 시킬 수 있다.
        * `@EqualsAndHashCode` : equals()와 hashCode() 메서드를 자동 생성한다.
        * `@Data` : `@Getter`, `@Setter`, `@RequiredArgsConstructor`, `@ToString`, `@EqualsAndHashCode`를 한꺼번에 설정한다.
        * `@Builder` : 빌더 패턴을 사용 할 수 있도록 한다.

* (2) MVC 란?
    
    * 모델(Model)
    
        * 화면에 전달 할 또는 화면에서 전달 받은 데이터를 담고 있는 객체이다.
    
        * 도메인 객체 또는 DTO를 말한다.
    
    * 뷰(View)
    
        * 모델이 담고 있는 데이터를 보여주는 역할을 한다.
    
        * 다양한 형태로 보여 줄 수 있다. (HTML, JSON, XML ...)
    
    * 컨트롤러(Controller)
    
        * 사용자의 입력을 받아 모델 객체의 데이터를 변경하거나 모델 객체를 뷰에 전달하는 역할을 한다.
    
            * ① 입력 값 검증
        
            * ② 입력 받은 데이터로 모델 객체를 변경
        
            * ③ 변경된 모델 객체를 뷰에 전달
    
* (3) Spring MVC
    
    * `Spring MVC`는 서블릿 기반의 웹 애플리케이션을 개발할 때, MVC 패턴을 쉽게 사용 할 수 있도록 도와주는 프레임워크이다.
    
* (4) 실습
    
    * ① 프로젝트 생성
    
        ![image 3](images/img3.png)
        
    * ② 컨트롤러(Controller) 작성
    
        ```java
        @Controller
        public class EventController {
        
            // "/events" GET 요청이 들어오면 처리 할 핸들러 지정
            // @RequestMapping(value = "/events" , method = RequestMethod.GET)
            @GetMapping("/events")
            public String events(Model model){
                return "events";
            }
        }
        ```
      
    * ③ 모델(Model) 작성
    
        ```java
        @Getter @Setter
        @Builder @NoArgsConstructor @AllArgsConstructor
        public class Event {
        
            private String name;
        
            private int limitOfEnrollment;
        
            private LocalDateTime startDateTime;
        
            private LocalDateTime endDateTime;
        }
        ```
       
    * ④ 서비스 작성
    
        ```java
        @Service
        public class EventService {
        
            public List<Event> getEvents(){
            	// Event 생성
                Event event1 = Event.builder()
                        .name("스프링 웹 MVC 스터디 1차")
                        .limitOfEnrollment(5)
                        .startDateTime(LocalDateTime.of(2019, 1, 10, 10, 0))
                        .endDateTime(LocalDateTime.of(2019, 1, 10, 12, 0))
                        .build();
        
                Event event2 = Event.builder()
                        .name("스프링 웹 MVC 스터디 2")
                        .limitOfEnrollment(5)
                        .startDateTime(LocalDateTime.of(2019, 1, 17, 10, 0))
                        .endDateTime(LocalDateTime.of(2019, 1, 17, 12, 0))
                        .build();
        
                return List.of(event1, event2); // List.of() : 변경 할 수 없는 List를 생성한다. 변경 시 예외 발생
            }
        
        }
        ```
       
    * ⑤ 컨트롤러(Controller) 변경
    
        ```java
        @Controller
        public class EventController {
        
            @Autowired
            EventService eventService; // Service를 주입 받아 사용한다.
        
            // "/events" GET 요청이 들어오면 처리 할 핸들러 지정
            // @RequestMapping(value = "/events" , method = RequestMethod.GET)
            @GetMapping("/events")
            public String events(Model model){
                model.addAttribute("events", eventService.getEvents()); // 모델에 담는다.
                return "events"; // 뷰의 이름
            }
        }
        ```
      
    * ⑥ 뷰(View) 작성 [events.html]
    
        ```html
        <!DOCTYPE html>
        <html lang="en" xmlns:th="http://www.thymeleaf.org">
        <head>
            <meta charset="UTF-8">
            <title>Title</title>
        </head>
        <body>
            <h1>이벤트 목록</h1>
            <table>
                <tr>
                    <th>이름</th>
                    <th>참가 인원</th>
                    <th>시작</th>
                    <th>종료</th>
                </tr>
                <tr th:each="event : ${events}">
                    <td th:text="${event.name}">이벤트 이름</td>
                    <td th:text="${event.limitOfEnrollment}">100</td>
                    <td th:text="${event.startDateTime}">2021년 1월 10일 오전 10시</td>
                    <td th:text="${event.endDateTime}">2021년 1월 10일 오전 12시</td>
                </tr>
            </table>
        </body>
        </html>
        ```

* (5) MVC 패턴의 장점
    
    * 동시 다발적(Simultaneous) 개발 가능

        * 백엔드 개발자와 프론트엔드 개발자가 독립적으로 개발을 진행 할 수 있다.

    * 높은 결합도
    
        * 논리적으로 관련있는 기능을 하나의 컨트롤러로 묶거나, 특정 모델과 관련있는 뷰를 그룹화 할 수 있다.

    * 낮은 의존도
    
        * 뷰, 모델, 컨트롤러는 각각 독립적이다.

        * (서로 간에 독립적이다.)

    * 개발 용이성
    
        * 책임이 구분되어 있어 코드 수정하는 것이 편하다.

    * 한 모델에 대한 여러 형태의 뷰를 가질 수 있다.

* (6) MVC 패턴의 단점

    * 코드 내비게이션 복잡함

    * 코드 일관성 유지에 노력이 필요함
    
    * 높은 학습 곡선이 필요함

#### 2) 서블릿 애플리케이션 

* (1) 서블릿(Servlet)

    * `서블릿(Servlet)`은 자바를 사용하여 웹 페이지를 동적으로 생성하는 서버 측 프로그램이다.

    * Java 코드 안에 HTML 코드가 있다.

* (2) JSP(Java Server Pages)

    * 서블릿 기반의 서버 사이드 스크립트 기술이다.

    * HTML 태그 안에 Java 코드가 있다.

* (3) 서블릿의 특징

    * 자바 엔터프라이즈 에디션은 웹 애플리케이션 개발용 스펙과 API를 제공한다.

    * 그 중에 가장 중요한 클래스 중 하나가 `HttpServlet`이며 요청 마다 새로운 프로세스를 만드는 것이 아닌 한 프로세스 내의 자원을 공유하는 스레드를 만들어서 요청을 처리한다.

    * 서블릿 등장 이전에 사용하던 기술인 CGI(Common Gateway Interface)는 요청 당 프로세스를 만들어서 사용한다.

* (4) 서블릿의 장점 (CGI에 비해)

    * 빠르다.

    * 플랫폼(OS)에 독립적이다.

    * 보안성이 좋다.

    * 이식성이 좋다.

* (5) 서블릿 엔진 또는 서블릿 컨테이너 (Tomcat, Jetty, Undertow, ...)

    * 서블릿 생명주기(Life Cycle)를 관리
    
    * 세션 관리

    * 네트워크 서비스

    * MIME(마임) 기반 메시지를 인코딩 / 디코딩

    * ...

* (6) 서블릿 생명주기(Life Cycle) 

    * 서블릿은 우리가 직접 실행 할 수 없으며 서블릿 컨테이너가 실행 할 수 있다.
    
    * 서블릿의 생명주기는 다음과 같다.

        * ① 서블릿 컨테이너가 서블릿 인스턴스의 `init()` 메소드를 호출하여 초기화 한다. 
    
            * (최초 요청을 받았을 때 한번 초기화 하고 나면 그 다음 요청 부터는 이 과정을 생략한다.)
    
        * ② 서블릿이 초기화 된 다음 부터 클라이언트의 요청을 처리 할 수 있다.
    
        * 각 요청은 별도의 쓰레드로 처리하고 이때 서블릿 인스턴스의 `service()` 메소드를 호출한다.
    
            * 이 안에서 HTTP 요청을 받고 클라이언트로 보낼 HTTP 응답을 만든다.
        
            * `service()`는 보통 HTTP Method에 따라 `doGet()`, `doPost()` 등으로 처리를 위임한다.
        
            * 따라서 보통 `doGet()` 또는 `doPost()`를 구현한다.
            
        * ③ 서블릿 컨테이너의 판단에 따라 해당 서블릿을 메모리에서 내려야 할 시점에 `destroy()`를 호출한다.

#### 3) 서블릿 애플리케이션 개발

* (1) 프로젝트 생성

    * ① Maven 프로젝트를 선택하고 `Create from archetype`를 체크한 다음, 아래와 같이 선택한다.
    
        * `archetype`는 메이븐(Maven)에서 미리 만들어 놓은 프로젝트 구조

        ![image 4](images/img4.png)

    * ② 다음과 같이 지정한다.
    
        ![image 5](images/img5.png)
        
    * ③ 메이븐 홈 디렉토리가 지정 되어 있다면 [FINISH]를 클릭한다.
      
    * (이 과정을 진행 하기 전에 메이븐이 설치 되어 있어야 한다.)  
    
        ![image 6](images/img6.png)   
        
* (2) 서블릿 생성

    * ① `pom.xml`에 서블릿 API 의존성을 추가한다.

        ```html
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        ```
      
        * provided : 어디선가 제공되는 의존성이라는 의미다. 코딩하는 시점에는 사용 할 수 있으나 War 패키징할 때는 제외된다. 
    
    * ② `src/main`에 java 디렉토리를 생성한다.
    
    * ③ 다음과 같이 `Project Structure`에서 java 디렉토리를 Source 디렉토리로 지정한다.
    
        ![image 7](images/img7.png)
        
    * ④ `src/java`에 `me.kevinntech` 패키지를 생성한 다음, HelloServlet 클래스를 작성한다.
    
        ```java
        public class HelloServlet extends HttpServlet {
        
            @Override
            public void init() throws ServletException {
                System.out.println("init");
            }
        
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                System.out.println("doGet");
                resp.getWriter().println("<html>");
                resp.getWriter().println("<head>");
                resp.getWriter().println("<body>");
                resp.getWriter().println("<h1>Hello Servlet</h1>");
                resp.getWriter().println("</body>");
                resp.getWriter().println("</head>");
                resp.getWriter().println("</html>");
            }
        
            @Override
            public void destroy() {
                System.out.println("destroy");
            }
            
        }
        ```
      
        * 서블릿을 실행 하려면 톰캣이 필요하며 서블릿을 독자적으로 실행 할 수는 없다.
    
* (3) 톰캣 설치
      
    * ① `pom.xml`에 서블릿 의존성을 추가한다.

        ![image 8](images/img8.png)
    
    * ② 다운로드 받은 파일을 압축 해제 한다.
     
    * ③ Mac의 경우, 톰캣 디렉토리로 이동하여 실행 파일(`.sh`)에 대한 실행 권한을 부여 해야한다.
    
        ![image 9](images/img9.png)
    
        * `chmod +x ./*.sh`는 현재 디렉토리에 있는 `.sh` 파일에 대해 실행 권한을 부여한다.
        
* (4) 톰캣 실행 (인텔리제이 얼티메이트 버전 기준)

    * ① [Add Configuration...]를 클릭한다.
    
        ![image 10](images/img10.png)
        
    * ② + 버튼을 클릭한 다음, Tomcat Server에서 Local를 선택한다.

        ![image 11](images/img11.png)
        
    * ③ 톰캣이 설치된 경로를 지정하고 Before launch에 maven goal로 `compile war:exploded`를 추가한다.
    
        ![image 12](images/img12.png)
        
    * ④ 그리고 Fix 버튼을 클릭한 다음, 아래와 같이 선택한다.
    
        ![image 13](images/img13.png)
        
        * `war exploded`는 war 압축을 해제한 상태로 톰캣에 배포한다는 의미다.

* (5) `web.xml`에 서블릿 등록 및 맵핑
    
```html
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>me.kevinntech.HelloServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```

#### 4) 서블릿 리스너와 서블릿 필터

    
     

